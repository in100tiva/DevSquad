<purpose>
Criar prompts de fase executáveis (arquivos PLAN.md) para uma fase do roteiro com pesquisa e verificação integradas. Fluxo padrão: Pesquisa (se necessário) -> Planejar -> Verificar -> Concluído. Orquestra agentes gsd-pesquisador-fase, gsd-planejador e gsd-verificador-plano com um loop de revisão (máx 3 iterações).
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</required_reading>

<available_agent_types>
Tipos válidos de subagente GSD (use nomes exatos — não recorra a 'general-purpose'):
- gsd-pesquisador-fase — Pesquisa abordagens técnicas para uma fase
- gsd-planejador — Cria planos detalhados a partir do escopo da fase
- gsd-verificador-plano — Revisa qualidade do plano antes da execução
</available_agent_types>

<process>

## 1. Inicializar

Carregar todo o contexto em uma chamada (apenas caminhos para minimizar contexto do orquestrador):

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `researcher_model`, `planner_model`, `checker_model`, `research_enabled`, `plan_checker_enabled`, `nyquist_validation_enabled`, `commit_docs`, `text_mode`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_reviews`, `has_plans`, `plan_count`, `planning_exists`, `roadmap_exists`, `phase_req_ids`.

**Caminhos de arquivo (para blocos <files_to_read>):** `state_path`, `roadmap_path`, `requirements_path`, `context_path`, `research_path`, `verification_path`, `uat_path`, `reviews_path`. Estes são null se os arquivos não existirem.

**Se `planning_exists` for false:** Erro — execute `/gsd-new-project` primeiro.

## 2. Analisar e Normalizar Argumentos

Extrair de {{GSD_ARGS}}: número da fase (inteiro ou decimal como `2.1`), flags (`--research`, `--skip-research`, `--gaps`, `--skip-verify`, `--prd <filepath>`, `--reviews`, `--text`).

Definir `TEXT_MODE=true` se `--text` estiver presente em {{GSD_ARGS}} OU `text_mode` do JSON de init for `true`. Quando `TEXT_MODE` estiver ativo, substituir toda chamada `conversational prompting` por uma lista numerada em texto simples e pedir ao usuário para digitar o número da escolha. Isto é necessário para sessões remotas do Cursor (modo `/rc`) onde menus TUI não funcionam através do Claude App.

Extrair `--prd <filepath>` de {{GSD_ARGS}}. Se presente, definir PRD_FILE para o filepath.

**Se sem número de fase:** Detectar próxima fase não planejada do roteiro.

**Se `phase_found` for false:** Validar que a fase existe no ROADMAP.md. Se válida, criar o diretório usando `phase_slug` e `padded_phase` do init:
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```

**Artefatos existentes do init:** `has_research`, `has_plans`, `plan_count`.

## 2.5. Validar Pré-requisito `--reviews`

**Pular se:** Sem flag `--reviews`.

**Se `--reviews` E `--gaps`:** Erro — não é possível combinar `--reviews` com `--gaps`. São modos conflitantes.

**Se `--reviews` E `has_reviews` for false (sem REVIEWS.md no diretório da fase):**

Erro:
```
Nenhum REVIEWS.md encontrado para Fase {N}. Execute reviews primeiro:

/gsd-review --phase {N}

Depois re-execute /gsd-plan-phase {N} --reviews
```
Sair do workflow.

## 3. Validar Fase

```bash
PHASE_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

**Se `found` for false:** Erro com fases disponíveis. **Se `found` for true:** Extrair `phase_number`, `phase_name`, `goal` do JSON.

## 3.5. Tratar Caminho Expresso PRD

**Pular se:** Sem flag `--prd` nos argumentos.

**Se `--prd <filepath>` fornecido:**

1. Ler o arquivo PRD:
```bash
PRD_CONTENT=$(cat "$PRD_FILE" 2>/dev/null)
if [ -z "$PRD_CONTENT" ]; then
  echo "Erro: Arquivo PRD não encontrado: $PRD_FILE"
  exit 1
fi
```

2. Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CAMINHO EXPRESSO PRD
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Usando PRD: {PRD_FILE}
Gerando CONTEXT.md a partir dos requisitos...
```

