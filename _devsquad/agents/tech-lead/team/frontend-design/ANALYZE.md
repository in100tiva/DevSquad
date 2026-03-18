# Frontend Design — Modo: Revisar

> Primeira leitura: *"Isso parece site/App gerado por IA genérico?"* — antes de aprofundar interação (Cora) e usabilidade (Casey).

---

## Scan de 5 segundos

Responder sem hesitar:

1. Há **uma** direção visual clara (não “Bootstrap default”)?
2. Tipografia foge de Inter/Roboto/system stack óbvio?
3. Paleta evita **gradiente roxo em fundo branco** e variações batidas?
4. Layout tem algum caráter (assimetria, ritmo, foco) ou é grid 12-col genérico?
5. Algum detalhe memorável (textura, motion pontual, cor ousada bem usada)?

Se 3+ respostas forem “não” → finding provável **🔴** em `level: "estética"`.

---

## Auditoria anti–AI slop

| Sinal | Severidade típica |
|-------|-------------------|
| Fonte Inter/Roboto/Arial sem razão de marca | 🟡–🔴 |
| Hero + 3 cards + purple gradient | 🔴 |
| Ícones genéricos + cards iguais em fileira infinita | 🟡 |
| Zero motion onde o contexto pede vida (produto criativo) | 🟡 |
| Motion excessivo sem `prefers-reduced-motion` | 🟡 (acessibilidade) |
| Tema idêntico a outro projeto do mesmo repo | 🟡 |

---

## Entregável

Para cada problema:

- `problem`, `location`, `severity`, `fix` (direção alternativa concreta)
- Não confundir com: copy longa (Casey), botão sem feedback (Cora), carga cognitiva (Clara)

Registrar em `HANDOFF.findings` com `member: "frontend-design"`, `level: "estética"`.
