# Frontend Design — Modo: Criar

> Aplica **depois** de Casey no pipeline DevSquad: estrutura, copy e usabilidade já definidas; aqui fixa **caráter visual** e especificação para implementação.

---

## Pré-requisitos do HANDOFF

Ler `refactor_steps`, `dx_issues`, `ux_issues`, `interaction_issues`, `usability_issues`, `trunk_test_result`. **Não** reabrir decisões de Clara/Cora/Casey — apenas **vestir** a UI com estética coerente.

---

## Passo 1 — Design Thinking (documentar em `aesthetic_spec`)

1. **purpose** — alinhado ao `scope.description`
2. **tone** — um rótulo forte (ex.: editorial-escuro, pastel-clínico, brutalista-dados)
3. **constraints** — stack, WCAG mínimo, performance (reduzir motion se `prefers-reduced-motion`)
4. **differentiation** — uma frase: *"O usuário vai lembrar de ___."*

---

## Passo 2 — Checklist estética

### Tipografia

- [ ] Display + body definidos (nomes de fonte ou Google Fonts / variável)
- [ ] Escala tipográfica (tamanhos/line-height) coerente com o tom
- [ ] Nada de Inter/Roboto/Arial como escolha “padrão” sem justificativa de marca

### Cor e tema

- [ ] `:root` ou theme tokens (CSS variables)
- [ ] 1 cor dominante + 1–2 acentos; evitar paleta “torta” genérica
- [ ] Modo claro/escuro se o produto exigir — decisão explícita

### Motion

- [ ] 1 momento de entrada forte (stagger leve) OU abstencão total se minimal
- [ ] Hover/focus visíveis e alinhados ao tom (não só `opacity: 0.8`)

### Composição espacial

- [ ] Grid ou quebra intencional documentada
- [ ] Hierarquia visual compatível com headings/copy que Casey definiu

### Fundo e detalhe

- [ ] Pelo menos um elemento de profundidade (gradient sutil, noise leve, borda, sombra contextual) — ou justificativa minimalista explícita

---

## Passo 3 — Entregável

Preencher `HANDOFF.aesthetic_spec` (ver Lucas CREATE.md) +, se útil, anexar:

- Snippet de tokens CSS
- Lista de classes/utilitários principais
- Nota: *"O que NÃO fazer"* (anti-patterns a evitar na implementação)

Não duplicar lista de problemas de usabilidade — isso é Casey.
