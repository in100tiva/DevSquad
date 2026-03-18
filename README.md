# DevSquad

Framework de code review baseado em agentes especialistas coordenados por **Lucas Tech Lead**.  
Repositório **isolado** — apenas o DevSquad, sem dependência do Opensquad.

## Instalação rápida

1. **Clone:** `git clone https://github.com/in100tiva/DevSquad.git`
2. **Copie** a pasta `_devsquad` e `.claude/skills/devsquad` para a raiz do seu projeto (onde você usa o Cursor).
3. **Configure** os MCPs (Context7, Playwright) — veja `_devsquad/PREREQUISITES.md`.
4. No Cursor: **`/devsquad`** para começar.

Passo a passo detalhado: **[INSTALL.md](INSTALL.md)**.

## O que é

- **Lucas Tech Lead** é o ponto de entrada único: recebe o pedido, identifica modo (criar / analisar / refatorar) e escopo, convoca a equipe e entrega um plano consolidado.
- **Equipe:** César (Clean Code), Camila (Clean Architecture), Diana (DDD), Giovana (GoF), Rafael (Software Architecture), Nadia (Everyday Things).
- Comandos: `/devsquad`, `/devsquad analyze`, `/devsquad create`, `/devsquad refactor`, `/devsquad team`, `/devsquad project`, `/devsquad prereqs`, `/devsquad help`.

## Estrutura deste repositório

```
_devsquad/           # Núcleo do framework (runner, agents, core, PREREQUISITES, mcp-check)
.claude/skills/      # Skill do Cursor para acionar /devsquad
```

## Usar em outro projeto

1. Clone ou baixe este repositório.
2. No seu projeto (onde você quer usar o DevSquad):
   - Copie a pasta **`_devsquad`** para a raiz do projeto.
   - Copie a pasta **`.claude/skills/devsquad`** para o seu `.claude/skills/` (crie `.claude/skills/` se não existir).
3. No Cursor, use `/devsquad` ou os comandos listados acima.
4. Antes da primeira execução, leia `_devsquad/PREREQUISITES.md` (MCPs: Context7, Playwright).

## Pré-requisitos (no projeto que usa o DevSquad)

- Cursor com MCPs configurados: **Context7** (obrigatório), **Playwright** (obrigatório), Docker MCP (opcional).
- Detalhes em `_devsquad/PREREQUISITES.md`.

## Licença

Use e adapte conforme sua necessidade. Atribuição apreciada.
