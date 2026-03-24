<purpose>
Verificar progresso do projeto, resumir trabalho recente e o que está pela frente, depois rotear inteligentemente para a próxima ação — ou executar um plano existente ou criar o próximo. Fornece consciência situacional antes de continuar o trabalho.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="init_context">
**Carregar contexto de progresso (apenas caminhos):**

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init progress)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `project_exists`, `roadmap_exists`, `state_exists`, `phases`, `current_phase`, `next_phase`, `milestone_version`, `completed_count`, `phase_count`, `paused_at`, `state_path`, `roadmap_path`, `project_path`, `config_path`.

```bash
DISCUSS_MODE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.discuss_mode 2>/dev/null || echo "discuss")
```

Se `project_exists` for false (sem diretório `.planning/`):

```
Nenhuma estrutura de planejamento encontrada.

Execute /gsd-new-project para iniciar um novo projeto.
```

Sair.

Se STATE.md ausente: sugerir `/gsd-new-project`.

**Se ROADMAP.md ausente mas PROJECT.md existir:**

Isso significa que um marco foi concluído e arquivado. Ir para **Rota F** (entre marcos).

Se ambos ROADMAP.md e PROJECT.md ausentes: sugerir `/gsd-new-project`.
</step>

<step name="load">
**Usar extração estruturada do gsd-tools:**

Ao invés de ler arquivos completos, usar ferramentas direcionadas para obter apenas os dados necessários para o relatório:
- `ROADMAP=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)`
- `STATE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state-snapshot)`

Isso minimiza o uso de contexto do orquestrador.
</step>

<step name="analyze_roadmap">
**Obter análise abrangente do roteiro (substitui parsing manual):**

