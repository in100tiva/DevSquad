<purpose>
Executar todos os planos de uma fase com execução paralela em ondas. O orquestrador permanece enxuto — delega a execução dos planos a subagentes.
</purpose>

<core_principle>
O orquestrador coordena, não executa. Cada subagente carrega o contexto completo de executar-plano. Orquestrador: descobrir planos → analisar dependências → agrupar ondas → disparar agentes → tratar pontos de verificação → coletar resultados.
</core_principle>

<runtime_compatibility>
**O disparo de subagentes é específico do runtime:**
- **Cursor:** Usa `Task(subagent_type="gsd-executor", ...)` — bloqueia até completar, retorna resultado
- **Copilot:** O disparo de subagente não retorna sinais de conclusão de forma confiável. **Usar execução sequencial inline como padrão**: ler e seguir executar-plano.md diretamente para cada plano
  ao invés de iniciar agentes paralelos. Só tente disparo paralelo se o usuário
  solicitar explicitamente — e nesse caso, confiar na verificação por amostragem do passo 3
  para detectar conclusão.
- **Outros runtimes:** Se ferramenta `Task`/`task` não estiver disponível, usar execução sequencial inline como
  fallback. Verificar disponibilidade da ferramenta em runtime ao invés de assumir baseado no nome do runtime.

**Regra de fallback:** Se um agente iniciado completou seu trabalho (commits visíveis, SUMMARY.md existe) mas
o orquestrador nunca recebeu o sinal de conclusão, tratá-lo como bem-sucedido baseado em verificações por amostragem
e continuar para a próxima onda/plano. Nunca bloquear indefinidamente esperando um sinal — sempre verificar
via sistema de arquivos e estado do git.
</runtime_compatibility>

<required_reading>
Ler STATE.md antes de qualquer operação para carregar contexto do projeto.
</required_reading>

<available_agent_types>
Estes são os tipos válidos de subagente GSD registrados em .claude/agents/ (ou equivalente para seu runtime).
Sempre usar o nome exato desta lista — não recorrer a 'general-purpose' ou outros tipos embutidos:

- gsd-executor — Executa tarefas do plano, commita, cria SUMMARY.md
- gsd-verificador — Verifica conclusão da fase, checa portões de qualidade
- gsd-planejador — Cria planos detalhados a partir do escopo da fase
- gsd-pesquisador-fase — Pesquisa abordagens técnicas para uma fase
- gsd-verificador-plano — Revisa qualidade do plano antes da execução
- gsd-depurador — Diagnostica e corrige problemas
- gsd-mapeador-codigo — Mapeia estrutura e dependências do projeto
- gsd-verificador-integracao — Verifica integração entre fases
- gsd-auditor-nyquist — Valida cobertura de verificação
- gsd-pesquisador-ui — Pesquisa abordagens de UI/UX
- gsd-verificador-ui — Revisa qualidade da implementação de UI
- gsd-auditor-ui — Audita UI contra requisitos de design
</available_agent_types>

<process>

<step name="parse_args" priority="first">
Analisar `{{GSD_ARGS}}` antes de carregar qualquer contexto:

- Primeiro token posicional → `PHASE_ARG`
- Opcional `--wave N` → `WAVE_FILTER`
- Opcional `--gaps-only` mantém seu significado atual

Se `--wave` estiver ausente, mantenha o comportamento atual de executar todas as ondas incompletas na fase.
</step>

<step name="initialize" priority="first">
Carregar todo o contexto em uma chamada:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `executor_model`, `verifier_model`, `commit_docs`, `parallelization`, `branching_strategy`, `branch_name`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `plans`, `incomplete_plans`, `plan_count`, `incomplete_count`, `state_exists`, `roadmap_exists`, `phase_req_ids`.

**Se `phase_found` for false:** Erro — diretório da fase não encontrado.
**Se `plan_count` for 0:** Erro — nenhum plano encontrado na fase.
**Se `state_exists` for false mas `.planning/` existir:** Oferecer reconstruir ou continuar.

Quando `parallelization` for false, os planos dentro de uma onda executam sequencialmente.

**Detecção de runtime para Copilot:**
Verificar se o runtime atual é Copilot testando o padrão de agente `@gsd-executor`
ou ausência da API de subagente `Task()`. Se executando sob Copilot, forçar execução sequencial inline
independente da configuração `parallelization` — sinais de conclusão de subagente do Copilot
são não confiáveis (veja `<runtime_compatibility>`). Definir `COPILOT_SEQUENTIAL=true`
internamente e pular o passo `execute_waves` em favor do caminho
inline de `check_interactive_mode` para cada plano.

