<purpose>
Gerar um contrato de design UI (UI-SPEC.md) para fases de frontend. Orquestra gsd-ui-researcher e gsd-ui-checker com um loop de revisão. Insere-se entre discuss-phase e plan-phase no ciclo de vida.

UI-SPEC.md fixa espaçamento, tipografia, cor, copywriting e decisões de design system antes que o planejador crie tarefas. Isso previne débito de design causado por decisões de estilo ad-hoc durante a execução.
</purpose>

<required_reading>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</required_reading>

<process>

## 1. Inicializar

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar JSON para: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_context`, `has_research`, `commit_docs`.

**Caminhos de arquivo:** `state_path`, `roadmap_path`, `requirements_path`, `context_path`, `research_path`.

Resolver modelos dos agentes UI:

```bash
UI_RESEARCHER_MODEL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-ui-researcher --raw)
UI_CHECKER_MODEL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-ui-checker --raw)
```

Verificar config:

```bash
UI_ENABLED=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_phase 2>/dev/null || echo "true")
```

**Se `UI_ENABLED` for `false`:**
```
Fase UI está desabilitada na config. Habilite via /gsd-settings.
```
Sair do workflow.

**Se `planning_exists` for false:** Erro — execute `/gsd-new-project` primeiro.

## 2. Analisar e Validar Fase

Extrair número da fase de {{GSD_ARGS}}. Se não fornecido, detectar próxima fase sem plano.

```bash
PHASE_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

**Se `found` for false:** Erro com fases disponíveis.

## 3. Verificar Pré-requisitos

**Se `has_context` for false:**
```
Nenhum CONTEXT.md encontrado para Fase {N}.
Recomendado: execute /gsd-discuss-phase {N} primeiro para capturar preferências de design.
Continuando sem decisões do usuário — pesquisador de UI fará todas as perguntas.
```
Continuar (não bloqueante).

**Se `has_research` for false:**
```
Nenhum RESEARCH.md encontrado para Fase {N}.
Nota: decisões de stack (biblioteca de componentes, abordagem de estilo) serão perguntadas durante a pesquisa UI.
```
Continuar (não bloqueante).

## 4. Verificar UI-SPEC Existente

```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
```

**Se existir:** Use conversational prompting:
- header: "UI-SPEC Existente"
- question: "UI-SPEC.md já existe para Fase {N}. O que você gostaria de fazer?"
- options:
  - "Atualizar — re-executar pesquisador com existente como base"
  - "Visualizar — exibir UI-SPEC atual e sair"
  - "Pular — manter UI-SPEC atual, prosseguir para verificação"

Se "Visualizar": exibir conteúdo do arquivo, sair.
Se "Pular": prosseguir para passo 7 (checker).
Se "Atualizar": continuar para passo 5.

