<purpose>
Extrair decisões de implementação que os agentes posteriores precisam — usando análise
prioritária do codebase e exposição de premissas em vez de questionamento estilo entrevista.

Você é um parceiro de raciocínio, não um entrevistador. Analise o codebase em profundidade,
mostre o que você acredita com base em evidências e peça ao usuário apenas que corrija o que
estiver errado.
</purpose>

<downstream_awareness>
**CONTEXT.md alimenta:**

1. **gsd-pesquisador-fase** — Lê CONTEXT.md para saber O QUE pesquisar
2. **gsd-planejador** — Lê CONTEXT.md para saber QUAIS decisões estão travadas

**Seu papel:** Capturar decisões com clareza suficiente para que os agentes posteriores possam
agir sem perguntar de novo ao usuário. A saída é idêntica ao modo discussão — mesmo formato de CONTEXT.md.
</downstream_awareness>

<philosophy>
**Filosofia do modo premissas:**

O usuário é visionário, não arqueólogo de codebase. Ele precisa de contexto suficiente para avaliar
se suas premissas batem com a intenção — não para responder perguntas que você poderia esclarecer
lendo o código.

- Leia o codebase PRIMEIRO, forme opiniões DEPOIS, pergunte APENAS sobre o que for genuinamente incerto
- Toda premissa deve citar evidência (caminhos de arquivo, padrões encontrados)
- Toda premissa deve declarar consequências se estiver errada
- Minimize interações com o usuário: ~2-4 correções vs ~15-20 perguntas
</philosophy>

<scope_guardrail>
**CRÍTICO: Sem expansão de escopo.**

O limite da fase vem de ROADMAP.md e é FIXO. A discussão esclarece COMO implementar
o que está no escopo, nunca SE deve acrescentar novas capacidades.

Quando o usuário sugerir expansão de escopo:
"[Recurso X] seria uma nova capacidade — isso é fase própria.
Quer que eu anote no backlog do roadmap? Por ora, vamos focar em [domínio da fase]."

Capture a ideia em "Ideias adiadas". Não perca, não execute.
</scope_guardrail>

<answer_validation>
**IMPORTANTE: Validação de resposta** — Após cada chamada de prompt conversacional, verifique se a resposta
está vazia ou só com espaços em branco. Se estiver:
1. Repita a pergunta uma vez com os mesmos parâmetros
2. Se ainda vazia, apresente as opções como lista numerada em texto simples

**Modo texto (`workflow.text_mode: true` no config ou flag `--text`):**
Com o modo texto ativo, não use prompt conversacional. Apresente cada pergunta como
lista numerada em texto simples e peça ao usuário que digite o número da opção.
</answer_validation>

<process>

<step name="initialize" priority="first">
Número da fase a partir do argumento (obrigatório).

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`,
`phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `has_verification`,
`plan_count`, `roadmap_exists`, `planning_exists`.

**Se `phase_found` for false:**
```
Fase [X] não encontrada no roadmap.

Use /gsd-progresso para ver as fases disponíveis.
```
Encerre o workflow.

**Se `phase_found` for true:** Continue para check_existing.

**Modo automático** — Se `--auto` estiver em ARGUMENTS:
- Em `check_existing`: selecione automaticamente "Atualizar" (se o contexto existir) ou continue sem prompt
- Em `present_assumptions`: pule o gate de confirmação, vá direto para write CONTEXT.md
- Em `correct_assumptions`: selecione automaticamente a opção recomendada para cada correção
- Registre cada escolha automática em linha
- Ao concluir, avance automaticamente para a fase de planejamento (plan-phase)
</step>

<step name="check_existing">
Verifique se CONTEXT.md já existe usando `has_context` do init.

```bash
ls ${phase_dir}/*-CONTEXT.md 2>/dev/null
```

**Se existir:**

**Se `--auto`:** Selecione automaticamente "Atualizar". Log: `[auto] Contexto existe — atualizando com análise baseada em premissas.`

**Caso contrário:** Use prompt conversacional:
- header: "Contexto"
- question: "A fase [X] já tem contexto. O que você quer fazer?"
- options:
  - "Atualizar" — Reanalisar o codebase e atualizar premissas
  - "Ver" — Mostrar o que já existe
  - "Pular" — Usar o contexto existente como está

Se "Atualizar": Carregue o existente, continue para load_prior_context
Se "Ver": Exiba CONTEXT.md, depois ofereça atualizar/pular
Se "Pular": Encerre o workflow

**Se não existir:**

Verifique `has_plans` e `plan_count` do init. **Se `has_plans` for true:**