3. Analisar o conteúdo do PRD e gerar CONTEXT.md. O orquestrador deve:
   - Extrair todos os requisitos, histórias de usuário, critérios de aceitação e restrições do PRD
   - Mapear cada um para uma decisão travada (tudo no PRD é tratado como decisão travada)
   - Identificar áreas que o PRD não cobre e marcar como "Discrição do Claude"
   - **Extrair refs canônicas** do ROADMAP.md para esta fase, mais quaisquer specs/ADRs referenciados no PRD — expandir para caminhos completos (OBRIGATÓRIO)
   - Criar CONTEXT.md no diretório da fase

4. Escrever CONTEXT.md:
```markdown
# Fase [X]: [Nome] - Contexto

**Coletado:** [data]
**Status:** Pronto para planejamento
**Fonte:** Caminho Expresso PRD ({PRD_FILE})

<domain>
## Limite da Fase

[Extraído do PRD — o que esta fase entrega]

</domain>

<decisions>
## Decisões de Implementação

{Para cada requisito/história/critério no PRD:}
### [Categoria derivada do conteúdo]
- [Requisito como decisão travada]

### Discrição do Claude
[Áreas não cobertas pelo PRD — detalhes de implementação, escolhas técnicas]

</decisions>

<canonical_refs>
## Referências Canônicas

**Agentes downstream DEVEM ler estes antes de planejar ou implementar.**

[OBRIGATÓRIO. Extrair do ROADMAP.md e quaisquer docs referenciados no PRD.
Usar caminhos relativos completos. Agrupar por área temática.]

### [Área temática]
- `caminho/para/spec-ou-adr.md` — [O que decide/define]

[Se sem specs externas: "Sem specs externas — requisitos completamente capturados nas decisões acima"]

</canonical_refs>

<specifics>
## Ideias Específicas

[Quaisquer referências específicas, exemplos ou requisitos concretos do PRD]

</specifics>

<deferred>
## Ideias Adiadas

[Itens no PRD explicitamente marcados como futuro/v2/fora-do-escopo]
[Se nenhum: "Nenhum — PRD cobre o escopo da fase"]

</deferred>

---

*Fase: XX-nome*
*Contexto coletado: [data] via Caminho Expresso PRD*
```

5. Commit:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): generate context from PRD" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

6. Definir `context_content` com o conteúdo do CONTEXT.md gerado e continuar para o passo 5 (Tratar Pesquisa).

**Efeito:** Isto contorna completamente o passo 4 (Carregar CONTEXT.md) pois acabamos de criá-lo. O resto do workflow (pesquisa, planejamento, verificação) prossegue normalmente com o contexto derivado do PRD.

## 4. Carregar CONTEXT.md

**Pular se:** Caminho expresso PRD foi usado (CONTEXT.md já criado no passo 3.5).

Verificar `context_path` do JSON de init.

Se `context_path` não for null, exibir: `Usando contexto da fase de: ${context_path}`

**Se `context_path` for null (sem CONTEXT.md):**

Ler modo discuss para rótulo do portal de contexto:
```bash
DISCUSS_MODE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.discuss_mode 2>/dev/null || echo "discuss")
```

Se `TEXT_MODE` for true, apresentar como lista numerada em texto simples:
```
Nenhum CONTEXT.md encontrado para Fase {X}. Os planos usarão apenas pesquisa e requisitos — suas preferências de design não serão incluídas.

1. Continuar sem contexto — Planejar usando pesquisa + requisitos apenas
[Se DISCUSS_MODE for "assumptions":]
2. Coletar contexto (modo suposições) — Analisar codebase e apresentar suposições antes de planejar
[Se DISCUSS_MODE for "discuss" ou não definido:]
2. Executar discuss-phase primeiro — Capturar decisões de design antes de planejar

Digite o número:
```

Caso contrário use conversational prompting:
- header: "Sem contexto"
- question: "Nenhum CONTEXT.md encontrado para Fase {X}. Os planos usarão apenas pesquisa e requisitos — suas preferências de design não serão incluídas. Continuar ou capturar contexto primeiro?"
- options:
  - "Continuar sem contexto" — Planejar usando pesquisa + requisitos apenas
  Se `DISCUSS_MODE` for `"assumptions"`:
  - "Coletar contexto (modo suposições)" — Analisar codebase e apresentar suposições antes de planejar
  Se `DISCUSS_MODE` for `"discuss"` (ou não definido):
  - "Executar discuss-phase primeiro" — Capturar decisões de design antes de planejar

