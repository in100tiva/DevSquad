# Template de Arquivo de Marco

Este template é usado pelo workflow complete-milestone para criar arquivos de arquivo em `.planning/milestones/`.

---

## Template do Arquivo

# Marco v{{VERSION}}: {{MILESTONE_NAME}}

**Status:** ✅ ENTREGUE {{DATE}}
**Fases:** {{PHASE_START}}-{{PHASE_END}}
**Total de Planos:** {{TOTAL_PLANS}}

## Visão Geral

{{MILESTONE_DESCRIPTION}}

## Fases

{{PHASES_SECTION}}

[Para cada fase neste marco, inclua:]

### Fase {{PHASE_NUM}}: {{PHASE_NAME}}

**Objetivo**: {{PHASE_GOAL}}
**Depende de**: {{DEPENDS_ON}}
**Planos**: {{PLAN_COUNT}} planos

Planos:

- [x] {{PHASE}}-01: {{PLAN_DESCRIPTION}}
- [x] {{PHASE}}-02: {{PLAN_DESCRIPTION}}
      [... todos os planos ...]

**Detalhes:**
{{PHASE_DETAILS_FROM_ROADMAP}}

**Para fases decimais, inclua o marcador (INSERIDA):**

### Fase 2.1: Patch Crítico de Segurança (INSERIDA)

**Objetivo**: Corrigir vulnerabilidade de bypass de autenticação
**Depende de**: Fase 2
**Planos**: 1 plano

Planos:

- [x] 02.1-01: Corrigir vulnerabilidade de autenticação

**Detalhes:**
{{PHASE_DETAILS_FROM_ROADMAP}}

---

## Resumo do Marco

**Fases Decimais:**

- Fase 2.1: Patch Crítico de Segurança (inserida após Fase 2 para correção urgente)
- Fase 5.1: Hotfix de Performance (inserida após Fase 5 para problema em produção)

**Decisões Principais:**
{{DECISIONS_FROM_PROJECT_STATE}}
[Exemplo:]

- Decisão: Usar divisão do ROADMAP.md (Justificativa: Custo constante de contexto)
- Decisão: Numeração decimal de fases (Justificativa: Semântica clara de inserção)

**Problemas Resolvidos:**
{{ISSUES_RESOLVED_DURING_MILESTONE}}
[Exemplo:]

- Corrigido overflow de contexto com 100+ fases
- Resolvida confusão de inserção de fases

**Problemas Adiados:**
{{ISSUES_DEFERRED_TO_LATER}}
[Exemplo:]

- Hierarquização do PROJECT-STATE.md (adiada até decisões > 300)

**Dívida Técnica Incorrida:**
{{SHORTCUTS_NEEDING_FUTURE_WORK}}
[Exemplo:]

- Alguns workflows ainda têm caminhos hardcoded (corrigir na Fase 5)

---

_Para status atual do projeto, veja .planning/ROADMAP.md_

---

## Diretrizes de Uso

<guidelines>
**Quando criar arquivos de marco:**
- Após completar todas as fases de um marco (v1.0, v1.1, v2.0, etc.)
- Acionado pelo workflow complete-milestone
- Antes de planejar o trabalho do próximo marco

**Como preencher o template:**

- Substitua os {{PLACEHOLDERS}} com valores reais
- Extraia detalhes das fases do ROADMAP.md
- Documente fases decimais com o marcador (INSERIDA)
- Inclua decisões-chave do PROJECT-STATE.md ou arquivos SUMMARY
- Liste problemas resolvidos vs adiados
- Capture dívida técnica para referência futura

**Local do arquivo:**

- Salve em `.planning/milestones/v{VERSION}-{NOME}.md`
- Exemplo: `.planning/milestones/v1.0-mvp.md`

**Após arquivar:**

- Atualize o ROADMAP.md para colapsar o marco concluído em tag `<details>`
- Atualize o PROJECT.md para formato brownfield com seção Estado Atual
- Continue a numeração de fases no próximo marco (nunca reinicie em 01)
  </guidelines>
