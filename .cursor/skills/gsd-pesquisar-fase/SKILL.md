---
name: gsd-pesquisar-fase
description: "Pesquisar como implementar uma fase (standalone - geralmente use /gsd-planejar-fase)"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-pesquisar-fase` ou descreve uma tarefa correspondente a esta skill.
- Trate todo o texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, solicite conversacionalmente:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Geração de Subagentes
Quando o workflow precisar gerar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

<objective>
Pesquisar como implementar uma fase. Gera agente gsd-pesquisador-fase com contexto da fase.

**Nota:** Este é um comando de pesquisa standalone. Para a maioria dos workflows, use `/gsd-planejar-fase` que integra pesquisa automaticamente.

**Use este comando quando:**
- Você quer pesquisar sem planejar ainda
- Você quer re-pesquisar após o planejamento estar completo
- Você precisa investigar antes de decidir se uma fase é viável

**Papel do orquestrador:** Analisar fase, validar contra roteiro, verificar pesquisa existente, coletar contexto, gerar agente pesquisador, apresentar resultados.

**Por que subagente:** Pesquisa consome contexto rápido (WebSearch, consultas Context7, verificação de fonte). Contexto novo de 200k para investigação. Contexto principal permanece enxuto para interação com usuário.
</objective>

<context>
Número da fase: {{GSD_ARGS}} (obrigatório)

Normalize a entrada de fase no passo 1 antes de qualquer busca de diretório.
</context>

<process>

## 0. Inicializar Contexto

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "{{GSD_ARGS}}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `phase_dir`, `phase_number`, `phase_name`, `phase_found`, `commit_docs`, `has_research`, `state_path`, `requirements_path`, `context_path`, `research_path`.

Resolver modelo do pesquisador:
```bash
RESEARCHER_MODEL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-pesquisador-fase --raw)
```

## 1. Validar Fase

```bash
PHASE_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}")
```

**Se `found` for false:** Erro e sair. **Se `found` for true:** Extrair `phase_number`, `phase_name`, `goal` do JSON.

## 2. Verificar Pesquisa Existente

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null
```

**Se existir:** Oferecer: 1) Atualizar pesquisa, 2) Ver existente, 3) Pular. Aguardar resposta.

**Se não existir:** Continuar.

## 3. Coletar Contexto da Fase

Usar caminhos do INIT (não incluir conteúdo dos arquivos no contexto do orquestrador):
- `requirements_path`
- `context_path`
- `state_path`

Apresentar resumo com descrição da fase e quais arquivos o pesquisador carregará.

## 4. Gerar Agente gsd-pesquisador-fase

Modos de pesquisa: ecosystem (padrão), feasibility, implementation, comparison.

```markdown
<research_type>
Pesquisa de Fase — investigando COMO implementar uma fase específica bem.
</research_type>

<key_insight>
A pergunta NÃO é "qual biblioteca devo usar?"

A pergunta é: "O que eu não sei que eu não sei?"

Para esta fase, descubra:
- Qual é o padrão de arquitetura estabelecido?
- Quais bibliotecas formam a stack padrão?
- Quais problemas as pessoas comumente encontram?
- O que é SOTA vs o que o treinamento do Claude acha que é SOTA?
- O que NÃO deve ser feito manualmente?
</key_insight>

<objective>
Pesquisar abordagem de implementação para Fase {phase_number}: {phase_name}
Modo: ecosystem
</objective>

<files_to_read>
- {requirements_path} (Requisitos)
- {context_path} (Contexto da fase de discutir-fase, se existir)
- {state_path} (Decisões anteriores do projeto e bloqueios)
</files_to_read>

<additional_context>
**Descrição da fase:** {phase_description}
</additional_context>

<downstream_consumer>
Seu RESEARCH.md será carregado por `/gsd-planejar-fase` que usa seções específicas:
- `## Stack Padrão` → Planos usam estas bibliotecas
- `## Padrões de Arquitetura` → Estrutura de tarefas segue estes
- `## Não Faça Manualmente` → Tarefas NUNCA constroem soluções customizadas para problemas listados
- `## Armadilhas Comuns` → Passos de verificação verificam estes
- `## Exemplos de Código` → Ações de tarefas referenciam estes padrões

Seja prescritivo, não exploratório. "Use X" e não "Considere X ou Y."
</downstream_consumer>

<quality_gate>
Antes de declarar completo, verifique:
- [ ] Todos os domínios investigados (não apenas alguns)
- [ ] Alegações negativas verificadas com docs oficiais
- [ ] Múltiplas fontes para alegações críticas
- [ ] Níveis de confiança atribuídos honestamente
- [ ] Nomes das seções correspondem ao que planejar-fase espera
</quality_gate>

<output>
Escrever em: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</output>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-pesquisador-fase",
  model="{researcher_model}",
  description="Pesquisar Fase {phase}"
)
```

## 5. Tratar Retorno do Agente

**`## PESQUISA COMPLETA`:** Exibir resumo, oferecer: Planejar fase, Aprofundar, Revisar completo, Concluído.

**`## CHECKPOINT ATINGIDO`:** Apresentar ao usuário, obter resposta, gerar continuação.

**`## PESQUISA INCONCLUSIVA`:** Mostrar o que foi tentado, oferecer: Adicionar contexto, Tentar modo diferente, Manual.

## 6. Gerar Agente de Continuação

```markdown
<objective>
Continuar pesquisa para Fase {phase_number}: {phase_name}
</objective>

<prior_state>
<files_to_read>
- .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md (Pesquisa existente)
</files_to_read>
</prior_state>

<checkpoint_response>
**Tipo:** {checkpoint_type}
**Resposta:** {user_response}
</checkpoint_response>
```

```
Task(
  prompt=continuation_prompt,
  subagent_type="gsd-pesquisador-fase",
  model="{researcher_model}",
  description="Continuar pesquisa Fase {phase}"
)
```

</process>

<success_criteria>
- [ ] Fase validada contra roteiro
- [ ] Pesquisa existente verificada
- [ ] gsd-pesquisador-fase gerado com contexto
- [ ] Checkpoints tratados corretamente
- [ ] Usuário sabe os próximos passos
</success_criteria>
</output>