**OBRIGATÓRIO — Sincronizar flag de cadeia com intenção.** Se o usuário invocou manualmente (sem `--auto`), limpar a flag efêmera de cadeia de qualquer cadeia `--auto` interrompida anteriormente. Isso previne que `_auto_chain_active: true` obsoleto cause auto-avanço indesejado. Isto NÃO toca em `workflow.auto_advance` (a preferência persistente do usuário nas configurações). Você DEVE executar este bloco bash antes de qualquer leitura de config:
```bash
# OBRIGATÓRIO: evita cadeia automática obsoleta de execuções --auto anteriores
if [[ ! "{{GSD_ARGS}}" =~ --auto ]]; then
  node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
fi
```
</step>

<step name="check_interactive_mode">
**Analisar flag `--interactive` de {{GSD_ARGS}}.**

**Se flag `--interactive` presente:** Mudar para modo de execução interativo.

O modo interativo executa planos sequencialmente **inline** (sem disparo de subagente) com
pontos de verificação do usuário entre tarefas. O usuário pode revisar, modificar ou redirecionar o trabalho a qualquer momento.

**Fluxo de execução interativo:**

1. Carregar inventário de planos normalmente (discover_and_group_plans)
2. Para cada plano (sequencialmente, ignorando agrupamento por onda):

   a. **Apresentar o plano ao usuário:**
      ```
      ## Plano {plan_id}: {plan_name}

      Objetivo: {do arquivo do plano}
      Tarefas: {task_count}

      Opções:
      - Executar (prosseguir com todas as tarefas)
      - Revisar primeiro (mostrar detalhamento de tarefas antes de começar)
      - Pular (mover para o próximo plano)
      - Parar (encerrar execução, salvar progresso)
      ```

   b. **Se "Revisar primeiro":** Ler e exibir o arquivo completo do plano. Perguntar novamente: Executar, Modificar, Pular.

   c. **Se "Executar":** Ler e seguir `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/executar-plano.md` **inline**
      (NÃO iniciar um subagente). Executar tarefas uma por vez.

   d. **Após cada tarefa:** Pausar brevemente. Se o usuário intervir (digitar qualquer coisa), parar e tratar
      o feedback antes de continuar. Caso contrário prosseguir para a próxima tarefa.

   e. **Após plano completo:** Mostrar resultados, fazer commit, criar SUMMARY.md, depois apresentar o próximo plano.

3. Após todos os planos: prosseguir para verificação (igual ao modo normal).

**Benefícios do modo interativo:**
- Sem custo extra de subagente — uso de tokens muito menor
- Usuário detecta erros cedo — economiza ciclos custosos de verificação
- Mantém a estrutura de planejamento/rastreamento do GSD
- Melhor para: fases pequenas, correções de bugs, lacunas de verificação, aprender o GSD

**Pular para o passo handle_branching** (planos interativos executam inline após agrupamento).
</step>

<step name="handle_branching">
Verificar `branching_strategy` do init:

**"none":** Pular, continuar no ramo (branch) atual.

**"phase" ou "milestone":** Usar `branch_name` pré-computado do init:
```bash
git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
```

Todos os commits subsequentes vão para este ramo (branch). O usuário gerencia o merge.
</step>

<step name="validate_phase">
Do JSON de init: `phase_dir`, `plan_count`, `incomplete_count`.

Relatar: "Encontrados {plan_count} planos em {phase_dir} ({incomplete_count} incompletos)"

**Atualizar STATE.md para início da fase:**
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state begin-phase --phase "${PHASE_NUMBER}" --name "${PHASE_NAME}" --plans "${PLAN_COUNT}"
```
Isso atualiza Status, Última Atividade, Foco atual, Posição Atual e contagens de planos no STATE.md para que frontmatter e texto do corpo reflitam a fase ativa imediatamente.
</step>

<step name="discover_and_group_plans">
Carregar inventário de planos com agrupamento por onda em uma chamada:

```bash
PLAN_INDEX=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase-plan-index "${PHASE_NUMBER}")
```

Analisar JSON para: `phase`, `plans[]` (cada um com `id`, `wave`, `autonomous`, `objective`, `files_modified`, `task_count`, `has_summary`), `waves` (mapa de número da onda → IDs de plano), `incomplete`, `has_checkpoints`.

**Filtragem:** Pular planos onde `has_summary: true`. Se `--gaps-only`: também pular planos que não são gap_closure. Se `WAVE_FILTER` estiver definido: também pular planos cuja `wave` não seja igual a `WAVE_FILTER`.

**Verificação de segurança da onda:** Se `WAVE_FILTER` estiver definido e ainda houver planos incompletos em qualquer onda inferior que correspondam ao modo de execução atual, PARE e informe o usuário de terminar ondas anteriores primeiro. Não permita que a Onda 2+ execute enquanto planos pré-requisito de ondas anteriores permanecerem incompletos.

Se todos filtrados: "Nenhum plano incompleto correspondente" → sair.

Reportar:
```
## Plano de Execução

