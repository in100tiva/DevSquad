# Flag de Fluxo de Trabalho (`--ws`)

## Visão Geral

A flag `--ws <nome>` delimita operações GSD a um fluxo de trabalho (workstream) específico, permitindo
trabalho paralelo de milestones por múltiplas instâncias do Cursor no mesmo codebase.

## Prioridade de Resolução

1. Flag `--ws <nome>` (explícito, maior prioridade)
2. Variável de ambiente `GSD_WORKSTREAM` (por instância)
3. Arquivo `.planning/active-workstream` (compartilhado, último escritor prevalece)
4. `null` — modo plano (sem workstreams)

## Propagação de Roteamento

Todos os comandos de roteamento de workflow incluem `${GSD_WS}` que:
- Expande para `--ws <nome>` quando um workstream está ativo
- Expande para string vazia no modo plano (retrocompatível)

Isso garante que o escopo do workstream encadeia automaticamente pelo workflow:
`new-milestone → discuss-phase → plan-phase → execute-phase → transition`

## Estrutura de Diretório

```
.planning/
├── PROJECT.md          # Compartilhado
├── config.json         # Compartilhado
├── milestones/         # Compartilhado
├── codebase/           # Compartilhado
├── active-workstream   # Aponta para ws atual
└── workstreams/
    ├── feature-a/      # Workstream A
    │   ├── STATE.md
    │   ├── ROADMAP.md
    │   ├── REQUIREMENTS.md
    │   └── phases/
    └── feature-b/      # Workstream B
        ├── STATE.md
        ├── ROADMAP.md
        ├── REQUIREMENTS.md
        └── phases/
```

## Uso via CLI

```bash
# Todos os comandos gsd-tools aceitam --ws
node gsd-tools.cjs state json --ws feature-a
node gsd-tools.cjs find-phase 3 --ws feature-b

# CRUD de Workstream
node gsd-tools.cjs workstream create <nome>
node gsd-tools.cjs workstream list
node gsd-tools.cjs workstream status <nome>
node gsd-tools.cjs workstream complete <nome>
```