```bash
ROADMAP=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Isso retorna JSON estruturado com:
- Todas as fases com status de disco (complete/partial/planned/empty/no_directory)
- Objetivo e dependências por fase
- Contagens de planos e resumos por fase
- Estatísticas agregadas: total de planos, resumos, porcentagem de progresso
- Identificação da fase atual e próxima

Usar isso ao invés de ler/analisar ROADMAP.md manualmente.
</step>

<step name="recent">
**Coletar contexto de trabalho recente:**

- Encontrar os 2-3 arquivos SUMMARY.md mais recentes
- Usar `summary-extract` para parsing eficiente:
  ```bash
  node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" summary-extract <path> --fields one_liner
  ```
- Isso mostra "no que estivemos trabalhando"
  </step>

<step name="position">
**Analisar posição atual do contexto de init e análise do roteiro:**

- Usar `current_phase` e `next_phase` de `$ROADMAP`
- Notar `paused_at` se trabalho foi pausado (de `$STATE`)
- Contar todos pendentes: usar `init todos` ou `list-todos`
- Verificar sessões ativas de depuração: `ls .planning/debug/*.md 2>/dev/null | grep -v resolved | wc -l`
  </step>

<step name="report">
**Gerar barra de progresso do gsd-tools, depois apresentar relatório rico de status:**

```bash
# Obter barra de progresso formatada
PROGRESS_BAR=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" progress bar --raw)
```

Apresentar:

```
# [Nome do Projeto]

**Progresso:** {PROGRESS_BAR}
**Perfil:** [quality/balanced/budget/inherit]
**Modo de discussão:** {DISCUSS_MODE}

## Trabalho Recente
- [Fase X, Plano Y]: [o que foi realizado - 1 linha do summary-extract]
- [Fase X, Plano Z]: [o que foi realizado - 1 linha do summary-extract]

## Posição Atual
Fase [N] de [total]: [nome-da-fase]
Plano [M] de [total-da-fase]: [status]
CONTEXTO: [✓ se has_context | - se não]

## Decisões-Chave Tomadas
- [extrair de $STATE.decisions[]]
- [ex: jq -r '.decisions[].decision' do state-snapshot]

## Bloqueios/Preocupações
- [extrair de $STATE.blockers[]]
- [ex: jq -r '.blockers[].text' do state-snapshot]

## Todos Pendentes
- [contagem] pendentes — /gsd-check-todos para revisar

## Sessões de Depuração Ativas
- [contagem] ativas — /gsd-debug para continuar
(Mostrar esta seção apenas se contagem > 0)

## O Que Vem a Seguir
[Objetivo da próxima fase/plano do roadmap analyze]
```

</step>

<step name="route">
**Determinar próxima ação baseada em contagens verificadas.**

**Passo 1: Contar planos, resumos e problemas na fase atual**

Listar arquivos no diretório da fase atual:

```bash
ls -1 .planning/phases/[current-phase-dir]/*-PLAN.md 2>/dev/null | wc -l
ls -1 .planning/phases/[current-phase-dir]/*-SUMMARY.md 2>/dev/null | wc -l
ls -1 .planning/phases/[current-phase-dir]/*-UAT.md 2>/dev/null | wc -l
```

Declarar: "Esta fase tem {X} planos, {Y} resumos."

**Passo 1.5: Verificar lacunas de UAT não resolvidas**

Verificar arquivos UAT.md com status "diagnosed" (tem lacunas que precisam de correção).

```bash
# Verificar UAT diagnosticado com lacunas ou parcial (testes incompletos)
grep -l "status: diagnosed\|status: partial" .planning/phases/[current-phase-dir]/*-UAT.md 2>/dev/null
```

Rastrear:
- `uat_with_gaps`: Arquivos UAT.md com status "diagnosed" (lacunas precisam de correção)
- `uat_partial`: Arquivos UAT.md com status "partial" (testes incompletos)

**Passo 1.6: Verificação de saúde entre fases**

Examinar TODAS as fases no marco atual para débito de verificação pendente usando o CLI (que respeita limites de marco via `getMilestonePhaseFilter`):

```bash
DEBT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" audit-uat --raw 2>/dev/null)
```

Extrair do JSON `summary.total_items` e `summary.total_files`.

Rastrear: `outstanding_debt` — `summary.total_items` da auditoria.

**Se outstanding_debt > 0:** Adicionar seção de aviso à saída do relatório de progresso (no passo `report`), entre "## O Que Vem a Seguir" e a sugestão de rota:

```markdown
## Débito de Verificação ({N} arquivos em fases anteriores)

| Fase | Arquivo | Problema |
|------|---------|----------|
| {fase} | {filename} | {pending_count} pendentes, {skipped_count} pulados, {blocked_count} bloqueados |
| {fase} | {filename} | human_needed — {count} itens |

Revisar: `/gsd-audit-uat ${GSD_WS}` — auditoria completa entre fases
Retomar testes: `/gsd-verify-work {fase} ${GSD_WS}` — retestar fase específica
```

Isso é um AVISO, não um bloqueio — roteamento prossegue normalmente. O débito é visível para que o usuário possa fazer uma escolha informada.

**Passo 2: Rotear baseado em contagens**

| Condição | Significado | Ação |
|----------|-------------|------|
| uat_partial > 0 | Testes UAT incompletos | Ir para **Rota E.2** |
| uat_with_gaps > 0 | Lacunas UAT precisam de planos de correção | Ir para **Rota E** |
| resumos < planos | Planos não executados existem | Ir para **Rota A** |
| resumos = planos E planos > 0 | Fase concluída | Ir para Passo 3 |
| planos = 0 | Fase ainda não planejada | Ir para **Rota B** |

---

**Rota A: Plano não executado existe**

Encontrar o primeiro PLAN.md sem SUMMARY.md correspondente.
Ler sua seção `<objective>`.

```
---

## ▶ Próximo

**{fase}-{plano}: [Nome do Plano]** — [resumo do objetivo do PLAN.md]

`/gsd-execute-phase {fase} ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---
```

---

**Rota B: Fase precisa de planejamento**

Verificar se `{phase_num}-CONTEXT.md` existe no diretório da fase.

Verificar se a fase atual tem indicadores de UI:

```bash
PHASE_SECTION=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${CURRENT_PHASE}" 2>/dev/null)
PHASE_HAS_UI=$(echo "$PHASE_SECTION" | grep -qi "UI hint.*yes" && echo "true" || echo "false")
```

**Se CONTEXT.md existir:**

```
---

## ▶ Próximo

**Fase {N}: {Nome}** — {Objetivo do ROADMAP.md}
<sub>✓ Contexto coletado, pronto para planejar</sub>

`/gsd-plan-phase {número-fase} ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---
```

**Se CONTEXT.md NÃO existir E fase tiver UI (`PHASE_HAS_UI` é `true`):**

```
---

## ▶ Próximo

**Fase {N}: {Nome}** — {Objetivo do ROADMAP.md}

`/gsd-discuss-phase {fase}` — coletar contexto e esclarecer abordagem

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-ui-phase {fase}` — gerar contrato de design UI (recomendado para fases de frontend)
- `/gsd-plan-phase {fase}` — pular discussão, planejar diretamente
- `/gsd-list-phase-assumptions {fase}` — ver premissas do Claude

---
```

**Se CONTEXT.md NÃO existir E fase não tiver UI:**

```
---

## ▶ Próximo

**Fase {N}: {Nome}** — {Objetivo do ROADMAP.md}

`/gsd-discuss-phase {fase} ${GSD_WS}` — coletar contexto e esclarecer abordagem

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-plan-phase {fase} ${GSD_WS}` — pular discussão, planejar diretamente
- `/gsd-list-phase-assumptions {fase} ${GSD_WS}` — ver premissas do Claude

---
```

---

**Rota E: Lacunas UAT precisam de planos de correção**

UAT.md existe com lacunas (problemas diagnosticados). Usuário precisa planejar correções.

```
---

## ⚠ Lacunas UAT Encontradas

**{phase_num}-UAT.md** tem {N} lacunas que requerem correções.

`/gsd-plan-phase {fase} --gaps ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-execute-phase {fase} ${GSD_WS}` — executar planos da fase
- `/gsd-verify-work {fase} ${GSD_WS}` — executar mais testes UAT

---
```

---

**Rota E.2: Testes UAT incompletos (parcial)**

UAT.md existe com `status: partial` — sessão de testes terminou antes de todos os itens serem resolvidos.

```
---

## Testes UAT Incompletos

**{phase_num}-UAT.md** tem {N} testes não resolvidos (pendentes, bloqueados ou pulados).

`/gsd-verify-work {fase} ${GSD_WS}` — retomar testes de onde parou

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-audit-uat ${GSD_WS}` — auditoria completa de UAT entre fases
- `/gsd-execute-phase {fase} ${GSD_WS}` — executar planos da fase

---
```

---

**Passo 3: Verificar status do marco (apenas quando fase concluída)**

Ler ROADMAP.md e identificar:
1. Número da fase atual
2. Todos os números de fase na seção do marco atual

Contar total de fases e identificar o maior número de fase.

Declarar: "Fase atual é {X}. Marco tem {N} fases (maior: {Y})."

**Rotear baseado em status do marco:**

| Condição | Significado | Ação |
|----------|-------------|------|
| fase atual < maior fase | Mais fases restam | Ir para **Rota C** |
| fase atual = maior fase | Marco concluído | Ir para **Rota D** |

---

**Rota C: Fase concluída, mais fases restam**

Ler ROADMAP.md para obter nome e objetivo da próxima fase.

Verificar se próxima fase tem indicadores de UI:

```bash
NEXT_PHASE_SECTION=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "$((Z+1))" 2>/dev/null)
NEXT_HAS_UI=$(echo "$NEXT_PHASE_SECTION" | grep -qi "UI hint.*yes" && echo "true" || echo "false")
```

**Se próxima fase tiver UI (`NEXT_HAS_UI` é `true`):**

```
---

## ✓ Fase {Z} Concluída

## ▶ Próximo

**Fase {Z+1}: {Nome}** — {Objetivo do ROADMAP.md}

`/gsd-discuss-phase {Z+1}` — coletar contexto e esclarecer abordagem

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-ui-phase {Z+1}` — gerar contrato de design UI (recomendado para fases de frontend)
- `/gsd-plan-phase {Z+1}` — pular discussão, planejar diretamente
- `/gsd-verify-work {Z}` — teste de aceite antes de continuar

---
```

**Se próxima fase não tiver UI:**

```
---

## ✓ Fase {Z} Concluída

## ▶ Próximo

**Fase {Z+1}: {Nome}** — {Objetivo do ROADMAP.md}

`/gsd-discuss-phase {Z+1} ${GSD_WS}` — coletar contexto e esclarecer abordagem

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-plan-phase {Z+1} ${GSD_WS}` — pular discussão, planejar diretamente
- `/gsd-verify-work {Z} ${GSD_WS}` — teste de aceite antes de continuar

---
```

---

**Rota D: Marco concluído**

```
---

## 🎉 Marco Concluído

Todas as {N} fases finalizadas!

## ▶ Próximo

**Concluir Marco** — arquivar e preparar para o próximo

`/gsd-complete-milestone ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-verify-work ${GSD_WS}` — teste de aceite antes de concluir marco

---
```

---

**Rota F: Entre marcos (ROADMAP.md ausente, PROJECT.md existe)**

Um marco foi concluído e arquivado. Pronto para iniciar o próximo ciclo de marco.

Ler MILESTONES.md para encontrar a última versão de marco concluída.

```
---

## ✓ Marco v{X.Y} Concluído

Pronto para planejar o próximo marco.

## ▶ Próximo

**Iniciar Próximo Marco** — questionamento → pesquisa → requisitos → roteiro

`/gsd-new-milestone ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---
```

</step>

<step name="edge_cases">
**Tratar casos extremos:**

- Fase concluída mas próxima fase não planejada → oferecer `/gsd-plan-phase [próxima] ${GSD_WS}`
- Todo trabalho concluído → oferecer conclusão de marco
- Bloqueios presentes → destacar antes de oferecer continuação
- Arquivo de handoff existe → mencionar, oferecer `/gsd-resume-work ${GSD_WS}`
  </step>

</process>

<success_criteria>

- [ ] Contexto rico fornecido (trabalho recente, decisões, problemas)
- [ ] Posição atual clara com progresso visual
- [ ] O que vem a seguir claramente explicado
- [ ] Roteamento inteligente: /gsd-execute-phase se planos existem, /gsd-plan-phase se não
- [ ] Usuário confirma antes de qualquer ação
- [ ] Handoff transparente para comando gsd apropriado
      </success_criteria>
