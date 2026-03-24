<purpose>
Listar todos os workspaces GSD encontrados em ~/gsd-workspaces/ com seu status.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 1. Setup

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init list-workspaces)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar JSON para: `workspace_base`, `workspaces`, `workspace_count`.

## 2. Exibir

**Se `workspace_count` for 0:**

```
Nenhum workspace encontrado em ~/gsd-workspaces/

Crie um com:
  /gsd-novo-workspace --name meu-workspace --repos repo1,repo2
```

Concluído.

**Se workspaces existirem:**

Exibir uma tabela:

```
Workspaces GSD (~/gsd-workspaces/)

| Nome | Repos | Estratégia | Projeto GSD |
|------|-------|------------|-------------|
| feature-a | 3 | worktree | Sim |
| feature-b | 2 | clone | Não |

Gerenciar:
  cd ~/gsd-workspaces/<nome>     # Entrar em um workspace
  /gsd-remover-workspace <nome>   # Remover um workspace
```

Para cada workspace, mostrar:
- **Nome** — nome do diretório
- **Repos** — contagem dos dados do init
- **Estratégia** — de WORKSPACE.md
- **Projeto GSD** — se `.planning/PROJECT.md` existe (Sim/Não)

</process>
