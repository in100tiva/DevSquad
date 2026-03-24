# Template de Log de Discussão

Template para `.planning/phases/XX-nome/{num_fase}-DISCUSSION-LOG.md` — trilha de auditoria das sessões de Q&A do discuss-phase.

**Propósito:** Trilha de auditoria de software para tomada de decisão. Captura todas as opções consideradas, não apenas a selecionada. Separado do CONTEXT.md, que é o artefato de implementação consumido pelos agentes downstream.

**NÃO é para consumo de LLM.** Este arquivo nunca deve ser referenciado em blocos `<files_to_read>` ou prompts de agentes.

## Formato

```markdown
# Fase [X]: [Nome] - Log de Discussão

> **Apenas trilha de auditoria.** Não use como entrada para agentes de planejamento, pesquisa ou execução.
> As decisões são capturadas no CONTEXT.md — este log preserva as alternativas consideradas.

**Data:** [data ISO]
**Fase:** [número da fase]-[nome da fase]
**Áreas discutidas:** [lista separada por vírgulas]

---

## [Nome da Área 1]

| Opção | Descrição | Selecionada |
|-------|-----------|-------------|
| [Opção 1] | [Breve descrição] | |
| [Opção 2] | [Breve descrição] | ✓ |
| [Opção 3] | [Breve descrição] | |

**Escolha do usuário:** [Opção selecionada ou resposta livre literal]
**Notas:** [Quaisquer esclarecimentos ou justificativas fornecidas durante a discussão]

---

## [Nome da Área 2]

...

---

## Critério do Claude

[Áreas delegadas ao julgamento do Claude — liste o que foi adiado e por quê]

## Ideias Adiadas

[Ideias mencionadas mas fora do escopo desta fase]

---

*Fase: XX-nome*
*Log de discussão gerado: [data]*
```

## Regras

- Gerado automaticamente ao final de cada sessão de discuss-phase
- Inclui TODAS as opções consideradas, não apenas a selecionada
- Inclui notas de texto livre e esclarecimentos do usuário
- Claramente marcado como apenas auditoria, não um artefato de implementação
- NÃO interfere na geração do CONTEXT.md ou no comportamento dos agentes downstream
- Commitado junto com o CONTEXT.md no mesmo commit git
