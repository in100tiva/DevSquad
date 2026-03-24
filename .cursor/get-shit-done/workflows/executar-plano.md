<purpose>
Executar um prompt de fase (PLAN.md) e criar o resumo de resultado (SUMMARY.md).
</purpose>

<required_reading>
Ler STATE.md antes de qualquer operação para carregar contexto do projeto.
Ler config.json para configurações de comportamento de planejamento.

@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/integracao-git.md
</required_reading>

<process>

<step name="init_context" priority="first">
Carregar contexto de execução (apenas caminhos para minimizar contexto do orquestrador):

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `executor_model`, `commit_docs`, `sub_repos`, `phase_dir`, `phase_number`, `plans`, `summaries`, `incomplete_plans`, `state_path`, `config_path`.

Se `.planning/` ausente: erro.
</step>

<step name="identify_plan">
```bash
# Usar planos/sumários do JSON de INIT, ou listar arquivos
ls .planning/phases/XX-name/*-PLAN.md 2>/dev/null | sort
ls .planning/phases/XX-name/*-SUMMARY.md 2>/dev/null | sort
```

Encontrar o primeiro PLAN sem SUMMARY correspondente. Fases decimais suportadas (`01.1-hotfix/`):

```bash
PHASE=$(echo "$PLAN_PATH" | grep -oE '[0-9]+(\.[0-9]+)?-[0-9]+')
# configurações podem ser buscadas via gsd-tools config-get se necessário
```

<if mode="yolo">
Auto-aprovar: `⚡ Executar {phase}-{plan}-PLAN.md [Plano X de Y para Fase Z]` → parse_segments.
</if>

<if mode="interactive" OR="custom with gates.execute_next_plan true">
Apresentar identificação do plano, aguardar confirmação.
</if>
</step>

<step name="record_start_time">
```bash
PLAN_START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_START_EPOCH=$(date +%s)
```
</step>

<step name="parse_segments">
```bash
grep -n "type=\"checkpoint" .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```

**Roteamento por tipo de checkpoint:**

| Checkpoints | Padrão | Execução |
|-------------|--------|----------|
| Nenhum | A (autônomo) | Subagente único: plano completo + SUMMARY + commit |
| Somente verificação | B (segmentado) | Segmentos entre checkpoints. Após none/human-verify → SUBAGENTE. Após decision/human-action → PRINCIPAL |
| Decisão | C (principal) | Executar inteiramente no contexto principal |

**Padrão A:** init_agent_tracking → iniciar Task(subagent_type="gsd-executor", model=executor_model, isolation="worktree") com prompt: executar plano em [caminho], autônomo, todas as tarefas + SUMMARY + commit, seguir regras de desvio/auth, reportar: nome do plano, tarefas, caminho do SUMMARY, hash do commit → rastrear agent_id → aguardar → atualizar rastreamento → reportar.

**Padrão B:** Executar segmento por segmento. Segmentos autônomos: iniciar subagente para tarefas atribuídas apenas (sem SUMMARY/commit). Checkpoints: contexto principal. Após todos os segmentos: agregar, criar SUMMARY, commitar. Veja segment_execution.

**Padrão C:** Executar no principal usando fluxo padrão (step name="execute").

Contexto limpo por subagente preserva qualidade de pico. Contexto principal permanece enxuto.
</step>

<step name="init_agent_tracking">
```bash
if [ ! -f .planning/agent-history.json ]; then
  echo '{"version":"1.0","max_entries":50,"entries":[]}' > .planning/agent-history.json
fi
rm -f .planning/current-agent-id.txt
if [ -f .planning/current-agent-id.txt ]; then
  INTERRUPTED_ID=$(cat .planning/current-agent-id.txt)
  echo "Agente interrompido encontrado: $INTERRUPTED_ID"
fi
```

Se interrompido: perguntar ao usuário para retomar (parâmetro `resume` do Task) ou começar novo.

**Protocolo de rastreamento:** No spawn: escrever agent_id em `current-agent-id.txt`, adicionar a agent-history.json: `{"agent_id":"[id]","task_description":"[desc]","phase":"[phase]","plan":"[plan]","segment":[num|null],"timestamp":"[ISO]","status":"spawned","completion_timestamp":null}`. Na conclusão: status → "completed", definir completion_timestamp, deletar current-agent-id.txt. Poda: se entries > max_entries, remover "completed" mais antigos (nunca "spawned").