Se "Continuar sem contexto": Prosseguir para o passo 5.
Se "Executar discuss-phase primeiro":
  **IMPORTANTE:** NÃO invocar discuss-phase como chamada Skill/Task aninhada — conversational prompting
  não funciona corretamente em subcontextos aninhados (#1009). Em vez disso, exibir o comando
  e sair para o usuário executá-lo como comando de nível superior:
  ```
  Execute este comando primeiro, depois re-execute /gsd-plan-phase {X} ${GSD_WS}:

  /gsd-discuss-phase {X} ${GSD_WS}
  ```
  **Sair do workflow plan-phase. Não continuar.**

## 5. Tratar Pesquisa

**Pular se:** flag `--gaps` ou flag `--skip-research` ou flag `--reviews`.

**Se `has_research` for true (do init) E sem flag `--research`:** Usar existente, pular para o passo 6.

**Se RESEARCH.md ausente OU flag `--research`:**

**Se sem flag explícita (`--research` ou `--skip-research`) e não `--auto`:**
Perguntar ao usuário se deve pesquisar, com recomendação contextual baseada na fase:

Se `TEXT_MODE` for true, apresentar como lista numerada em texto simples:
```
Pesquisar antes de planejar Fase {X}: {phase_name}?

1. Pesquisar primeiro (Recomendado) — Investigar domínio, padrões e dependências antes de planejar. Melhor para funcionalidades novas, integrações desconhecidas ou mudanças arquiteturais.
2. Pular pesquisa — Planejar diretamente a partir de contexto e requisitos. Melhor para correções de bugs, refatorações simples ou tarefas bem conhecidas.

Digite o número:
```

Caso contrário use conversational prompting:
```
conversational prompting([
  {
    question: "Pesquisar antes de planejar Fase {X}: {phase_name}?",
    header: "Pesquisa",
    multiSelect: false,
    options: [
      { label: "Pesquisar primeiro (Recomendado)", description: "Investigar domínio, padrões e dependências antes de planejar. Melhor para funcionalidades novas, integrações desconhecidas ou mudanças arquiteturais." },
      { label: "Pular pesquisa", description: "Planejar diretamente a partir de contexto e requisitos. Melhor para correções de bugs, refatorações simples ou tarefas bem conhecidas." }
    ]
  }
])
```

Se o usuário selecionar "Pular pesquisa": pular para o passo 6.

**Se `--auto` e `research_enabled` for false:** Pular pesquisa silenciosamente (preserva comportamento automatizado).

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISANDO FASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando pesquisador...
```

### Iniciar gsd-pesquisador-fase

```bash
PHASE_DESC=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" --pick section)
```

Prompt de pesquisa:

```markdown
<objective>
Pesquisar como implementar Fase {phase_number}: {phase_name}
Responder: "O que preciso saber para PLANEJAR bem esta fase?"
</objective>

<files_to_read>
- {context_path} (DECISÕES DO USUÁRIO de /gsd-discuss-phase)
- {requirements_path} (Requisitos do projeto)
- {state_path} (Decisões e histórico do projeto)
</files_to_read>

<additional_context>
**Descrição da fase:** {phase_description}
**IDs de requisitos da fase (DEVEM ser tratados):** {phase_req_ids}

**Instruções do projeto:** Ler .cursor/rules/ se existir — seguir diretrizes específicas do projeto
**Skills do projeto:** Verificar diretório .cursor/skills/ ou .agents/skills/ (se algum existir) — ler arquivos SKILL.md, pesquisa deve considerar padrões de skills do projeto
</additional_context>

<output>
Escrever em: {phase_dir}/{phase_num}-RESEARCH.md
</output>
```

```
Task(
  prompt=research_prompt,
  subagent_type="gsd-pesquisador-fase",
  model="{researcher_model}",
  description="Pesquisar Fase {phase}"
)
```

### Tratar Retorno do Pesquisador

- **`## RESEARCH COMPLETE`:** Exibir confirmação, continuar para o passo 6
- **`## RESEARCH BLOCKED`:** Exibir bloqueador, oferecer: 1) Fornecer contexto, 2) Pular pesquisa, 3) Abortar

## 5.5. Criar Estratégia de Validação

Pular se `nyquist_validation_enabled` for false OU `research_enabled` for false.

Se `research_enabled` for false e `nyquist_validation_enabled` for true: avisar "Validação Nyquist habilitada mas pesquisa desabilitada — VALIDACAO.md não pode ser criado sem RESEARCH.md. Os planos não terão requisitos de validação (Dimensão 8)." Continuar para o passo 6.

**Mas Nyquist não é aplicável para esta execução** quando todos os seguintes são verdadeiros:
- `research_enabled` é false
- `has_research` é false
- nenhuma flag `--research` foi fornecida

