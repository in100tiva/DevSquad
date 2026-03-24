<purpose>
Remover um workspace GSD, limpando git worktrees e deletando o diretório do workspace.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 1. Setup

Extrair nome do workspace de {{GSD_ARGS}}.

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init remove-workspace "$WORKSPACE_NAME")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar JSON para: `workspace_name`, `workspace_path`, `has_manifest`, `strategy`, `repos`, `repo_count`, `dirty_repos`, `has_dirty_repos`.

**Se nenhum nome de workspace fornecido:**

Primeiro execute `/gsd-listar-workspaces` para mostrar workspaces disponíveis, depois pergunte:

Use conversational prompting:
- header: "Remover Workspace"
- question: "Qual workspace você deseja remover?"
- requireAnswer: true

Re-execute init com o nome fornecido.

## 2. Verificações de Segurança

**Se `has_dirty_repos` for true:**

```
Não é possível remover workspace "$WORKSPACE_NAME" — os seguintes repos têm mudanças não commitadas:

  - repo1
  - repo2

Faça commit ou stash das mudanças nesses repos antes de remover o workspace:
  cd $WORKSPACE_PATH/repo1
  git stash   # ou git commit
```

Sair. NÃO prossiga.

## 3. Confirmar Remoção

Use conversational prompting:
- header: "Confirmar Remoção"
- question: "Remover workspace '$WORKSPACE_NAME' em $WORKSPACE_PATH? Isto deletará todos os arquivos no diretório do workspace. Digite o nome do workspace para confirmar:"
- requireAnswer: true

**Se a resposta não corresponder a `$WORKSPACE_NAME`:** Sair com "Remoção cancelada."

## 4. Limpar Worktrees

**Se estratégia for `worktree`:**

Para cada repo no workspace:

```bash
cd "$SOURCE_REPO_PATH"
git worktree remove "$WORKSPACE_PATH/$REPO_NAME" 2>&1 || true
```

Se `git worktree remove` falhar, avisar mas continuar:
```
Aviso: Não foi possível remover worktree para $REPO_NAME — repo fonte pode ter sido movido ou deletado.
```

## 5. Deletar Diretório do Workspace

```bash
rm -rf "$WORKSPACE_PATH"
```

## 6. Relatório

```
Workspace "$WORKSPACE_NAME" removido.

  Caminho: $WORKSPACE_PATH (deletado)
  Repos: $REPO_COUNT worktrees limpos
```

</process>