Executar para Padrão A/B antes do spawn. Padrão C: pular.
</step>

<step name="segment_execution">
Somente Padrão B (checkpoints de verificação apenas). Pular para A/C.

1. Analisar mapa de segmentos: localizações e tipos de checkpoint
2. Por segmento:
   - Rota de subagente: iniciar gsd-executor para tarefas atribuídas apenas. Prompt: intervalo de tarefas, caminho do plano, ler plano completo para contexto, executar tarefas atribuídas, rastrear desvios, SEM SUMMARY/commit. Rastrear via protocolo de agente.
   - Rota principal: executar tarefas usando fluxo padrão (step name="execute")
3. Após TODOS os segmentos: agregar arquivos/desvios/decisões → criar SUMMARY.md → commitar → auto-verificação:
   - Verificar que key-files.created existem no disco com `[ -f ]`
   - Verificar que `git log --oneline --all --grep="{phase}-{plan}"` retorna ≥1 commit
   - Anexar `## Auto-verificação: PASSOU` ou `## Auto-verificação: FALHOU` ao SUMMARY
</step>

<step name="load_prompt">
```bash
cat .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```
Estas são as instruções de execução. Seguir exatamente. Se o plano referencia CONTEXT.md: honrar a visão do usuário durante todo o processo.

**Se o plano contém bloco `<interfaces>`:** Estas são definições de tipo e contratos pré-extraídos. Usá-los diretamente — NÃO reler os arquivos fonte para descobrir tipos. O planejador já extraiu o que você precisa.
</step>

<step name="previous_phase_check">
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phases list --type summaries --raw
# Extrair o penúltimo resumo do resultado JSON
```
Se SUMMARY anterior tem "Issues Encountered" ou bloqueadores "Next Phase Readiness" não resolvidos: conversational prompting(header="Problemas Anteriores", options: "Prosseguir mesmo assim" | "Tratar primeiro" | "Revisar anterior").
</step>

<step name="execute">
Desvios são normais — tratar via regras abaixo.

1. Ler arquivos @context do prompt
2. **Ferramentas MCP:** Se .cursor/rules/ ou instruções do projeto referenciam ferramentas MCP (ex: jCodeMunch para navegação de código), preferi-las sobre Grep/Glob quando disponíveis. Recorrer a Grep/Glob se ferramentas MCP não estiverem acessíveis.
3. Por tarefa:
   - **Portal read_first OBRIGATÓRIO:** Se a tarefa tem campo `<read_first>`, você DEVE ler todo arquivo listado ANTES de fazer qualquer edição. Isto não é opcional. Não pular arquivos porque você "já sabe" o que contêm — leia-os. Os arquivos read_first estabelecem a verdade base para a tarefa.
   - `type="auto"`: se `tdd="true"` → execução TDD. Implementar com regras de desvio + portais de auth. Verificar critérios de conclusão. Commitar (veja task_commit). Rastrear hash para Summary.
   - `type="checkpoint:*"`: PARAR → checkpoint_protocol → aguardar usuário → continuar somente após confirmação.
   - **Verificação acceptance_criteria OBRIGATÓRIA:** Após completar cada tarefa, se tem `<acceptance_criteria>`, verificar CADA critério antes de mover para a próxima tarefa. Usar grep, leitura de arquivo ou comandos CLI para confirmar cada critério. Se qualquer critério falhar, corrigir a implementação antes de prosseguir. Não pular critérios ou marcá-los como "verificarei depois".
3. Executar verificações `<verification>`
4. Confirmar `<success_criteria>` atendidos
5. Documentar desvios no Summary
</step>

<authentication_gates>

## Portais de Autenticação

Erros de auth durante execução NÃO são falhas — são pontos de interação esperados.

**Indicadores:** "Not authenticated", "Unauthorized", 401/403, "Please run {tool} login", "Set {ENV_VAR}"

**Protocolo:**
1. Reconhecer portal de auth (não é um bug)
2. PARAR execução da tarefa
3. Criar checkpoint dinâmico:human-action com passos exatos de auth
4. Aguardar usuário autenticar
5. Verificar que credenciais funcionam
6. Tentar novamente tarefa original
7. Continuar normalmente

**Exemplo:** `vercel --yes` → "Not authenticated" → checkpoint pedindo ao usuário para `vercel login` → verificar com `vercel whoami` → tentar deploy novamente → continuar

**No Summary:** Documentar como fluxo normal sob "## Portais de Autenticação", não como desvios.

</authentication_gates>

<deviation_rules>

## Regras de Desvio

Você descobrirá trabalho não planejado. Aplicar automaticamente, rastrear todos para o Summary.

| Regra | Gatilho | Ação | Permissão |
|-------|---------|------|-----------|
| **1: Bug** | Comportamento quebrado, erros, queries erradas, erros de tipo, vulns de segurança, race conditions, leaks | Corrigir → testar → verificar → rastrear `[Regra 1 - Bug]` | Auto |
| **2: Crítico Faltante** | Essenciais faltantes: tratamento de erro, validação, auth, CSRF/CORS, rate limiting, índices, logging | Adicionar → testar → verificar → rastrear `[Regra 2 - Crítico Faltante]` | Auto |
| **3: Bloqueante** | Impede conclusão: deps faltantes, tipos errados, imports quebrados, env/config/arquivos faltantes, deps circulares | Corrigir bloqueador → verificar que prossegue → rastrear `[Regra 3 - Bloqueante]` | Auto |
| **4: Arquitetural** | Mudança estrutural: nova tabela DB, mudança de schema, novo serviço, troca de libs, API quebrando, nova infra | PARAR → apresentar decisão (abaixo) → rastrear `[Regra 4 - Arquitetural]` | Perguntar usuário |

**Formato da Regra 4:**
```
⚠️ Decisão Arquitetural Necessária