**Fase {X}: {Nome}** — {total_plans} planos correspondentes em {wave_count} onda(s)

{Se WAVE_FILTER estiver definido: `Filtro de onda ativo: executando apenas a Onda {WAVE_FILTER}`.}

| Onda | Planos | O que constrói |
|------|--------|----------------|
| 1 | 01-01, 01-02 | {dos objetivos dos planos, 3-8 palavras} |
| 2 | 01-03 | ... |
```
</step>

<step name="execute_waves">
Executar cada onda selecionada em sequência. Dentro de uma onda: paralelo se `PARALLELIZATION=true`, sequencial se `false`.

**Para cada onda:**

1. **Descrever o que está sendo construído (ANTES do disparo):**

   Ler o `<objective>` de cada plano. Extrair o que está sendo construído e por quê.

   ```
   ---
   ## Onda {N}

   **{Plan ID}: {Nome do Plano}**
   {2-3 frases: o que constrói, abordagem técnica, por que importa}

   Iniciando {count} agente(s)...
   ---
   ```

   - Ruim: "Executando plano de geração de terreno"
   - Bom: "Gerador de terreno procedural usando ruído Perlin — cria mapas de altura, zonas de bioma e meshes de colisão. Necessário antes que a física de veículos possa interagir com o chão."

2. **Iniciar agentes executores:**

   Passar apenas caminhos — executores leem os arquivos com sua janela de contexto limpa.
   Para modelos de 200k, isso mantém o contexto do orquestrador enxuto (~10-15%).
   Para modelos de 1M+ (Opus 4.6, Sonnet 4.6), contexto mais rico pode ser passado diretamente.

   ```
   Task(
     subagent_type="gsd-executor",
     model="{executor_model}",
     isolation="worktree",
     prompt="
       <objective>
       Executar plano {plan_number} da fase {phase_number}-{phase_name}.
       Fazer commit de cada tarefa de forma atômica. Criar SUMMARY.md. Atualizar STATE.md e ROADMAP.md.
       </objective>

       <parallel_execution>
       Você está executando como agente executor PARALELO. Use --no-verify em todos os
       commits git para evitar contenção de hook pre-commit com outros agentes. O
       orquestrador valida hooks uma vez após todos os agentes completarem.
       Para commits do gsd-tools: adicionar flag --no-verify.
       Para commits git diretos: usar git commit --no-verify -m "..."
       </parallel_execution>

       <execution_context>
       @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/executar-plano.md
       @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/resumo.md
       @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/pontos-verificacao.md
       @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/tdd.md
       </execution_context>

       <files_to_read>
       Ler estes arquivos no início da execução usando a ferramenta Read:
       - {phase_dir}/{plan_file} (Plano)
       - .planning/PROJECT.md (Contexto do projeto — valor central, requisitos, regras de evolução)
       - .planning/STATE.md (Estado)
       - .planning/config.json (Config, se existir)
       - .cursor/rules/ (Instruções do projeto, se existir — seguir diretrizes e convenções de código específicas do projeto)
       - .cursor/skills/ ou .agents/skills/ (Skills do projeto, se algum existir — listar skills, ler SKILL.md de cada, seguir regras relevantes durante implementação)
       </files_to_read>

       <mcp_tools>
       Se .cursor/rules/ ou instruções do projeto referenciam ferramentas MCP (ex: jCodeMunch, context7,
       ou outros servidores MCP), preferir essas ferramentas sobre Grep/Glob para navegação de código quando disponíveis.
       Ferramentas MCP frequentemente economizam tokens significativos fornecendo índices de código estruturados.
       Verificar disponibilidade da ferramenta primeiro — se ferramentas MCP não estiverem acessíveis, recorrer a Grep/Glob.
       </mcp_tools>

       <success_criteria>
       - [ ] Todas as tarefas executadas
       - [ ] Cada tarefa com commit individual
       - [ ] SUMMARY.md criado no diretório do plano
       - [ ] STATE.md atualizado com posição e decisões
       - [ ] ROADMAP.md atualizado com progresso do plano (via `roadmap update-plan-progress`)
       </success_criteria>
     "
   )
   ```

3. **Aguardar todos os agentes na onda concluírem.**

   **Fallback de sinal de conclusão (Copilot e runtimes onde Task() pode não retornar):**

   Se um agente iniciado não retornar sinal de conclusão mas parecer ter terminado
   seu trabalho, NÃO bloquear indefinidamente. Em vez disso, verificar conclusão via amostragem:

   ```bash
   # Para cada plano nesta onda, verificar se o executor terminou:
   SUMMARY_EXISTS=$(test -f "{phase_dir}/{plan_number}-{plan_padded}-SUMMARY.md" && echo "true" || echo "false")
   COMMITS_FOUND=$(git log --oneline --all --grep="{phase_number}-{plan_padded}" --since="1 hour ago" | head -1)
   ```

   **Se SUMMARY.md existe E commits encontrados:** O agente completou com sucesso —
   tratar como concluído e prosseguir para o passo 4. Registrar: `"✓ {Plan ID} completado (verificado via amostragem — sinal de conclusão não recebido)"`

   **Se SUMMARY.md NÃO existir após espera razoável:** O agente pode ainda estar
   executando ou pode ter falhado silenciosamente. Verificar `git log --oneline -5` para
   atividade recente. Se commits ainda aparecem, esperar mais. Se sem atividade, reportar
   o plano como falho e encaminhar ao tratamento de falhas no passo 5.

   **Este fallback aplica-se automaticamente a todos os runtimes.** O Task() do Cursor normalmente
   retorna sincronamente, mas o fallback garante resiliência caso não retorne.

4. **Validação de hooks pós-onda (somente modo paralelo):**

   Quando agentes commitaram com `--no-verify`, rode os hooks pre-commit uma vez após a onda:
   ```bash
   # Executa os hooks pre-commit do projeto no estado atual
   git diff --cached --quiet || git stash
   git hook run pre-commit 2>&1 || echo "⚠ Hooks pre-commit falharam — revisar antes de continuar"
   ```
   Se os hooks falharem: relate a falha e pergunte "Corrigir problemas de hooks agora?" ou "Continuar para a próxima onda?"

5. **Reportar conclusão — verificar por amostragem primeiro:**

   Para cada SUMMARY.md:
   - Verificar que os primeiros 2 arquivos de `key-files.created` existem no disco
   - Verificar que `git log --oneline --all --grep="{phase}-{plan}"` retorna ≥1 commit
   - Verificar marcador `## Self-Check: FAILED`

   Se QUALQUER verificação falhar: relate qual plano falhou, encaminhe ao tratamento de falhas — pergunte "Repetir o plano?" ou "Continuar com as ondas restantes?"

   Se aprovado:
   ```
   ---
   ## Onda {N} concluída

   **{Plan ID}: {Nome do Plano}**
   {O que foi construído — do SUMMARY.md}
   {Desvios notáveis, se houver}

   {Se houver mais ondas: o que isso habilita para a próxima onda}
   ---
   ```

   - Ruim: "Onda 2 concluída. Seguindo para a Onda 3."
   - Bom: "Sistema de terreno completo — 3 tipos de bioma, texturização por altura, malhas de colisão para física. A física de veículos (Onda 3) pode agora referenciar superfícies do chão."