Nesse caso: **pular criação de estratégia de validação inteiramente**. **Não** esperar `RESEARCH.md` ou `VALIDACAO.md` para esta execução, e continuar para o Passo 6.

```bash
grep -l "## Validation Architecture" "${PHASE_DIR}"/*-RESEARCH.md 2>/dev/null
```

**Se encontrado:**
1. Ler template: `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/VALIDACAO.md`
2. Escrever em `${PHASE_DIR}/${PADDED_PHASE}-VALIDACAO.md` (usar ferramenta Write)
3. Preencher frontmatter: `{N}` → número da fase, `{phase-slug}` → slug, `{date}` → data atual
4. Verificar:
```bash
test -f "${PHASE_DIR}/${PADDED_PHASE}-VALIDACAO.md" && echo "VALIDATION_CREATED=true" || echo "VALIDATION_CREATED=false"
```
5. Se `VALIDATION_CREATED=false`: PARAR — não prosseguir para o Passo 6
6. Se `commit_docs`: `commit "docs(phase-${PHASE}): add validation strategy"`

**Se não encontrado:** Avisar e continuar — planos podem falhar na Dimensão 8.

## 5.6. Portal de Contrato de Design UI

> Pular se `workflow.ui_phase` for explicitamente `false` E `workflow.ui_safety_gate` for explicitamente `false` em `.planning/config.json`. Se as chaves estiverem ausentes, tratar como habilitado.

```bash
UI_PHASE_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_phase 2>/dev/null || echo "true")
UI_GATE_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_safety_gate 2>/dev/null || echo "true")
```

**Se ambos forem `false`:** Pular para o passo 6.

Verificar se a fase tem indicadores de frontend:

```bash
PHASE_SECTION=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" 2>/dev/null)
echo "$PHASE_SECTION" | grep -iE "UI|interface|frontend|component|layout|page|screen|view|form|dashboard|widget" > /dev/null 2>&1
HAS_UI=$?
```

**Se `HAS_UI` for 0 (indicadores de frontend encontrados):**

Verificar UI-SPEC existente:
```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-ESPECIFICACAO-UI.md 2>/dev/null | head -1)
```

**Se ESPECIFICACAO-UI.md encontrado:** Definir `UI_SPEC_PATH=$UI_SPEC_FILE`. Exibir: `Usando contrato de design UI: ${UI_SPEC_PATH}`

**Se ESPECIFICACAO-UI.md ausente E `UI_GATE_CFG` for `true`:**

Se `TEXT_MODE` for true, apresentar como lista numerada em texto simples:
```
Fase {N} tem indicadores de frontend mas sem ESPECIFICACAO-UI.md. Gerar um contrato de design antes de planejar?

1. Gerar ESPECIFICACAO-UI primeiro — Executar /gsd-ui-phase {N} depois re-executar /gsd-plan-phase {N}
2. Continuar sem ESPECIFICACAO-UI
3. Não é uma fase de frontend

Digite o número:
```

Caso contrário use conversational prompting:
- header: "Contrato de Design UI"
- question: "Fase {N} tem indicadores de frontend mas sem ESPECIFICACAO-UI.md. Gerar um contrato de design antes de planejar?"
- options:
  - "Gerar ESPECIFICACAO-UI primeiro" → Exibir: "Execute `/gsd-ui-phase {N} ${GSD_WS}` depois re-execute `/gsd-plan-phase {N} ${GSD_WS}`". Sair do workflow.
  - "Continuar sem ESPECIFICACAO-UI" → Continuar para o passo 6.
  - "Não é uma fase de frontend" → Continuar para o passo 6.

**Se `HAS_UI` for 1 (sem indicadores de frontend):** Pular silenciosamente para o passo 6.

## 6. Verificar Planos Existentes

```bash
ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null
```

**Se existe E flag `--reviews`:** Pular prompt — ir direto para replanejar (o propósito de `--reviews` é replanejar com feedback de review).

**Se existe E sem flag `--reviews`:** Oferecer: 1) Adicionar mais planos, 2) Ver existentes, 3) Replanejar do zero.

## 7. Usar Caminhos de Contexto do INIT

Extrair do JSON de INIT:

