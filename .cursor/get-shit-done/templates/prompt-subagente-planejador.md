# Template de Prompt do Subagente Planejador

Template para criar o agente gsd-planner. O agente contém toda a expertise de planejamento - este template fornece apenas o contexto de planejamento.

---

## Template

```markdown
<planning_context>

**Phase:** {phase_number}
**Mode:** {standard | gap_closure}

**Project State:**
@.planning/STATE.md

**Roadmap:**
@.planning/ROADMAP.md

**Requirements (if exists):**
@.planning/REQUIREMENTS.md

**Phase Context (if exists):**
@.planning/phases/{phase_dir}/{phase_num}-CONTEXT.md

**Research (if exists):**
@.planning/phases/{phase_dir}/{phase_num}-RESEARCH.md

**Gap Closure (if --gaps mode):**
@.planning/phases/{phase_dir}/{phase_num}-VERIFICATION.md
@.planning/phases/{phase_dir}/{phase_num}-UAT.md

</planning_context>

<downstream_consumer>
Saída consumida por /gsd-execute-phase
Os planos devem ser prompts executáveis com:
- Frontmatter (wave, depends_on, files_modified, autonomous)
- Tarefas em formato XML
- Critérios de verificação
- must_haves para verificação goal-backward
</downstream_consumer>

<quality_gate>
Antes de retornar PLANEJAMENTO COMPLETO:
- [ ] Arquivos PLAN.md criados no diretório da fase
- [ ] Cada plano tem frontmatter válido
- [ ] Tarefas são específicas e acionáveis
- [ ] Dependências corretamente identificadas
- [ ] Waves atribuídas para execução paralela
- [ ] must_haves derivados do objetivo da fase
</quality_gate>
```

---

## Placeholders

| Placeholder | Origem | Exemplo |
|-------------|--------|---------|
| `{phase_number}` | Do roadmap/argumentos | `5` ou `2.1` |
| `{phase_dir}` | Nome do diretório da fase | `05-perfis-usuario` |
| `{phase}` | Prefixo da fase | `05` |
| `{standard \| gap_closure}` | Flag de modo | `standard` |

---

## Uso

**A partir do /gsd-plan-phase (modo padrão):**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-planner",
  description="Planejar Fase {phase}"
)
```

**A partir do /gsd-plan-phase --gaps (modo de fechamento de lacunas):**
```python
Task(
  prompt=filled_template,  # com mode: gap_closure
  subagent_type="gsd-planner",
  description="Planejar lacunas da Fase {phase}"
)
```

---

## Continuação

Para checkpoints, crie um novo agente com:

```markdown
<objective>
Continuar planejamento para Fase {phase_number}: {phase_name}
</objective>

<prior_state>
Diretório da fase: @.planning/phases/{phase_dir}/
Planos existentes: @.planning/phases/{phase_dir}/*-PLAN.md
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>

<mode>
Continue: {standard | gap_closure}
</mode>
```

---

**Nota:** Metodologia de planejamento, decomposição de tarefas, análise de dependências, atribuição de waves, detecção de TDD e derivação goal-backward estão incorporados no agente gsd-planner. Este template apenas passa contexto.
