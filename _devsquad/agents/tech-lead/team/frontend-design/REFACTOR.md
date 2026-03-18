# Frontend Design — Modo: Refatorar

> Mudanças **visuais** sem alterar comportamento nem fluxo acordado por Casey/Cora.

---

## Ordem de impacto (baixo risco → alto)

1. **Tokens** — cores, radius, sombras via CSS variables (sem mudar HTML)
2. **Tipografia** — trocar família/escala; validar contraste
3. **Backgrounds** — textura leve, gradient contextual, remover “flat” genérico
4. **Motion** — uma entrada ou hover forte; respeitar `prefers-reduced-motion`
5. **Composição** — só se não quebrar hierarquia de copy/trunk test (conflito → `conflicts_with: "usability_issues"`)

---

## Playbook rápido

**FD-01 — Troca de fonte**  
Substituir stack genérica por par display + body; atualizar imports / `next/font` / `@font-face`.

**FD-02 — Tema coeso**  
Definir `--color-bg`, `--color-fg`, `--color-accent`, `--radius`, `--shadow`; remover cores hardcoded espalhadas.

**FD-03 — Hero genérico**  
Quebrar simetria: offset, overlap, imagem crop ousado, ou tipografia grande com menos elementos.

**FD-04 — Motion único**  
Um bloco com `animation-delay` escalonado no load; não poluir cada `div`.

---

## Conflitos

- Cortar texto para “limpar” visual → pode afetar signifiers (Cora) ou scan (Casey) → registrar `conflicts_with`.
- Lucas decide trade-off na síntese.

## Checklist pós-refatoração

- [ ] Contraste texto/fundo aceitável
- [ ] Nenhum regressão de a11y (focus, labels)
- [ ] `aesthetic_spec` atualizado com o novo estado