```bash
_gsd_field() { node -e "const o=JSON.parse(process.argv[1]); const v=o[process.argv[2]]; process.stdout.write(v==null?'':String(v))" "$1" "$2"; }
STATE_PATH=$(_gsd_field "$INIT" state_path)
ROADMAP_PATH=$(_gsd_field "$INIT" roadmap_path)
REQUIREMENTS_PATH=$(_gsd_field "$INIT" requirements_path)
RESEARCH_PATH=$(_gsd_field "$INIT" research_path)
VERIFICATION_PATH=$(_gsd_field "$INIT" verification_path)
UAT_PATH=$(_gsd_field "$INIT" uat_path)
CONTEXT_PATH=$(_gsd_field "$INIT" context_path)
REVIEWS_PATH=$(_gsd_field "$INIT" reviews_path)
```

## 7.5. Verificar Artefatos Nyquist

Pular se `nyquist_validation_enabled` for false OU `research_enabled` for false.

Também pular se todos os seguintes forem verdadeiros:
- `research_enabled` é false
- `has_research` é false
- nenhuma flag `--research` foi fornecida

Nesse caminho sem pesquisa, artefatos Nyquist **não são necessários** para esta execução.

```bash
VALIDATION_EXISTS=$(ls "${PHASE_DIR}"/*-VALIDACAO.md 2>/dev/null | head -1)
```

Se ausente e Nyquist ainda habilitado/aplicável — perguntar ao usuário:
1. Re-executar: `/gsd-plan-phase {PHASE} --research ${GSD_WS}`
2. Desabilitar Nyquist com o comando exato:
   `node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow.nyquist_validation false`
3. Continuar mesmo assim (planos falham na Dimensão 8)

Prosseguir para o Passo 8 somente se o usuário selecionar 2 ou 3.

## 8. Iniciar Agente gsd-planejador

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PLANEJANDO FASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando planejador...
```

Prompt do planejador:

```markdown
<planning_context>
**Fase:** {phase_number}
**Modo:** {standard | gap_closure | reviews}

<files_to_read>
- {state_path} (Estado do Projeto)
- {roadmap_path} (Roteiro)
- {requirements_path} (Requisitos)
- {context_path} (DECISÕES DO USUÁRIO de /gsd-discuss-phase)
- {research_path} (Pesquisa Técnica)
- {verification_path} (Lacunas de Verificação - se --gaps)
- {uat_path} (Lacunas de TAU - se --gaps)
- {reviews_path} (Feedback de Review Cross-AI - se --reviews)
- {UI_SPEC_PATH} (Contrato de Design UI — especificações visuais/interação, se existir)
</files_to_read>

**IDs de requisitos da fase (cada ID DEVE aparecer no campo `requirements` de um plano):** {phase_req_ids}

**Instruções do projeto:** Ler .cursor/rules/ se existir — seguir diretrizes específicas do projeto
**Skills do projeto:** Verificar diretório .cursor/skills/ ou .agents/skills/ (se algum existir) — ler arquivos SKILL.md, planos devem considerar regras de skills do projeto
</planning_context>

<downstream_consumer>
Saída consumida por /gsd-execute-phase. Os planos precisam:
- Frontmatter (wave, depends_on, files_modified, autonomous)
- Tarefas em formato XML com campos read_first e acceptance_criteria (OBRIGATÓRIO em toda tarefa)
- Critérios de verificação
- must_haves para verificação objetivo-reversa
</downstream_consumer>

<deep_work_rules>
## Regras Anti-Execução Superficial (OBRIGATÓRIO)

Toda tarefa DEVE incluir estes campos — NÃO são opcionais:

1. **`<read_first>`** — Arquivos que o executor DEVE ler antes de tocar em qualquer coisa. Sempre incluir:
   - O arquivo sendo modificado (para que o executor veja o estado atual, não suposições)
   - Qualquer arquivo "fonte de verdade" referenciado no CONTEXT.md (implementações de referência, padrões existentes, arquivos de config, schemas)
   - Qualquer arquivo cujos padrões, assinaturas, tipos ou convenções devem ser replicados ou respeitados

2. **`<acceptance_criteria>`** — Condições verificáveis que provam que a tarefa foi feita corretamente. Regras:
   - Todo critério deve ser verificável com grep, leitura de arquivo, comando de teste ou saída CLI
   - NUNCA usar linguagem subjetiva ("parece correto", "propriamente configurado", "consistente com")
   - SEMPRE incluir strings exatas, padrões, valores ou saídas de comando que devem estar presentes
   - Exemplos:
     - Código: `auth.py contém def verify_token(` / `test_auth.py retorna 0`
     - Config: `.env.example contém DATABASE_URL=` / `Dockerfile contém HEALTHCHECK`
     - Docs: `README.md contém '## Instalação'` / `API.md lista todos os endpoints`
     - Infra: `deploy.yml tem passo de rollback` / `docker-compose.yml tem healthcheck para db`