5. **Tratar falhas:**

   Para falhas reais: relate qual plano falhou → pergunte "Continuar?" ou "Parar?" → se continuar, planos dependentes também podem falhar. Se parar, relatório de conclusão parcial.

5b. **Verificação de dependência pré-onda (somente ondas 2+):**

    Antes de iniciar a onda N+1, para cada plano na onda que vem:
    ```bash
    node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" verify key-links {phase_dir}/{plan}-PLAN.md
    ```

    Se algum key-link de artefato de onda ANTERIOR falhar na verificação:

    ## Lacuna de Conexão Entre Planos

    | Plano | Ligação | Origem | Padrão esperado | Status |
    |-------|---------|--------|-----------------|--------|
    | {plan} | {via} | {origem} | {pattern} | NÃO ENCONTRADO |

    Artefatos da Onda {N} podem não estar devidamente integrados. Opções:
    1. Investigar e corrigir antes de continuar
    2. Continuar (pode causar falhas em cascata na onda {N+1})

    Key-links que referenciam arquivos na onda ATUAL (que vem) são ignorados.

6. **Executar planos de ponto de verificação entre ondas** — veja `<checkpoint_handling>`.

7. **Prosseguir para a próxima onda.**
</step>

<step name="checkpoint_handling">
Planos com `autonomous: false` requerem interação do usuário.

**Tratamento de ponto de verificação em modo automático:**

Ler config de auto-avanço (flag de cadeia + preferência do usuário):
```bash
AUTO_CHAIN=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
AUTO_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
```

Quando o executor retorna um ponto de verificação E (`AUTO_CHAIN` for `"true"` OU `AUTO_CFG` for `"true"`):
- **human-verify** → Disparar automaticamente agente de continuação com `{user_response}` = `"approved"`. Registrar `⚡ Ponto de verificação aprovado automaticamente`.
- **decision** → Disparar automaticamente agente de continuação com `{user_response}` = primeira opção dos detalhes do ponto de verificação. Registrar `⚡ Selecionado automaticamente: [opção]`.
- **human-action** → Apresentar ao usuário (comportamento existente abaixo). Portais de auth não podem ser automatizados.

