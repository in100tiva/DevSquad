# DreamCut — especificação pixel-perfect (padrão DevSquad)

Este documento é o **referencial visual** que o agente **Frontend Design** deve seguir em projetos internos (ex.: Hub Médico) quando o produto adotar DreamCut.

## Fonte canônica (copiar para novos projetos)

No repositório **devsquad**, o modelo implementado está em:

- `hubmedico-platform/src/design/tokens.css` — todas as variáveis `:root`
- `hubmedico-platform/src/design/tokens.ts` — espelho tipado (`import tokens from "@/design/tokens"`)
- `hubmedico-platform/src/design/examples.tsx` — 7 exemplos (Button, Chip, Input, Kanban, etc.)
- `hubmedico-platform/src/app/globals.css` — `@import` do tokens + `@theme` Tailwind v4

## Princípios

1. **Uma fonte de verdade:** alterar token no CSS (e espelhar em `tokens.ts` se existir).
2. **Dark:** três camadas — `#0D0D0D` → `#1A1A1A` → `#252729`.
3. **CTA dark:** `#0023D2` (pill). **Digital** `#E6FF00` só pontual (dark).
4. **Foco input:** sempre **pink** `#F02D7D`.
5. **Labels:** sempre uppercase + letter-spacing positivo.
6. **Raio perfeito:** raio externo = raio interno + padding.

## Checklist (antes de aprovar UI)

Ver seção final em `hubmedico-platform/docs/DESIGN_SYSTEM.md` (itens 1–12).

## Conflito com “anti-slop” genérico

DreamCut é **intencionalmente** sistemático (DM Sans + Inter pesado, cinzas frios, CTAs azuis). Não substituir por Inter roxo-gradiente genérico: o diferencial aqui é **aderência ao spec**, não “surpresa estética” aleatória.