3. **`<action>`** — Deve incluir valores CONCRETOS, não referências. Regras:
   - NUNCA dizer "alinhar X com Y", "corresponder X a Y", "atualizar para ser consistente" sem especificar o estado alvo exato
   - SEMPRE incluir os valores reais: chaves de config, assinaturas de função, statements SQL, nomes de classe, caminhos de import, vars de ambiente, etc.
   - Se CONTEXT.md tem uma tabela comparativa ou valores esperados, copiá-los para a ação literalmente
   - O executor deve poder completar a tarefa apenas do texto da ação, sem precisar ler CONTEXT.md ou arquivos de referência (read_first é para verificação, não descoberta)

**Por que isso importa:** Agentes executores trabalham do texto do plano. Instruções vagas como "atualizar o config para corresponder à produção" produzem mudanças superficiais de uma linha. Instruções concretas como "adicionar DATABASE_URL=postgresql://... , definir POOL_SIZE=20, adicionar REDIS_URL=redis://..." produzem trabalho completo. O custo de planos verbosos é muito menor que o custo de refazer execução superficial.
</deep_work_rules>

<quality_gate>
- [ ] Arquivos PLAN.md criados no diretório da fase
- [ ] Cada plano tem frontmatter válido
- [ ] Tarefas são específicas e acionáveis
- [ ] Toda tarefa tem `<read_first>` com pelo menos o arquivo sendo modificado
- [ ] Toda tarefa tem `<acceptance_criteria>` com condições verificáveis por grep
- [ ] Todo `<action>` contém valores concretos (sem "alinhar X com Y" sem especificar o quê)
- [ ] Dependências corretamente identificadas
- [ ] Waves atribuídas para execução paralela
- [ ] must_haves derivados do objetivo da fase
</quality_gate>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-planejador",
  model="{planner_model}",
  description="Planejar Fase {phase}"
)
```

## 9. Tratar Retorno do Planejador

- **`## PLANNING COMPLETE`:** Exibir contagem de planos. Se `--skip-verify` ou `plan_checker_enabled` for false (do init): pular para o passo 13. Caso contrário: passo 10.
- **`## CHECKPOINT REACHED`:** Apresentar ao usuário, obter resposta, iniciar continuação (passo 12)
- **`## PLANNING INCONCLUSIVE`:** Mostrar tentativas, oferecer: Adicionar contexto / Tentar novamente / Manual

## 10. Iniciar Agente gsd-verificador-plano

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO PLANOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando verificador de plano...
```

Prompt do verificador:

```markdown
<verification_context>
**Fase:** {phase_number}
**Objetivo da Fase:** {goal do ROADMAP}

