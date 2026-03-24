<purpose>
Auditoria visual retroativa de 6 pilares do código frontend implementado. Comando independente que funciona em qualquer projeto — gerenciado pelo GSD ou não. Produz UI-REVIEW.md com pontuação e achados acionáveis.
</purpose>

<required_reading>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</required_reading>

<process>

## 0. Inicializar

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `commit_docs`.

```bash
UI_AUDITOR_MODEL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-auditor-ui --raw)
```

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUDITORIA UI — FASE {N}: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 1. Detectar Estado de Entrada

```bash
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
UI_REVIEW_FILE=$(ls "${PHASE_DIR}"/*-UI-REVIEW.md 2>/dev/null | head -1)
```

**Se `SUMMARY_FILES` vazio:** Sair — "Fase {N} não executada. Execute /gsd-executar-fase {N} primeiro."

**Se `UI_REVIEW_FILE` não vazio:** Use conversational prompting:
- header: "Revisão UI Existente"
- question: "UI-REVIEW.md já existe para Fase {N}."
- options:
  - "Re-auditar — executar auditoria nova"
  - "Visualizar — exibir revisão atual e sair"

Se "Visualizar": exibir arquivo, sair.
Se "Re-auditar": continuar.

## 2. Coletar Caminhos de Contexto

Construir lista de arquivos para o auditor:
- Todos os arquivos SUMMARY.md no diretório da fase
- Todos os arquivos PLAN.md no diretório da fase
- UI-SPEC.md (se existir — base da auditoria)
- CONTEXT.md (se existir — decisões fixadas)

## 3. Criar gsd-auditor-ui

```
◆ Criando auditor UI...
```

Construir prompt:

```markdown
Read D:/projetos/Estudo/devsquad/.cursor/agents/gsd-auditor-ui.md for instructions.

<objective>
Conduzir auditoria visual de 6 pilares da Fase {phase_number}: {phase_name}
{Se UI-SPEC existir: "Auditar contra contrato de design UI-SPEC.md."}
{Se não houver UI-SPEC: "Auditar contra padrões abstratos de 6 pilares."}
</objective>

<files_to_read>
- {summary_paths} (Sumários de execução)
- {plan_paths} (Planos de execução — o que era pretendido)
- {ui_spec_path} (Contrato de Design UI — base da auditoria, se existir)
- {context_path} (Decisões do usuário, se existir)
</files_to_read>

<config>
phase_dir: {phase_dir}
padded_phase: {padded_phase}
</config>
```

Omitir caminhos de arquivo nulos.

```
Task(
  prompt=ui_audit_prompt,
  subagent_type="gsd-auditor-ui",
  model="{UI_AUDITOR_MODEL}",
  description="Auditoria UI Fase {N}"
)
```

## 4. Tratar Retorno

**Se `## UI REVIEW COMPLETE`:**

Exibir resumo de pontuação:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUDITORIA UI COMPLETA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {N}: {Nome}** — Geral: {pontuação}/24

| Pilar | Pontuação |
|-------|-----------|
| Copywriting | {N}/4 |
| Visuais | {N}/4 |
| Cor | {N}/4 |
| Tipografia | {N}/4 |
| Espaçamento | {N}/4 |
| Design de Experiência | {N}/4 |

Principais correções:
1. {correção}
2. {correção}
3. {correção}

Revisão completa: {caminho para UI-REVIEW.md}

───────────────────────────────────────────────────────────────

## ▶ Próximo

- `/gsd-verificar-trabalho {N}` — testes UAT
- `/gsd-planejar-fase {N+1}` — planejar próxima fase

<sub>/clear primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────
```

## 5. Commit (se configurado)

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): revisão de auditoria UI" --files "${PHASE_DIR}/${PADDED_PHASE}-UI-REVIEW.md"
```

</process>

<success_criteria>
- [ ] Fase validada
- [ ] Arquivos SUMMARY.md encontrados (execução concluída)
- [ ] Revisão existente tratada (re-auditar/visualizar)
- [ ] gsd-auditor-ui criado com contexto correto
- [ ] UI-REVIEW.md criada no diretório da fase
- [ ] Resumo de pontuação exibido ao usuário
- [ ] Próximos passos apresentados
</success_criteria>
