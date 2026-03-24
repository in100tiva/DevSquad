---
name: gsd-depurar
description: "Depuração sistemática com estado persistente entre resets de contexto"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-depurar` ou descreve uma tarefa correspondente a esta skill.
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
Depurar problemas usando método científico com isolamento de subagente.

**Papel do orquestrador:** Coletar sintomas, gerar agente gsd-depurador, tratar checkpoints, gerar continuações.

**Por que subagente:** Investigação consome contexto rápido (leitura de arquivos, formação de hipóteses, testes). Contexto novo de 200k por investigação. Contexto principal permanece enxuto para interação com usuário.
</objective>

<context>
Problema do usuário: {{GSD_ARGS}}

Verificar sessões ativas:
```bash
ls .planning/debug/*.md 2>/dev/null | grep -v resolved | head -5
```
</context>

<process>

## 0. Inicializar Contexto

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair `commit_docs` do JSON de init. Resolver modelo do depurador:
```bash
debugger_model=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-depurador --raw)
```

## 1. Verificar Sessões Ativas

Se sessões ativas existirem E sem {{GSD_ARGS}}:
- Listar sessões com status, hipótese, próxima ação
- Usuário escolhe número para retomar OU descreve novo problema

Se {{GSD_ARGS}} fornecido OU usuário descreve novo problema:
- Continuar para coleta de sintomas

## 2. Coletar Sintomas (se novo problema)

Usar solicitação conversacional para cada:

1. **Comportamento esperado** - O que deveria acontecer?
2. **Comportamento real** - O que acontece em vez disso?
3. **Mensagens de erro** - Algum erro? (colar ou descrever)
4. **Linha do tempo** - Quando começou? Já funcionou?
5. **Reprodução** - Como você aciona isso?

Após todos coletados, confirmar pronto para investigar.

## 3. Gerar Agente gsd-depurador

Preencher prompt e gerar:

```markdown
<objective>
Investigar problema: {slug}

**Resumo:** {trigger}
</objective>

<symptoms>
esperado: {expected}
real: {actual}
erros: {errors}
reprodução: {reproduction}
linha_do_tempo: {timeline}
</symptoms>

<mode>
symptoms_prefilled: true
goal: find_and_fix
</mode>

<debug_file>
Criar: .planning/debug/{slug}.md
</debug_file>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-depurador",
  model="{debugger_model}",
  description="Depurar {slug}"
)
```

## 4. Tratar Retorno do Agente

**Se `## CAUSA RAIZ ENCONTRADA`:**
- Exibir causa raiz e resumo de evidências
- Oferecer opções:
  - "Corrigir agora" - gerar subagente de correção
  - "Planejar correção" - sugerir /gsd-planejar-fase --gaps
  - "Correção manual" - concluído

**Se `## CHECKPOINT ATINGIDO`:**
- Apresentar detalhes do checkpoint ao usuário
- Obter resposta do usuário
- Se tipo de checkpoint é `human-verify`:
  - Se usuário confirma corrigido: continuar para que agente finalize/resolva/arquive
  - Se usuário reporta problemas: continuar para que agente retorne à investigação/correção
- Gerar agente de continuação (ver passo 5)

**Se `## INVESTIGAÇÃO INCONCLUSIVA`:**
- Mostrar o que foi verificado e eliminado
- Oferecer opções:
  - "Continuar investigando" - gerar novo agente com contexto adicional
  - "Investigação manual" - concluído
  - "Adicionar mais contexto" - coletar mais sintomas, gerar novamente

## 5. Gerar Agente de Continuação (Após Checkpoint)

Quando usuário responder ao checkpoint, gerar agente novo:

```markdown
<objective>
Continuar depuração de {slug}. Evidências estão no arquivo de debug.
</objective>

<prior_state>
<files_to_read>
- .planning/debug/{slug}.md (Estado da sessão de debug)
</files_to_read>
</prior_state>

<checkpoint_response>
**Tipo:** {checkpoint_type}
**Resposta:** {user_response}
</checkpoint_response>

<mode>
goal: find_and_fix
</mode>
```

```
Task(
  prompt=continuation_prompt,
  subagent_type="gsd-depurador",
  model="{debugger_model}",
  description="Continuar depuração {slug}"
)
```

</process>

<success_criteria>
- [ ] Sessões ativas verificadas
- [ ] Sintomas coletados (se novo)
- [ ] gsd-depurador gerado com contexto
- [ ] Checkpoints tratados corretamente
- [ ] Causa raiz confirmada antes de corrigir
</success_criteria>
</output>