<files_to_read>
- {PHASE_DIR}/*-PLAN.md (Planos para verificar)
- {roadmap_path} (Roteiro)
- {requirements_path} (Requisitos)
- {context_path} (DECISÕES DO USUÁRIO de /gsd-discuss-phase)
- {research_path} (Pesquisa Técnica — inclui Arquitetura de Validação)
</files_to_read>

**IDs de requisitos da fase (TODOS devem ser cobertos):** {phase_req_ids}

**Instruções do projeto:** Ler .cursor/rules/ se existir — verificar que planos respeitam diretrizes do projeto
**Skills do projeto:** Verificar diretório .cursor/skills/ ou .agents/skills/ (se algum existir) — verificar que planos consideram regras de skills do projeto
</verification_context>

<expected_output>
- ## VERIFICATION PASSED — todas as verificações passam
- ## ISSUES FOUND — lista estruturada de problemas
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-verificador-plano",
  model="{checker_model}",
  description="Verificar planos da Fase {phase}"
)
```

## 11. Tratar Retorno do Verificador

- **`## VERIFICATION PASSED`:** Exibir confirmação, prosseguir para o passo 13.
- **`## ISSUES FOUND`:** Exibir problemas, verificar contagem de iterações, prosseguir para o passo 12.

## 12. Loop de Revisão (Máx 3 Iterações)

Rastrear `iteration_count` (começa em 1 após plano inicial + verificação).

**Se iteration_count < 3:**

Exibir: `Enviando de volta ao planejador para revisão... (iteração {N}/3)`

Prompt de revisão:

```markdown
<revision_context>
**Fase:** {phase_number}
**Modo:** revisão

<files_to_read>
- {PHASE_DIR}/*-PLAN.md (Planos existentes)
- {context_path} (DECISÕES DO USUÁRIO de /gsd-discuss-phase)
</files_to_read>

**Problemas do verificador:** {structured_issues_from_checker}
</revision_context>

<instructions>
Fazer atualizações direcionadas para tratar os problemas do verificador.
NÃO replanejar do zero a menos que os problemas sejam fundamentais.
Retornar o que mudou.
</instructions>
```

```
Task(
  prompt=revision_prompt,
  subagent_type="gsd-planejador",
  model="{planner_model}",
  description="Revisar planos da Fase {phase}"
)
```

Após planejador retornar -> iniciar verificador novamente (passo 10), incrementar iteration_count.

**Se iteration_count >= 3:**

Exibir: `Máximo de iterações atingido. {N} problemas permanecem:` + lista de problemas

Oferecer: 1) Forçar prosseguimento, 2) Fornecer orientação e tentar novamente, 3) Abandonar

## 13. Portal de Cobertura de Requisitos

Após os planos passarem no verificador (ou verificador ser pulado), verificar que todos os requisitos da fase são cobertos por pelo menos um plano.

**Pular se:** `phase_req_ids` for null ou TBD (sem requisitos mapeados para esta fase).

**Passo 1: Extrair IDs de requisitos declarados pelos planos**
```bash
# Coletar todos os IDs de requisitos do frontmatter dos planos
PLAN_REQS=$(grep -h "requirements_addressed\|requirements:" ${PHASE_DIR}/*-PLAN.md 2>/dev/null | tr -d '[]' | tr ',' '\n' | sed 's/^[[:space:]]*//' | sort -u)
```

**Passo 2: Comparar com requisitos da fase do ROADMAP**

Para cada REQ-ID em `phase_req_ids`:
- Se REQ-ID aparece em `PLAN_REQS` → coberto ✓
- Se REQ-ID NÃO aparece em nenhum plano → não coberto ✗

**Passo 3: Verificar funcionalidades do CONTEXT.md contra objetivos dos planos**

Ler seção `<decisions>` do CONTEXT.md. Extrair nomes de funcionalidades/capacidades. Verificar cada contra blocos `<objective>` dos planos. Funcionalidades não mencionadas em nenhum objetivo de plano → potencialmente removidas.

**Passo 4: Relatório**

Se todos os requisitos cobertos e nenhuma funcionalidade removida:
```
✓ Cobertura de requisitos: {N}/{N} REQ-IDs cobertos por planos
```
→ Prosseguir para o passo 14.

Se lacunas encontradas:
```
## ⚠ Lacuna de Cobertura de Requisitos

{M} de {N} requisitos da fase não estão atribuídos a nenhum plano:

| REQ-ID | Descrição | Planos |
|--------|-----------|--------|
| {id} | {do REQUIREMENTS.md} | Nenhum |

{K} funcionalidades do CONTEXT.md não encontradas em objetivos dos planos:
- {feature_name} — descrita no CONTEXT.md mas nenhum plano a cobre

Opções:
1. Re-planejar para incluir requisitos faltantes (recomendado)
2. Mover requisitos não cobertos para a próxima fase
3. Prosseguir mesmo assim — aceitar lacunas de cobertura
```

Se `TEXT_MODE` for true, apresentar como lista numerada em texto simples (opções já mostradas no bloco acima). Caso contrário use conversational prompting para apresentar as opções.

## 14. Apresentar Status Final

Rotear para `<offer_next>` OU `auto_advance` dependendo de flags/config.

## 15. Verificação de Auto-Avanço

Verificar gatilho de auto-avanço:

1. Analisar flag `--auto` de {{GSD_ARGS}}
2. **Sincronizar flag de cadeia com intenção** — se o usuário invocou manualmente (sem `--auto`), limpar a flag efêmera de cadeia de qualquer cadeia `--auto` interrompida anteriormente. Isto NÃO toca em `workflow.auto_advance` (a preferência persistente do usuário nas configurações):
   ```bash
   if [[ ! "{{GSD_ARGS}}" =~ --auto ]]; then
     node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. Ler tanto a flag de cadeia quanto a preferência do usuário:
   ```bash
   AUTO_CHAIN=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se flag `--auto` presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true:**

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTO-AVANÇANDO PARA EXECUÇÃO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Planos prontos. Iniciando execute-phase...
```