Tarefa atual: [nome da tarefa]
Descoberta: [o que motivou isto]
Mudança proposta: [modificação]
Por que necessário: [justificativa]
Impacto: [o que isto afeta]
Alternativas: [outras abordagens]

Prosseguir com mudança proposta? (sim / abordagem diferente / adiar)
```

**Prioridade:** Regra 4 (PARAR) > Regras 1-3 (auto) > incerto → Regra 4
**Casos extremos:** validação faltante → R2 | crash de null → R1 | nova tabela → R4 | nova coluna → R1/2
**Heurística:** Afeta correção/segurança/conclusão? → R1-3. Talvez? → R4.

</deviation_rules>

<deviation_documentation>

## Documentando Desvios

O Summary DEVE incluir seção de desvios. Nenhum? → `## Desvios do Plano\n\nNenhum - plano executado exatamente como escrito.`

Por desvio: **[Regra N - Categoria] Título** — Encontrado durante: Tarefa X | Problema | Correção | Arquivos modificados | Verificação | Hash do commit

Terminar com: **Total de desvios:** N auto-corrigidos (detalhamento). **Impacto:** avaliação.

</deviation_documentation>

<tdd_plan_execution>
## Execução TDD

Para planos `type: tdd` — RED-GREEN-REFACTOR:

1. **Infraestrutura** (somente primeiro plano TDD): detectar projeto, instalar framework, config, verificar suíte vazia
2. **RED:** Ler `<behavior>` → teste(s) que falham → executar (DEVE falhar) → commit: `test({phase}-{plan}): adicionar teste que falha para [funcionalidade]`
3. **GREEN:** Ler `<implementation>` → código mínimo → executar (DEVE passar) → commit: `feat({phase}-{plan}): implementar [funcionalidade]`
4. **REFACTOR:** Limpar → testes DEVEM passar → commit: `refactor({phase}-{plan}): limpar [funcionalidade]`

Erros: RED não falha → investigar teste/feature existente. GREEN não passa → debugar, iterar. REFACTOR quebra → desfazer.

Veja `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/tdd.md` para estrutura.
</tdd_plan_execution>

<precommit_failure_handling>
## Tratamento de Falha de Hook Pre-commit

Seus commits podem disparar hooks pre-commit. Hooks de auto-correção se tratam transparentemente — arquivos são corrigidos e re-staged automaticamente.

**Se executando como agente executor paralelo (iniciado por execute-phase):**
Use `--no-verify` em todos os commits. Hooks pre-commit causam contenção de lock de build quando múltiplos agentes commitam simultaneamente (ex: disputas de cargo lock em projetos Rust). O orquestrador valida uma vez após todos os agentes completarem.

**Se executando como único executor (modo sequencial):**
Se um commit for BLOQUEADO por um hook:

