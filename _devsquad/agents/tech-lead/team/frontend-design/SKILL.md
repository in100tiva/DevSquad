---
name: frontend-design
description: >
  Frontend Design (skill Anthropic) cria interfaces web distintas e production-grade,
  com estética memorável e código que evita o visual genérico de IA.
  Use no DevSquad após usabilidade/copy (Casey): define direção visual, tipografia,
  cor, motion e composição. Acionar quando o escopo for componente, página ou app
  com UI; ou quando o usuário pedir landing, dashboard, tema forte, anti-AI-slop.
---

# Frontend Design

> *"Intenção clara na estética — maximalismo e minimalismo refinado são válidos; o que não vale é parecer template de IA."*

Este membro guia criação de frontends **distintos**, com atenção a detalhe estético e escolhas criativas — alinhado ao skill Anthropic *frontend-design*.

O usuário fornece requisitos: componente, página, aplicação ou interface. Pode incluir propósito, audiência ou restrições técnicas.

## Modos de trabalho

| Modo | Arquivo |
|------|---------|
| Criar UI distintiva | [`CREATE.md`](./CREATE.md) |
| Revisar estética / anti-slop | [`ANALYZE.md`](./ANALYZE.md) |
| Refatorar visual | [`REFACTOR.md`](./REFACTOR.md) |

---

## Design Thinking

Antes de codar, entender o contexto e comprometer uma direção estética **clara**:

- **Purpose**: Que problema resolve? Quem usa?
- **Tone**: Um extremo coerente (minimal brutal, maximalista, retro-futuro, orgânico, luxo, lúdico, editorial, brutalista, art déco, pastel, industrial…).
- **Constraints**: Framework, performance, acessibilidade.
- **Differentiation**: O que torna isto **inesquecível**? O que alguém vai lembrar?

**CRÍTICO**: Direção conceitual clara + execução precisa. Intencionalidade importa mais que “gritar”.

Implementar código real (HTML/CSS/JS, React, Vue, etc.) que seja:

- Production-grade e funcional
- Visualmente marcante e memorável
- Coeso com ponto de vista estético definido
- Refinado no detalhe

---

## Frontend Aesthetics Guidelines

- **Typography**: Fontes com caráter; evitar Arial, Inter, Roboto genéricos. Display distintivo + corpo refinado.
- **Color & Theme**: Paleta coesa; CSS variables; cor dominante + acentos fortes.
- **Motion**: Micro-interações e momentos de impacto (ex.: entrada em página com stagger). CSS-first; Motion (React) quando disponível; hover/scroll que surpreendem sem poluir.
- **Spatial Composition**: Layouts inesperados, assimetria, sobreposição, quebra de grid, negativo generoso ou densidade controlada.
- **Backgrounds & Details**: Atmosfera — gradient meshes, noise, padrões geométricos, transparências, sombras, bordas decorativas, grain.

**NUNCA** convergir para clichê de IA: Inter + gradiente roxo no branco, layouts previsíveis, Space Grotesk em tudo, “cookie-cutter”.

**IMPORTANTE**: Complexidade de código proporcional à visão — maximalista exige mais código; minimalista exige restrição e precisão de espaçamento/tipografia.

Ver também [`README.md`](./README.md) e o [cookbook Anthropic](https://github.com/anthropics/claude-cookbooks/blob/main/coding/prompting_for_frontend_aesthetics.ipynb).