**Fluxo padrão (não modo automático, ou tipo human-action):**

1. Disparar agente para o plano de ponto de verificação
2. O agente executa até a tarefa de ponto de verificação ou portal de auth → retorna estado estruturado
3. O retorno do agente inclui: tabela de tarefas concluídas, tarefa atual + bloqueio, tipo/detalhes do ponto de verificação, o que está aguardando
4. **Apresentar ao usuário:**
   ```
   ## Ponto de verificação: [Tipo]

   **Plano:** 03-03 Layout do Dashboard
   **Progresso:** 2/3 tarefas concluídas

   [Detalhes do ponto de verificação no retorno do agente]
   [Seção Aguardando do retorno do agente]
   ```
5. O usuário responde: "approved"/"done" | descrição do problema | seleção de decisão
6. **Disparar agente de continuação (NÃO retomar)** usando o template continuation-prompt.md:
   - `{completed_tasks_table}`: Do retorno do ponto de verificação
   - `{resume_task_number}` + `{resume_task_name}`: Tarefa atual
   - `{user_response}`: O que o usuário forneceu
   - `{resume_instructions}`: Com base no tipo de ponto de verificação
7. Agente de continuação verifica commits anteriores, continua do ponto de retomada
8. Repetir até plano completar ou usuário parar

**Por que agente novo, não retomar:** Retomar depende de serialização interna que quebra com chamadas de ferramenta paralelas. Agentes novos com estado explícito são mais confiáveis.

**Pontos de verificação em ondas paralelas:** O agente pausa e retorna enquanto outros agentes paralelos podem concluir. Apresente o ponto de verificação, dispare a continuação, espere todos antes da próxima onda.
</step>

<step name="aggregate_results">
Após todas as ondas:

```markdown
## Fase {X}: {Nome} — execução concluída

**Ondas:** {N} | **Planos:** {M}/{total} concluídos

| Onda | Planos | Status |
|------|--------|--------|
| 1 | plano-01, plano-02 | ✓ Completo |
| CP | plano-03 | ✓ Verificado |
| 2 | plano-04 | ✓ Completo |

### Detalhes dos Planos
1. **03-01**: [resumo de uma linha do SUMMARY.md]
2. **03-02**: [resumo de uma linha do SUMMARY.md]

### Problemas Encontrados
[Agregar dos SUMMARYs, ou "Nenhum"]
```
</step>

<step name="handle_partial_wave_execution">
Se `WAVE_FILTER` foi usado, re-executar descoberta de planos após execução:

```bash
POST_PLAN_INDEX=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase-plan-index "${PHASE_NUMBER}")
```

Aplicar as mesmas regras de filtragem "incompleto" de antes:
- ignorar planos com `has_summary: true`
- se `--gaps-only`, considerar apenas planos `gap_closure: true`

**Se planos incompletos ainda restarem em qualquer lugar na fase:**
- PARAR aqui
- NÃO executar verificação de fase
- NÃO marcar a fase como completa em ROADMAP/STATE
- Apresentar:

```markdown
## Onda {WAVE_FILTER} concluída

A onda selecionada terminou com sucesso. Esta fase ainda tem planos incompletos, então a verificação e a conclusão em nível de fase foram intencionalmente ignoradas.

/gsd-executar-fase {phase} ${GSD_WS}                # Continuar ondas restantes
/gsd-executar-fase {phase} --wave {next} ${GSD_WS}  # Executar a próxima onda explicitamente
```

**Se nenhum plano incompleto restar após a onda selecionada terminar:**
- prossiga com o fluxo normal de verificação e conclusão em nível de fase abaixo
- isto significa que a onda selecionada era, por acaso, o último trabalho restante na fase
</step>

<step name="close_parent_artifacts">
**Somente para fases decimais/de polimento (padrão X.Y):** Fechar o loop de feedback resolvendo artefatos TAU e de depuração do pai.

**Pular se** número da fase não tem decimal (ex: `3`, `04`) — aplica-se apenas a fases de fechamento de lacunas como `4.1`, `03.1`.

**1. Detectar fase decimal e derivar pai:**
```bash
# Verificar se phase_number contém decimal
if [[ "$PHASE_NUMBER" == *.* ]]; then
  PARENT_PHASE="${PHASE_NUMBER%%.*}"
fi
```

**2. Encontrar arquivo TAU do pai:**
```bash
PARENT_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" find-phase "${PARENT_PHASE}" --raw)
```