1. O comando `git commit` falha com saída de erro do hook
2. Ler o erro — ele diz exatamente qual hook e o que falhou
3. Corrigir o problema (erro de tipo, violação de lint, vazamento de segredo, etc.)
4. `git add` os arquivos corrigidos
5. Tentar o commit novamente
6. Orçar 1-2 ciclos de tentativa por commit
</precommit_failure_handling>

<task_commit>
## Protocolo de Commit por Tarefa

Após cada tarefa (verificação passada, critérios de conclusão atendidos), commitar imediatamente.

**1. Verificar:** `git status --short`

**2. Staged individualmente** (NUNCA `git add .` ou `git add -A`):
```bash
git add src/api/auth.ts
git add src/types/user.ts
```

**3. Tipo de commit:**

| Tipo | Quando | Exemplo |
|------|--------|---------|
| `feat` | Nova funcionalidade | feat(08-02): criar endpoint de registro de usuário |
| `fix` | Correção de bug | fix(08-02): corrigir regex de validação de e-mail |
| `test` | Somente teste (TDD RED) | test(08-02): adicionar teste que falha para hashing de senha |
| `refactor` | Sem mudança de comportamento (TDD REFACTOR) | refactor(08-02): extrair validação para helper |
| `perf` | Performance | perf(08-02): adicionar índice no banco |
| `docs` | Documentação | docs(08-02): adicionar documentação da API |
| `style` | Formatação | style(08-02): formatar módulo de auth |
| `chore` | Config/deps | chore(08-02): adicionar dependência bcrypt |

**4. Formato:** `{type}({phase}-{plan}): {description}` com bullet points para mudanças chave.

<sub_repos_commit_flow>
**Modo sub-repos:** Se `sub_repos` estiver configurado (array não vazio do contexto de init), usar `commit-to-subrepo` ao invés de git commit padrão. Isso roteia arquivos para seu sub-repo correto baseado no prefixo do caminho.

```bash
node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs commit-to-subrepo "{type}({phase}-{plan}): {description}" --files file1 file2 ...
```

O comando agrupa arquivos por prefixo de sub-repo e commita atomicamente em cada. Retorna JSON: `{ committed: true, repos: { "backend": { hash: "abc", files: [...] }, ... } }`.

Registrar hashes de cada repo na resposta para rastreamento do SUMMARY.

**Se `sub_repos` estiver vazio ou não definido:** Usar fluxo padrão de git commit abaixo.
</sub_repos_commit_flow>

**5. Registrar hash:**
```bash
TASK_COMMIT=$(git rev-parse --short HEAD)
TASK_COMMITS+=("Task ${TASK_NUM}: ${TASK_COMMIT}")
```

**6. Verificar arquivos gerados não rastreados:**
```bash
git status --short | grep '^??'
```
Se novos arquivos não rastreados apareceram após executar scripts ou ferramentas, decidir para cada:
- **Commitar** — se é arquivo fonte, config ou artefato intencional
- **Adicionar ao .gitignore** — se é saída gerada/runtime (artefatos de build, arquivos `.env`, cache, output compilado)
- NÃO deixar arquivos gerados não rastreados

</task_commit>

<step name="checkpoint_protocol">
Em `type="checkpoint:*"`: automatizar tudo possível primeiro. Checkpoints são apenas para verificação/decisões.

Exibir: caixa `CHECKPOINT: [Tipo]` → Progresso {X}/{Y} → Nome da tarefa → conteúdo específico do tipo → `SUA AÇÃO: [sinal]`

| Tipo | Conteúdo | Sinal de retomada |
|------|----------|-------------------|
| human-verify (90%) | O que foi construído + passos de verificação (comandos/URLs) | "aprovado" ou descrever problemas |
| decision (9%) | Decisão necessária + contexto + opções com prós/contras | "Selecionar: option-id" |
| human-action (1%) | O que foi automatizado + UM passo manual + plano de verificação | "concluído" |

Após resposta: verificar se especificado. Aprovado → continuar. Falhou → informar, aguardar. AGUARDAR o usuário — NÃO alucinar conclusão.

Veja D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/pontos-verificacao.md para detalhes.
</step>

<step name="checkpoint_return_for_orchestrator">
Quando iniciado via Task e atingindo checkpoint: retornar estado estruturado (não pode interagir com usuário diretamente).