Iniciar execute-phase usando a ferramenta Skill para evitar sessões Task aninhadas (que causam congelamentos de runtime devido a aninhamento profundo de agentes):
```
Skill(skill="gsd-execute-phase", args="${PHASE} --auto --no-transition ${GSD_WS}")
```

A flag `--no-transition` diz ao execute-phase para retornar status após verificação ao invés de encadear adiante. Isso mantém a cadeia de auto-avanço plana — cada fase executa no mesmo nível de aninhamento ao invés de gerar agentes Task mais profundos.

**Tratar retorno do execute-phase:**
- **PHASE COMPLETE** → Exibir resumo final:
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► FASE ${PHASE} COMPLETA ✓
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Pipeline de auto-avanço finalizado.

  Próximo: /gsd-discuss-phase ${NEXT_PHASE} --auto ${GSD_WS}
  ```
- **GAPS FOUND / VERIFICATION FAILED** → Exibir resultado, parar cadeia:
  ```
  Auto-avanço parado: Execução precisa de revisão.

  Revise a saída acima e continue manualmente:
  /gsd-execute-phase ${PHASE} ${GSD_WS}
  ```

**Se nem `--auto` nem config habilitado:**
Rotear para `<offer_next>` (comportamento existente).

</process>

<offer_next>
Exibir este markdown diretamente (não como bloco de código):

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► FASE {X} PLANEJADA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {X}: {Nome}** — {N} plano(s) em {M} wave(s)

| Wave | Planos | O que constrói |
|------|--------|----------------|
| 1    | 01, 02 | [objetivos] |
| 2    | 03     | [objetivo]  |

Pesquisa: {Completada | Usou existente | Pulada}
Verificação: {Aprovada | Aprovada com override | Pulada}

───────────────────────────────────────────────────────────────

## ▶ Próximo

**Executar Fase {X}** — executar todos os {N} planos

/gsd-execute-phase {X} ${GSD_WS}

<sub>/clear primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────

**Também disponível:**
- cat .planning/phases/{phase-dir}/*-PLAN.md — revisar planos
- /gsd-plan-phase {X} --research — re-pesquisar primeiro
- /gsd-review --phase {X} --all — revisar planos com IAs externas
- /gsd-plan-phase {X} --reviews — replanejar incorporando feedback de review

───────────────────────────────────────────────────────────────
</offer_next>

<windows_troubleshooting>
**Usuários Windows:** Se plan-phase congelar durante o spawn de agentes (comum no Windows devido a
deadlocks de stdio com servidores MCP — veja Cursor issue anthropics/claude-code#28126):

1. **Force-kill:** Fechar o terminal (Ctrl+C pode não funcionar)
2. **Limpar processos órfãos:**
   ```powershell
   # Matar processos node órfãos de servidores MCP obsoletos
   Get-Process node -ErrorAction SilentlyContinue | Where-Object {$_.StartTime -lt (Get-Date).AddHours(-1)} | Stop-Process -Force
   ```
3. **Limpar diretórios de tarefas obsoletos:**
   ```powershell
   # Remover diretórios de subagentes obsoletos (Cursor nunca limpa em crash)
   Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\tasks\*" -ErrorAction SilentlyContinue
   ```
4. **Reduzir contagem de servidores MCP:** Desabilitar temporariamente servidores MCP não essenciais em settings.json
5. **Tentar novamente:** Reiniciar Cursor e executar `/gsd-plan-phase` novamente

Se congelamentos persistirem, tente `--skip-research` para reduzir a cadeia de agentes de 3 para 2:
```
/gsd-plan-phase N --skip-research
```
</windows_troubleshooting>

<success_criteria>
- [ ] Diretório .planning/ validado
- [ ] Fase validada contra o roteiro
- [ ] Diretório da fase criado se necessário
- [ ] CONTEXT.md carregado cedo (passo 4) e passado a TODOS os agentes
- [ ] Pesquisa completada (a menos que --skip-research ou --gaps ou já existir)
- [ ] gsd-pesquisador-fase iniciado com CONTEXT.md
- [ ] Planos existentes verificados
- [ ] gsd-planejador iniciado com CONTEXT.md + RESEARCH.md
- [ ] Planos criados (PLANNING COMPLETE ou CHECKPOINT tratado)
- [ ] gsd-verificador-plano iniciado com CONTEXT.md
- [ ] Verificação aprovada OU override do usuário OU máximo de iterações com decisão do usuário
- [ ] Usuário vê status entre spawns de agentes
- [ ] Usuário sabe os próximos passos
</success_criteria>
</output>