**Se `--auto`:** Selecione automaticamente "Continuar e replanejar depois". Log: `[auto] Planos existem — seguindo com análise de premissas; replanejaremos depois.`

**Caso contrário:** Use prompt conversacional:
- header: "Planos existem"
- question: "A fase [X] já tem {plan_count} plano(s) criado(s) sem contexto do usuário. Suas decisões aqui não afetarão os planos existentes a menos que você replaneje."
- options:
  - "Continuar e replanejar depois"
  - "Ver planos existentes"
  - "Cancelar"

Se "Continuar e replanejar depois": Continue para load_prior_context.
Se "Ver planos existentes": Exiba os arquivos de plano, depois ofereça "Continuar" / "Cancelar".
Se "Cancelar": Encerre o workflow.

**Se `has_plans` for false:** Continue para load_prior_context.
</step>

<step name="load_prior_context">
Leia o contexto do projeto e das fases anteriores para não repetir perguntas já decididas.

**Passo 1: Ler arquivos de nível de projeto**
```bash
cat .planning/PROJECT.md 2>/dev/null
cat .planning/REQUIREMENTS.md 2>/dev/null
cat .planning/STATE.md 2>/dev/null
```

Extraia destes:
- **PROJECT.md** — Visão, princípios, não negociáveis, preferências do usuário
- **REQUIREMENTS.md** — Critérios de aceite, restrições
- **STATE.md** — Progresso atual, flags

**Passo 2: Ler todos os CONTEXT.md anteriores**
```bash
find .planning/phases -name "*-CONTEXT.md" 2>/dev/null | sort
```

Para cada CONTEXT.md em que o número da fase < fase atual:
- Leia a seção `<decisions>` — são preferências travadas
- Leia `<specifics>` — referências particulares ou momentos "quero assim como X"
- Anote padrões (ex.: "usuário prefere consistentemente UI mínima")

**Passo 3: Montar contexto interno `<prior_decisions>`**

Estruture a informação extraída para uso na geração de premissas.

**Se não houver contexto anterior:** Continue sem — esperado nas fases iniciais.
</step>

<step name="cross_reference_todos">
Verifique se há todos pendentes relevantes ao escopo desta fase.

```bash
TODO_MATCHES=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" todo match-phase "${PHASE_NUMBER}")
```

Analise o JSON para: `todo_count`, `matches[]`.

**Se `todo_count` for 0:** Pule em silêncio.

**Se houver correspondências:** Apresente os todos correspondentes, use prompt conversacional (multiSelect) para incorporar os relevantes ao escopo.

**Para os selecionados (incorporados):** Guarde como `<folded_todos>` para a seção `<decisions>` do CONTEXT.md.
**Para os não selecionados:** Guarde como `<reviewed_todos>` para a seção `<deferred>` do CONTEXT.md.

**Modo automático (`--auto`):** Incorpore automaticamente todos com score >= 0.4. Registre a seleção.
</step>

<step name="scout_codebase">
Varredura leve do código existente para informar a geração de premissas.

**Passo 1: Verificar se há mapas do codebase**
```bash
ls .planning/codebase/*.md 2>/dev/null
```

**Se existirem mapas:** Leia os relevantes (CONVENTIONS.md, STRUCTURE.md, STACK.md). Extraia componentes reutilizáveis, padrões, pontos de integração. Pule para o Passo 3.

**Passo 2: Se não houver mapas, faça grep direcionado**

Extraia termos-chave do objetivo da fase, busque arquivos relacionados.

```bash
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" 2>/dev/null | head -10
```

Leia os 3-5 arquivos mais relevantes.

**Passo 3: Montar `<codebase_context>` interno**

Identifique ativos reutilizáveis, padrões estabelecidos, pontos de integração e opções criativas. Guarde internamente para uso em deep_codebase_analysis.
</step>

<step name="deep_codebase_analysis">
Dispare um agente `gsd-analisador-premissas` para analisar profundamente o codebase desta fase. Isso
mantém o conteúdo bruto dos arquivos fora da janela principal de contexto, preservando o orçamento de tokens.

**Resolver tier de calibração (se USER-PROFILE.md existir):**

```bash
PROFILE_PATH="D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md"
```

Se o arquivo existir em PROFILE_PATH:
- Prioridade 1: Ler config.json > preferences.vendor_philosophy (override de nível de projeto)
- Prioridade 2: Ler USER-PROFILE.md avaliação Vendor Choices/Philosophy (global)
- Prioridade 3: Padrão "standard"

Mapear para tier de calibração:
- conservative OR thorough-evaluator → full_maturity (mais alternativas, evidência detalhada)
- opinionated → minimal_decisive (menos alternativas, recomendações decisivas)
- pragmatic-fast OR qualquer outro valor → standard