**Se nenhum TAU pai encontrado:** Pular este passo (fechamento de lacuna pode ter sido acionado por VERIFICATION.md).

**3. Atualizar status de lacunas no TAU:**

Ler seção `## Gaps` do arquivo TAU pai. Para cada entrada de lacuna com `status: failed`:
- Atualizar para `status: resolved`

**4. Atualizar frontmatter do TAU:**

Se todas as lacunas agora têm `status: resolved`:
- Atualizar frontmatter `status: diagnosed` → `status: resolved`
- Atualizar timestamp `updated:` do frontmatter

**5. Resolver sessões de depuração referenciadas:**

Para cada lacuna que tem campo `debug_session:`:
- Ler o arquivo de sessão de depuração
- Atualizar frontmatter `status:` → `resolved`
- Atualizar timestamp `updated:` do frontmatter
- Mover para diretório resolvido:
```bash
mkdir -p .planning/debug/resolved
mv .planning/debug/{slug}.md .planning/debug/resolved/
```

**6. Fazer commit dos artefatos atualizados:**
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PARENT_PHASE}): resolver lacunas TAU e sessões de depuração após fechamento de lacuna ${PHASE_NUMBER}" --files .planning/phases/*${PARENT_PHASE}*/*-UAT.md .planning/debug/resolved/*.md
```
</step>

<step name="regression_gate">
Executar suítes de teste de fases anteriores para capturar regressões entre fases ANTES da verificação.

**Pular se:** Esta é a primeira fase (sem fases anteriores), ou sem arquivos VERIFICATION.md anteriores.

**Passo 1: Descobrir arquivos de teste de fases anteriores**
```bash
# Encontrar todos os arquivos VERIFICATION.md de fases anteriores no marco atual
PRIOR_VERIFICATIONS=$(find .planning/phases/ -name "*-VERIFICATION.md" ! -path "*${PHASE_NUMBER}*" 2>/dev/null)
```

**Passo 2: Extrair listas de arquivos de teste de verificações anteriores**

Para cada VERIFICATION.md encontrado, procurar referências de arquivos de teste:
- Linhas contendo caminhos `test`, `spec` ou `__tests__`
- Seções com títulos em inglês comuns como "Test Suite" (suíte de testes) ou "Automated Checks" (verificações automatizadas)
- Padrões de arquivo de `key-files.created` em arquivos SUMMARY.md correspondentes que correspondam a `*.test.*` ou `*.spec.*`

Coletar todos os caminhos únicos de arquivos de teste em `REGRESSION_FILES`.

**Passo 3: Executar testes de regressão (se algum encontrado)**

```bash
# Detectar executor de testes e rodar testes de fases anteriores
if [ -f "package.json" ]; then
  # Node.js — usar o executor de testes do projeto
  npx jest ${REGRESSION_FILES} --passWithNoTests --no-coverage -q 2>&1 || npx vitest run ${REGRESSION_FILES} 2>&1
elif [ -f "Cargo.toml" ]; then
  cargo test 2>&1