**Retorno obrigatório:** 1) Tabela de Tarefas Completadas (hashes + arquivos) 2) Tarefa Atual (o que está bloqueando) 3) Detalhes do Checkpoint (conteúdo voltado ao usuário) 4) Aguardando (o que é necessário do usuário)

Orquestrador analisa → apresenta ao usuário → inicia continuação nova com seu estado de tarefas completadas. Você NÃO será retomado. No contexto principal: usar checkpoint_protocol acima.
</step>

<step name="verification_failure_gate">
Se verificação falhar:

**Verificar se reparo de nó está habilitado** (padrão: ligado):
```bash
NODE_REPAIR=$(node "./.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.node_repair 2>/dev/null || echo "true")
```

Se `NODE_REPAIR` for `true`: invocar `@./.cursor/get-shit-done/workflows/reparar-node.md` com:
- FAILED_TASK: número da tarefa, nome, critérios de conclusão
- ERROR: resultado esperado vs real
- PLAN_CONTEXT: nomes de tarefas adjacentes + objetivo da fase
- REPAIR_BUDGET: `workflow.node_repair_budget` do config (padrão: 2)

Reparo de nó tentará RETRY, DECOMPOSE ou PRUNE autonomamente. Só atinge este portal novamente se o orçamento de reparo for esgotado (ESCALATE).

Se `NODE_REPAIR` for `false` OU reparo retornar ESCALATE: PARAR. Apresentar: "Verificação falhou para Tarefa [X]: [nome]. Esperado: [critério]. Real: [resultado]. Reparo tentado: [resumo do que foi tentado]." Opções: Tentar novamente | Pular (marcar incompleto) | Parar (investigar). Se pulado → SUMMARY "Issues Encountered".
</step>

<step name="record_completion_time">
```bash
PLAN_END_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_END_EPOCH=$(date +%s)

DURATION_SEC=$(( PLAN_END_EPOCH - PLAN_START_EPOCH ))
DURATION_MIN=$(( DURATION_SEC / 60 ))

if [[ $DURATION_MIN -ge 60 ]]; then
  HRS=$(( DURATION_MIN / 60 ))
  MIN=$(( DURATION_MIN % 60 ))
  DURATION="${HRS}h ${MIN}m"
else
  DURATION="${DURATION_MIN} min"
fi
```
</step>

<step name="generate_user_setup">
```bash
grep -A 50 "^user_setup:" .planning/phases/XX-name/{phase}-{plan}-PLAN.md | head -50
```

Se user_setup existir: criar `{phase}-USER-SETUP.md` usando template `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/user-setup.md`. Por serviço: tabela de vars de ambiente, checklist de configuração de conta, config de dashboard, notas de dev local, comandos de verificação. Status "Incompleto". Definir `USER_SETUP_CREATED=true`. Se vazio/ausente: pular.
</step>

<step name="create_summary">
Criar `{phase}-{plan}-SUMMARY.md` em `.planning/phases/XX-name/`. Usar `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/resumo.md`.

**Frontmatter:** phase, plan, subsystem, tags | requires/provides/affects | tech-stack.added/patterns | key-files.created/modified | key-decisions | requirements-completed (**DEVE** copiar array `requirements` do frontmatter do PLAN.md literalmente) | duration ($DURATION), completed ($PLAN_END_TIME data).

Título: `# Fase [X] Plano [Y]: [Nome] Resumo`

Resumo de uma linha SUBSTANTIVO: "Auth JWT com rotação de refresh usando biblioteca jose" não "Autenticação implementada"

Incluir: duração, horas de início/fim, contagem de tarefas, contagem de arquivos.

Próximo: mais planos → "Pronto para {próximo-plano}" | último → "Fase completa, pronto para próximo passo".
</step>

<step name="update_current_position">
Atualizar STATE.md usando gsd-tools:

```bash
# Avançar contador de plano (trata caso de último-plano)
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state advance-plan

# Recalcular barra de progresso do estado em disco
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state update-progress

# Registrar métricas de execução
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state record-metric \
  --phase "${PHASE}" --plan "${PLAN}" --duration "${DURATION}" \
  --tasks "${TASK_COUNT}" --files "${FILE_COUNT}"
```
</step>

<step name="extract_decisions_and_issues">
Do SUMMARY: Extrair decisões e adicionar ao STATE.md:

```bash
# Adicionar cada decisão das key-decisions do SUMMARY
# Preferir inputs de arquivo para texto seguro no shell (preserva `$`, `*`, etc. exatamente)
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state add-decision \
  --phase "${PHASE}" --summary-file "${DECISION_TEXT_FILE}" --rationale-file "${RATIONALE_FILE}"

# Adicionar bloqueadores se algum encontrado
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state add-blocker --text-file "${BLOCKER_TEXT_FILE}"
```
</step>

<step name="update_session_continuity">
Atualizar info de sessão usando gsd-tools:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Completed ${PHASE}-${PLAN}-PLAN.md" \
  --resume-file "None"
```

Manter STATE.md abaixo de 150 linhas.
</step>

<step name="issues_review_gate">
Se SUMMARY "Issues Encountered" ≠ "None": yolo → registrar e continuar. Interativo → apresentar problemas, aguardar reconhecimento.
</step>

<step name="update_roadmap">
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap update-plan-progress "${PHASE}"
```
Conta arquivos PLAN vs SUMMARY no disco. Atualiza linha da tabela de progresso com contagem correta e status (`Em Progresso` ou `Completo` com data).
</step>

<step name="update_requirements">
Marcar requisitos completados do campo `requirements:` do frontmatter do PLAN.md:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" requirements mark-complete ${REQ_IDS}
```

Extrair IDs de requisitos do frontmatter do plano (ex: `requirements: [AUTH-01, AUTH-02]`). Se sem campo requirements, pular.
</step>

<step name="git_commit_metadata">
Código das tarefas já commitado por tarefa. Commitar metadados do plano:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}-{plan}): concluir plano [nome-do-plano]" --files .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md .planning/REQUIREMENTS.md
```
</step>

<step name="update_codebase_map">
Se .planning/codebase/ não existir: pular.

```bash
FIRST_TASK=$(git log --oneline --grep="feat({phase}-{plan}):" --grep="fix({phase}-{plan}):" --grep="test({phase}-{plan}):" --reverse | head -1 | cut -d' ' -f1)
git diff --name-only ${FIRST_TASK}^..HEAD 2>/dev/null
```

Atualizar apenas mudanças estruturais: novo dir src/ → STRUCTURE.md | deps → STACK.md | padrão de arquivo → CONVENTIONS.md | cliente API → INTEGRATIONS.md | config → STACK.md | renomeado → atualizar caminhos. Pular mudanças somente-código/bugfix/conteúdo.

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "" --files .planning/codebase/*.md --amend
```
</step>

<step name="offer_next">
Se `USER_SETUP_CREATED=true`: exibir `⚠️ CONFIGURAÇÃO DO USUÁRIO NECESSÁRIA` com caminho + tarefas de env/config no TOPO.

```bash
ls -1 .planning/phases/[current-phase-dir]/*-PLAN.md 2>/dev/null | wc -l
ls -1 .planning/phases/[current-phase-dir]/*-SUMMARY.md 2>/dev/null | wc -l
```

| Condição | Rota | Ação |
|----------|------|------|
| summaries < plans | **A: Mais planos** | Encontrar próximo PLAN sem SUMMARY. Yolo: auto-continuar. Interativo: mostrar próximo plano, sugerir `/gsd-executar-fase {phase}` + `/gsd-verificar-trabalho`. PARAR aqui. |
| summaries = plans, atual < fase mais alta | **B: Fase concluída** | Mostrar conclusão, sugerir `/gsd-planejar-fase {Z+1}` + `/gsd-verificar-trabalho {Z}` + `/gsd-discutir-fase {Z+1}` |
| summaries = plans, atual = fase mais alta | **C: Marco concluído** | Mostrar banner, sugerir `/gsd-completar-marco` + `/gsd-verificar-trabalho` + `/gsd-adicionar-fase` |

Todas as rotas: `/clear` primeiro para contexto limpo.
</step>

</process>

<success_criteria>

- Todas as tarefas do PLAN.md completadas
- Todas as verificações passam
- USER-SETUP.md gerado se user_setup no frontmatter
- SUMMARY.md criado com conteúdo substantivo
- STATE.md atualizado (posição, decisões, problemas, sessão)
- ROADMAP.md atualizado
- Se mapa de codebase existir: mapa atualizado com mudanças de execução (ou pulado se sem mudanças significativas)
- Se USER-SETUP.md criado: apresentado proeminentemente na saída de conclusão
</success_criteria>