Se não houver USER-PROFILE.md: calibration_tier = "standard"

**Disparar subagente gsd-analisador-premissas:**

```
Task(subagent_type="gsd-analisador-premissas", prompt="""
Analise o codebase para a Fase {PHASE}: {phase_name}.

Objetivo da fase: {roadmap_description}
Decisões anteriores: {prior_decisions_summary}
Dicas do scout do codebase: {codebase_context_summary}
Calibração: {calibration_tier}

Seu trabalho:
1. Ler a descrição da fase {PHASE} em ROADMAP.md
2. Ler quaisquer CONTEXT.md de fases anteriores
3. Glob/Grep por arquivos relacionados a: {phase_relevant_terms}
4. Ler 5-15 arquivos-fonte mais relevantes
5. Retornar premissas estruturadas

## Formato de saída

Retorne EXATAMENTE esta estrutura:

## Premissas

### [Nome da área] (ex.: "Abordagem técnica")
- **Premissa:** [Declaração da decisão]
  - **Por este caminho:** [Evidência do codebase — cite caminhos de arquivo]
  - **Se errado:** [Consequência concreta de estar errado]
  - **Confiança:** Confiante | Provável | Incerto

(3-5 áreas, calibradas pelo tier:
- full_maturity: 3-5 áreas, 2-3 alternativas por item Provável/Incerto
- standard: 3-4 áreas, 2 alternativas por item Provável/Incerto
- minimal_decisive: 2-3 áreas, recomendação única decisiva por item)

## Precisa de pesquisa externa
[Tópicos em que o codebase sozinho é insuficiente — compatibilidade de versão de biblioteca,
melhores práticas do ecossistema, etc. Deixe vazio se o codebase der evidência suficiente.]
""")
```

Analise a resposta do subagente. Extraia:
- `assumptions[]` — cada uma com area, statement, evidence, consequence, confidence
- `needs_research[]` — tópicos que exigem pesquisa externa (pode estar vazio)

**Inicializar acumulador de refs canônicas:**
- Fonte 1: Copiar `Canonical refs:` de ROADMAP.md desta fase, expandir para caminhos completos
- Fonte 2: Verificar REQUIREMENTS.md e PROJECT.md por specs/ADRs referenciados
- Fonte 3: Acrescentar docs referenciados nos resultados do scout do codebase
</step>

<step name="external_research">
**Pule se:** `needs_research` de deep_codebase_analysis estiver vazio.

Se tópicos de pesquisa foram sinalizados, dispare um agente de pesquisa generalPurpose:

```
Task(subagent_type="generalPurpose", prompt="""
Pesquise os seguintes tópicos para a Fase {PHASE}: {phase_name}.

Tópicos que precisam de pesquisa:
{needs_research_content}

Para cada tópico, retorne:
- **Achado:** [O que você aprendeu]
- **Fonte:** [URL ou referência à documentação da biblioteca]
- **Impacto na confiança:** [Qual premissa isso resolve e para qual nível de confiança]

Use Context7 (resolve-library-id depois query-docs) para perguntas específicas de biblioteca.
Use WebSearch para ecossistema / melhores práticas.
""")
```

Mescle os achados de volta nas premissas:
- Atualize níveis de confiança onde a pesquisa resolve ambiguidade
- Acrescente atribuição de fonte às premissas afetadas
- Guarde achados de pesquisa para DISCUSSION-LOG.md

**Se nenhuma lacuna foi sinalizada:** Pule por completo. A maioria das fases pulará este passo.
</step>

<step name="present_assumptions">
Exiba todas as premissas agrupadas por área com selos de confiança.

**Formato de exibição:**

```
## Fase {PHASE}: {phase_name} — Premissas

Com base na análise do codebase, eu seguiria com:

### {Nome da área}
{Selo de confiança} **{Declaração da premissa}**
↳ Evidência: {caminhos de arquivo citados}
↳ Se errado: {consequência}

### {Nome da área 2}
...

[Se pesquisa externa foi feita:]
### Pesquisa externa aplicada
- {Tópico}: {Achado} (Fonte: {URL})
```

**Se `--auto`:**
- Se todas as premissas forem Confiante ou Provável: registre as premissas, pule para write_context.
  Log: `[auto] Todas as premissas Confiante/Provável — seguindo para captura de contexto.`
- Se alguma premissa for Incerto: registre aviso, selecione automaticamente a alternativa recomendada para
  cada item Incerto. Log: `[auto] {N} premissas Incerto resolvidas automaticamente com padrões recomendados.`
  Prossiga para write_context.