elif [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
  python -m pytest ${REGRESSION_FILES} -q --tb=short 2>&1
fi
```

**Passo 4: Reportar resultados**

Se todos os testes passarem:
```
✓ Portal de regressão: {N} arquivos de teste de fases anteriores aprovados — nenhuma regressão detectada
```
→ Prosseguir para o passo `verify_phase_goal`

Se algum teste falhar:
```
## ⚠ Regressão Entre Fases Detectada

Execução da Fase {X} pode ter quebrado funcionalidade de fases anteriores.

| Arquivo de Teste | Fase | Status | Detalhe |
|------------------|------|--------|---------|
| {file} | {origin_phase} | FALHOU | {first_failure_line} |

Opções:
1. Corrigir regressões antes da verificação (recomendado)
2. Continuar para verificação mesmo assim (regressões se acumularão)
3. Abortar fase — reverter e re-planejar
```

Use diálogo conversacional para apresentar as opções.
</step>

<step name="verify_phase_goal">
Verificar se a fase alcançou seu OBJETIVO, não apenas completou tarefas.

```
Task(
  prompt="Verificar se a fase {phase_number} alcançou seu objetivo.
Diretório da fase: {phase_dir}
Objetivo da fase: {objetivo no ROADMAP.md}
IDs de requisitos da fase: {phase_req_ids}
Conferir must_haves contra o código real.
Cruzar IDs de requisito do frontmatter dos PLAN.md com REQUIREMENTS.md — todo ID deve estar contabilizado.
Criar VERIFICATION.md.",
  subagent_type="gsd-verificador",
  model="{verifier_model}"
)
```

Ler status:
```bash
grep "^status:" "$PHASE_DIR"/*-VERIFICATION.md | cut -d: -f2 | tr -d ' '
```

| Status | Ação |
|--------|------|
| `passed` | → update_roadmap |
| `human_needed` | Apresentar itens para teste humano, obter aprovação ou feedback |
| `gaps_found` | Apresentar resumo de lacunas, oferecer `/gsd-planejar-fase {phase} --gaps ${GSD_WS}` |

**Se human_needed:**

**Passo A: Persistir itens de verificação humana como arquivo TAU.**

Criar `{phase_dir}/{phase_num}-HUMAN-UAT.md` usando o formato do template TAU:

```markdown
---
status: partial
phase: {phase_num}-{phase_name}
source: [{phase_num}-VERIFICATION.md]
started: [agora ISO]
updated: [agora ISO]
---

## Teste Atual

[aguardando teste humano]

## Testes

{Para cada item human_verification do VERIFICATION.md:}

### {N}. {descrição do item}
expected: {comportamento esperado do VERIFICATION.md}
result: [pendente]

## Resumo

total: {contagem}
passed: 0
issues: 0
pending: {contagem}
skipped: 0
blocked: 0

## Lacunas
```

Fazer commit do arquivo:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "test({phase_num}): persistir itens de verificação humana como TAU" --files "{phase_dir}/{phase_num}-HUMAN-UAT.md"
```

**Passo B: Apresentar ao usuário:**

```
## ✓ Fase {X}: {Nome} — Verificação Humana Necessária

Todas as verificações automatizadas passaram. {N} itens precisam de teste humano:

{Da seção human_verification do VERIFICATION.md}

Itens salvos em `{phase_num}-HUMAN-UAT.md` — aparecerão em `/gsd-progresso` e `/gsd-auditar-tau`.

"approved" → continuar | Relatar problemas → fechamento de lacunas
```

**Se o usuário disser "approved":** Prosseguir para `update_roadmap`. O arquivo HUMAN-UAT.md permanece com `status: partial` e surgirá em checagens de progresso futuras até o usuário executar `/gsd-verificar-trabalho` nele.

**Se o usuário relatar problemas:** Prosseguir para o fechamento de lacunas como já implementado.

**Se gaps_found:**
```
## ⚠ Fase {X}: {Nome} — Lacunas Encontradas

**Pontuação:** {N}/{M} obrigatórios verificados
**Relatório:** {phase_dir}/{phase_num}-VERIFICATION.md

### O que falta
{Resumos de lacunas do VERIFICATION.md}

---
## ▶ Próximo

`/gsd-planejar-fase {X} --gaps ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

Também: `cat {phase_dir}/{phase_num}-VERIFICATION.md` — relatório completo
Também: `/gsd-verificar-trabalho {X} ${GSD_WS}` — teste manual primeiro
```

Ciclo de fechamento de lacunas: `/gsd-planejar-fase {X} --gaps ${GSD_WS}` lê VERIFICATION.md → cria planos de lacuna com `gap_closure: true` → usuário executa `/gsd-executar-fase {X} --gaps-only ${GSD_WS}` → verificador re-executa.
</step>

<step name="update_roadmap">
**Marcar fase como completa e atualizar todos os arquivos de rastreamento:**

```bash
COMPLETION=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase complete "${PHASE_NUMBER}")
```

A interface de linha de comando (CLI) trata:
- Marcar checkbox da fase `[x]` com data de conclusão
- Atualizar tabela de Progresso (Status → Completo, data)
- Atualizar contagem de planos para final
- Avançar STATE.md para próxima fase
- Atualizar rastreabilidade do REQUIREMENTS.md
- Verificar dívida de verificação (retorna array `warnings`)

Extrair do resultado: `next_phase`, `next_phase_name`, `is_last_phase`, `warnings`, `has_warnings`.

**Se has_warnings for true:**
```
## Fase {X} marcada como completa com {N} avisos:

{listar cada aviso}

Estes itens estão rastreados e aparecerão em `/gsd-progresso` e `/gsd-auditar-tau`.
```

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-{X}): concluir execução da fase" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md {phase_dir}/*-VERIFICATION.md
```
</step>

<step name="update_project_md">
**Evoluir PROJECT.md para refletir a conclusão da fase (evita deriva silenciosa do documento de planejamento — #956):**

PROJECT.md rastreia requisitos validados, decisões e estado atual. Sem este passo,
PROJECT.md fica defasado silenciosamente ao longo de múltiplas fases.

1. Ler `.planning/PROJECT.md`
2. Se o arquivo existir e tiver seção `## Validated Requirements` ou `## Requirements`:
   - Mover quaisquer requisitos validados por esta fase de Ativos → Validados
   - Adicionar nota breve: `Validado na Fase {X}: {Nome}`
3. Se o arquivo tiver seção `## Current State` ou similar:
   - Atualizar para refletir a conclusão desta fase (ex: "Fase {X} completa — {resumo de uma linha}")
4. Atualizar rodapé `Last updated:` para a data de hoje
5. Commitar a alteração:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-{X}): evoluir PROJECT.md após conclusão da fase" --files .planning/PROJECT.md
```

**Pular este passo se** `.planning/PROJECT.md` não existir.
</step>

<step name="offer_next">

**Exceção:** Se `gaps_found`, o passo `verify_phase_goal` já apresenta o caminho de fechamento de lacunas (`/gsd-planejar-fase {X} --gaps`). Nenhum roteamento adicional necessário — pular auto-avanço.

**Verificação de não-transição (iniciado por cadeia de auto-avanço):**

Analisar flag `--no-transition` de {{GSD_ARGS}}.

**Se flag `--no-transition` presente:**

executar-fase foi disparado pelo auto-avanço de planejar-fase. NÃO executar transicao.md.
Após a verificação passar e o roadmap ser atualizado, retorne o status de conclusão ao processo pai:

```
## FASE CONCLUÍDA

Fase: ${PHASE_NUMBER} - ${PHASE_NAME}
Planos: ${completed_count}/${total_count}
Verificação: {Aprovada | Lacunas encontradas}

[Incluir saída de aggregate_results]
```

PARAR. Não prosseguir para auto-avanço ou transição.

**Se flag `--no-transition` NÃO presente:**

**Detecção de auto-avanço:**

1. Analisar flag `--auto` de {{GSD_ARGS}}
2. Ler tanto a flag de cadeia quanto a preferência do usuário (flag de cadeia já sincronizada no passo de init):
   ```bash
   AUTO_CHAIN=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se flag `--auto` presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true (E verificação aprovada sem lacunas):**

```
╔══════════════════════════════════════════╗
║  AUTO-AVANÇANDO → TRANSIÇÃO              ║
║  Fase {X} verificada, continuando cadeia ║
╚══════════════════════════════════════════╝
```

Executar o workflow de transição inline (NÃO usar Task — contexto do orquestrador está em ~10-15%, transição precisa de dados de conclusão da fase já em contexto):

Ler e seguir `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/transicao.md`, passando a flag `--auto` para que propague para a próxima invocação de fase.

**Se nenhum de `--auto`, `AUTO_CHAIN` ou `AUTO_CFG` for true:**

**PARAR. Não auto-avançar. Não executar transição. Não planejar próxima fase. Apresentar opções ao usuário e aguardar.**

**IMPORTANTE: Não existe comando `/gsd-transicao`. Nunca sugeri-lo. O workflow de transição é somente interno.**

```
## ✓ Fase {X}: {Nome} concluída

/gsd-progresso ${GSD_WS} — ver roadmap atualizado
/gsd-discutir-fase {next} ${GSD_WS} — discutir próxima fase antes de planejar
/gsd-planejar-fase {next} ${GSD_WS} — planejar próxima fase
/gsd-executar-fase {next} ${GSD_WS} — executar próxima fase
```

Sugerir apenas os comandos listados acima. Não inventar ou alucinar nomes de comandos.
</step>

</process>

<context_efficiency>
Orquestrador: ~10-15% de contexto para janelas de 200k, pode usar mais para janelas de 1M+.
Subagentes: contexto novo a cada execução (200k-1M conforme o modelo). Sem sondagem periódica (Task bloqueia). Sem vazamento de contexto entre agentes.

Para modelos de contexto 1M+, considere:
- Passar contexto mais rico (trechos de código, saídas de dependências) diretamente aos executores em vez de apenas caminhos de arquivo
- Executar fases pequenas (≤3 planos, sem dependências) inline sem custo extra de disparo de subagente
- Relaxar recomendações de /clear — o início da degradação de contexto fica muito mais distante com janela 5x
</context_efficiency>

<failure_handling>
- **Agente falha no meio do plano:** SUMMARY.md ausente → relatar, perguntar ao usuário como prosseguir
- **Cadeia de dependências quebra:** Onda 1 falha → dependentes da Onda 2 provavelmente falham → usuário escolhe tentar ou pular
- **Todos os agentes na onda falham:** Problema sistêmico → parar, relatar para investigação
- **Ponto de verificação irresolvível:** "Pular este plano?" ou "Abortar execução da fase?" → registrar progresso parcial em STATE.md
</failure_handling>

<resumption>
Execute novamente `/gsd-executar-fase {phase}` → a descoberta de planos encontra SUMMARYs já concluídos → pula-os → retoma do primeiro plano incompleto → continua a execução em ondas.

STATE.md rastreia: último plano concluído, onda atual, pontos de verificação pendentes.
</resumption>