## 5. Criar gsd-ui-researcher

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CONTRATO DE DESIGN UI — FASE {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Criando pesquisador UI...
```

Construir prompt:

```markdown
Read D:/projetos/Estudo/devsquad/.cursor/agents/gsd-ui-researcher.md for instructions.

<objective>
Criar contrato de design UI para Fase {phase_number}: {phase_name}
Responder: "Quais contratos visuais e de interação esta fase precisa?"
</objective>

<files_to_read>
- {state_path} (Estado do Projeto)
- {roadmap_path} (Roadmap)
- {requirements_path} (Requisitos)
- {context_path} (DECISÕES DO USUÁRIO de /gsd-discuss-phase)
- {research_path} (Pesquisa Técnica — decisões de stack)
</files_to_read>

<output>
Write to: {phase_dir}/{padded_phase}-UI-SPEC.md
Template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/UI-SPEC.md
</output>

<config>
commit_docs: {commit_docs}
phase_dir: {phase_dir}
padded_phase: {padded_phase}
</config>
```

Omitir caminhos de arquivo nulos de `<files_to_read>`.

```
Task(
  prompt=ui_research_prompt,
  subagent_type="gsd-ui-researcher",
  model="{UI_RESEARCHER_MODEL}",
  description="Contrato de Design UI Fase {N}"
)
```

## 6. Tratar Retorno do Pesquisador

**Se `## UI-SPEC COMPLETE`:**
Exibir confirmação. Continuar para passo 7.

**Se `## UI-SPEC BLOCKED`:**
Exibir detalhes do bloqueio e opções. Sair do workflow.

## 7. Criar gsd-ui-checker

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO UI-SPEC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Criando verificador UI...
```

Construir prompt:

```markdown
Read D:/projetos/Estudo/devsquad/.cursor/agents/gsd-ui-checker.md for instructions.

<objective>
Validar contrato de design UI para Fase {phase_number}: {phase_name}
Verificar todas as 6 dimensões. Retornar APPROVED ou BLOCKED.
</objective>

<files_to_read>
- {phase_dir}/{padded_phase}-UI-SPEC.md (Contrato de Design UI — ENTRADA PRINCIPAL)
- {context_path} (DECISÕES DO USUÁRIO — verificar conformidade)
- {research_path} (Pesquisa Técnica — verificar alinhamento de stack)
</files_to_read>

<config>
ui_safety_gate: {valor config ui_safety_gate}
</config>
```

```
Task(
  prompt=ui_checker_prompt,
  subagent_type="gsd-ui-checker",
  model="{UI_CHECKER_MODEL}",
  description="Verificar UI-SPEC Fase {N}"
)
```

## 8. Tratar Retorno do Verificador

**Se `## UI-SPEC VERIFIED`:**
Exibir resultados das dimensões. Prosseguir para passo 10.

**Se `## ISSUES FOUND`:**
Exibir problemas bloqueantes. Prosseguir para passo 9.

## 9. Loop de Revisão (Máx 2 Iterações)

Rastrear `revision_count` (começa em 0).

**Se `revision_count` < 2:**
- Incrementar `revision_count`
- Re-criar gsd-ui-researcher com contexto de revisão:

```markdown
<revision>
O verificador UI encontrou problemas na UI-SPEC.md atual.

### Problemas a Corrigir
{colar problemas bloqueantes do retorno do verificador}

Leia a UI-SPEC.md existente, corrija APENAS os problemas listados, re-escreva o arquivo.
NÃO re-faça perguntas ao usuário que já foram respondidas.
</revision>
```

- Após retorno do pesquisador → re-criar verificador (passo 7)

**Se `revision_count` >= 2:**
```
Máximo de iterações de revisão atingido. Problemas restantes:

{listar problemas restantes}

Opções:
1. Forçar aprovação — prosseguir com UI-SPEC atual (FLAGs se tornam aceitos)
2. Editar manualmente — abrir UI-SPEC.md no editor, re-executar /gsd-ui-phase
3. Abandonar — sair sem aprovar
```

Use conversational prompting para a escolha.

## 10. Apresentar Status Final

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► UI-SPEC PRONTA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {N}: {Nome}** — Contrato de design UI aprovado

Dimensões: 6/6 aprovadas
{Se houver FLAGs: "Recomendações: {N} (não bloqueantes)"}

───────────────────────────────────────────────────────────────

## ▶ Próximo

**Planejar Fase {N}** — planejador usará UI-SPEC.md como contexto de design

`/gsd-plan-phase {N}`

<sub>/clear primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────
```

## 11. Commit (se configurado)

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): contrato de design UI" --files "${PHASE_DIR}/${PADDED_PHASE}-UI-SPEC.md"
```

## 12. Atualizar Estado

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "UI-SPEC da Fase ${PHASE} aprovada" \
  --resume-file "${PHASE_DIR}/${PADDED_PHASE}-UI-SPEC.md"
```

</process>

<success_criteria>
- [ ] Config verificada (sair se ui_phase desabilitada)
- [ ] Fase validada contra roadmap
- [ ] Pré-requisitos verificados (CONTEXT.md, RESEARCH.md — avisos não bloqueantes)
- [ ] UI-SPEC existente tratada (atualizar/visualizar/pular)
- [ ] gsd-ui-researcher criado com contexto e caminhos de arquivo corretos
- [ ] UI-SPEC.md criada na localização correta
- [ ] gsd-ui-checker criado com UI-SPEC.md
- [ ] Todas as 6 dimensões avaliadas
- [ ] Loop de revisão se BLOCKED (máx 2 iterações)
- [ ] Status final exibido com próximos passos
- [ ] UI-SPEC.md commitada (se commit_docs habilitado)
- [ ] Estado atualizado
</success_criteria>