**Caso contrário:** Use prompt conversacional:
- header: "Premissas"
- question: "Tudo isso parece correto?"
- options:
  - "Sim, continuar" — Escrever CONTEXT.md com estas premissas como decisões
  - "Quero corrigir algumas" — Escolher quais premissas alterar

**Se "Sim, continuar":** Pule para write_context.
**Se "Quero corrigir algumas":** Continue para correct_assumptions.
</step>

<step name="correct_assumptions">
As premissas já foram exibidas acima em present_assumptions.

Apresente um multiSelect em que o rótulo de cada opção é a declaração da premissa e a descrição
é a consequência "Se errado":

Use prompt conversacional (multiSelect):
- header: "Correções"
- question: "Quais premissas precisam de correção?"
- options: [uma por premissa, label = declaração da premissa, description = "Se errado: {consequence}"]

Para cada correção selecionada, faça UMA pergunta focada:

Use prompt conversacional:
- header: "{Nome da área}"
- question: "O que devemos fazer em vez disso para: {declaração da premissa}?"
- options: [2-3 alternativas concretas descrevendo resultados visíveis ao usuário, opção recomendada primeiro]

Registre cada correção:
- Premissa original
- Alternativa escolhida pelo usuário
- Motivo (se fornecido via texto livre "Outro")

Depois de processar todas as correções, continue para write_context com premissas atualizadas.

**Modo automático:** Não deveria chegar a este passo (--auto pula de present_assumptions).
</step>

<step name="write_context">
Crie o diretório da fase se necessário. Escreva CONTEXT.md usando o formato padrão de 6 seções.

**Arquivo:** `${phase_dir}/${padded_phase}-CONTEXT.md`

Mapeie premissas para seções do CONTEXT.md:
- Premissas → `<decisions>` (cada premissa vira decisão travada: D-01, D-02, etc.)
- Correções → sobrescrevem a premissa original em `<decisions>`
- Áreas em que todas as premissas foram Confiante → marcadas como decisões travadas
- Áreas com correções → incluir a alternativa escolhida pelo usuário como decisão
- Todos incorporados → incluir em `<decisions>` em "### Todos incorporados"

```markdown
# Fase {PHASE}: {phase_name} - Context

**Reunido em:** {date} (modo premissas)
**Status:** Pronto para planejamento

<domain>
## Limite da fase

{Limite de domínio a partir de ROADMAP.md — declaração clara da âncora de escopo}
</domain>

<decisions>
## Decisões de implementação

### {Nome da área 1}
- **D-01:** {Decisão — da premissa ou correção}
- **D-02:** {Decisão}

### {Nome da área 2}
- **D-03:** {Decisão}

### Discricionariedade do Claude
{Premissas em que o usuário confirmou "você decide" ou manteve com confiança Provável}

### Todos incorporados
{Se houver todos incorporados ao escopo}
</decisions>

<canonical_refs>
## Referências canônicas

**Agentes posteriores DEVEM ler isto antes de planejar ou implementar.**

{Refs canônicas acumuladas do passo analyze — caminhos relativos completos}

[Se não houver specs externas: "Sem specs externas — requisitos totalmente capturados nas decisões acima"]
</canonical_refs>

<code_context>
## Insights do código existente

### Ativos reutilizáveis
{Do scout do codebase + achados do subagente gsd-analisador-premissas}

### Padrões estabelecidos
{Padrões que restringem/permitem esta fase}

### Pontos de integração
{Onde o novo código se conecta ao sistema existente}
</code_context>

<specifics>
## Ideias específicas

{Referências particulares de correções ou entrada do usuário}

[Se nenhuma: "Sem requisitos específicos — aberto a abordagens padrão"]
</specifics>

<deferred>
## Ideias adiadas

{Ideias mencionadas nas correções que estão fora do escopo}

### Todos revisados (não incorporados)
{Todos revisados mas não incorporados — com motivo}

[Se nenhuma: "Nenhuma — a análise permaneceu no escopo da fase"]
</deferred>
```

Escreva o arquivo.
</step>

<step name="write_discussion_log">
Escreva a trilha de auditoria de premissas e correções.

**Arquivo:** `${phase_dir}/${padded_phase}-DISCUSSION-LOG.md`

```markdown
# Fase {PHASE}: {phase_name} - Discussion Log (modo premissas)

> **Somente trilha de auditoria.** Não usar como entrada para agentes de planejamento, pesquisa ou execução.
> Decisões em CONTEXT.md — este log preserva a análise.

**Data:** {ISO date}
**Fase:** {padded_phase}-{phase_name}
**Modo:** premissas
**Áreas analisadas:** {nomes de área separados por vírgula}

## Premissas apresentadas

### {Nome da área}
| Premissa | Confiança | Evidência |
|----------|-----------|-----------|
| {Statement} | {Confiante/Provável/Incerto} | {caminhos de arquivo} |

{Repetir por área}

## Correções feitas

{Se houver correções:}

### {Nome da área}
- **Premissa original:** {o que o Claude assumiu}
- **Correção do usuário:** {o que o usuário escolheu}
- **Motivo:** {racional do usuário, se fornecido}

{Se não houver correções: "Sem correções — todas as premissas confirmadas."}

## Resolvidas automaticamente

{Se --auto e itens Incerto existiram:}
- {Premissa}: selecionado automaticamente {opção recomendada}

{Se não aplicável: omitir esta seção}

## Pesquisa externa

{Se pesquisa foi feita:}
- {Tópico}: {Achado} (Fonte: {URL})

{Se não houver pesquisa: omitir esta seção}
```

Escreva o arquivo.
</step>

<step name="git_commit">
Faça commit do contexto da fase e do log de discussão:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): capturar contexto da fase (modo premissas)" --files "${phase_dir}/${padded_phase}-CONTEXT.md" "${phase_dir}/${padded_phase}-DISCUSSION-LOG.md"
```

Confirme: "Commit realizado: docs(${padded_phase}): capturar contexto da fase (modo premissas)"
</step>

<step name="update_state">
Atualize STATE.md com informações da sessão:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Fase ${PHASE} contexto reunido (modo premissas)" \
  --resume-file "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Commit de STATE.md:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(state): registrar sessão de contexto da fase ${PHASE}" --files .planning/STATE.md
```
</step>

<step name="confirm_creation">
Apresente resumo e próximos passos:

```
Criado: .planning/phases/${PADDED_PHASE}-${SLUG}/${PADDED_PHASE}-CONTEXT.md

## Decisões capturadas (modo premissas)

### {Nome da área}
- {Decisão-chave} (da premissa / corrigida)

{Repetir por área}

[Se houver correções:]
## Correções aplicadas
- {Área}: {original} → {corrigido}

[Se houver ideias adiadas:]
## Anotado para depois
- {Ideia adiada} — fase futura

---

## ▶ Próximo

**Fase ${PHASE}: {phase_name}** — {Objetivo de ROADMAP.md}

`/gsd-planejar-fase ${PHASE}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-planejar-fase ${PHASE} --skip-research` — planejar sem pesquisa
- `/gsd-fase-ui ${PHASE}` — gerar contrato de design UI (se houver trabalho de frontend)
- Revisar/editar CONTEXT.md antes de continuar

---
```
</step>

<step name="auto_advance">
Verifique o gatilho de avanço automático:

1. Analise a flag `--auto` em {{GSD_ARGS}}
2. Flag de cadeia sync:
   ```bash
   if [[ ! "{{GSD_ARGS}}" =~ --auto ]]; then
     node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. Leia a flag de cadeia e a preferência do usuário:
   ```bash
   AUTO_CHAIN=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se a flag `--auto` estiver presente E `AUTO_CHAIN` não for true:**
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

**Se a flag `--auto` estiver presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true:**

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AVANÇO AUTOMÁTICO PARA O PLANO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Contexto capturado (modo premissas). Iniciando fase de planejamento (plan-phase)...
```

Dispare: `Skill(skill="gsd-plan-phase", args="${PHASE} --auto")`

Trate o retorno: FASE CONCLUÍDA / PLANEJAMENTO CONCLUÍDO / INCONCLUSIVO / LACUNAS ENCONTRADAS
(tratamento idêntico ao passo auto_advance de discutir-fase.md)

**Se nem `--auto` nem config estiverem habilitados:**
Encaminhe para o passo confirm_creation.
</step>

</process>

<success_criteria>
- Fase validada contra o roadmap
- Contexto anterior carregado (sem repetir perguntas já decididas)
- Codebase analisado em profundidade via subagente gsd-analisador-premissas (5-15 arquivos lidos)
- Premissas expostas com evidência e níveis de confiança
- Usuário confirmou ou corrigiu premissas (~2-4 interações no máximo)
- Expansão de escopo redirecionada para ideias adiadas
- CONTEXT.md captura as decisões reais (formato idêntico ao modo discussão)
- CONTEXT.md inclui canonical_refs com caminhos completos de arquivo (OBRIGATÓRIO)
- CONTEXT.md inclui code_context da análise do codebase
- DISCUSSION-LOG.md registra premissas e correções como trilha de auditoria
- STATE.md atualizado com informações da sessão
- Usuário sabe os próximos passos
</success_criteria>
