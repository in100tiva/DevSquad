<purpose>
Criar um diretório de workspace isolado com cópias de repos git (worktrees ou clones) e um diretório `.planning/` independente. Suporta orquestração multi-repo e isolamento de branch de feature em repo único.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 1. Setup

**PRIMEIRO PASSO OBRIGATÓRIO — Executar comando init:**

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init new-workspace)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar JSON para: `default_workspace_base`, `child_repos`, `child_repo_count`, `worktree_available`, `is_git_repo`, `cwd_repo_name`, `project_root`.

## 2. Analisar Argumentos

Extrair de {{GSD_ARGS}}:
- `--name` → `WORKSPACE_NAME` (obrigatório)
- `--repos` → `REPO_LIST` (caminhos ou nomes separados por vírgula)
- `--path` → `TARGET_PATH` (padrão: `$default_workspace_base/$WORKSPACE_NAME`)
- `--strategy` → `STRATEGY` (padrão: `worktree`)
- `--branch` → `BRANCH_NAME` (padrão: `workspace/$WORKSPACE_NAME`)
- `--auto` → pular perguntas interativas

**Se `--name` estiver ausente e não for `--auto`:**

Use conversational prompting:
- header: "Nome do Workspace"
- question: "Como este workspace deve se chamar?"
- requireAnswer: true

## 3. Selecionar Repos

**Se `--repos` for fornecido:** Analise valores separados por vírgula. Para cada valor:
- Se for um caminho absoluto, use diretamente
- Se for um caminho relativo ou nome, resolva contra `$project_root`
- Caso especial: `.` significa repo atual (use `$project_root`, nomeie como `$cwd_repo_name`)

**Se `--repos` NÃO for fornecido e não for `--auto`:**

**Se `child_repo_count` > 0:**

Apresente repos filhos para seleção:

Use conversational prompting:
- header: "Selecionar Repos"
- question: "Quais repos devem ser incluídos no workspace?"
- options: Liste cada repo filho do array `child_repos` por nome
- multiSelect: true

**Se `child_repo_count` for 0 e `is_git_repo` for true:**

Use conversational prompting:
- header: "Repo Atual"
- question: "Nenhum repo filho encontrado. Criar um workspace com o repo atual?"
- options:
  - "Sim — criar workspace com repo atual" → usar repo atual
  - "Cancelar" → sair

**Se `child_repo_count` for 0 e `is_git_repo` for false:**

Erro:
```
Nenhum repo git encontrado no diretório atual e este não é um repo git.

Execute este comando a partir de um diretório contendo repos git, ou especifique repos explicitamente:
  /gsd-new-workspace --name meu-workspace --repos /caminho/para/repo1,/caminho/para/repo2
```
Sair.

**Se `--auto` e `--repos` NÃO for fornecido:**

Erro:
```
Erro: --auto requer --repos para especificar quais repos incluir.

Uso:
  /gsd-new-workspace --name meu-workspace --repos repo1,repo2 --auto
```
Sair.

## 4. Selecionar Estratégia

**Se `--strategy` for fornecido:** Use-o (validar: deve ser `worktree` ou `clone`).

**Se `--strategy` NÃO for fornecido e não for `--auto`:**

Use conversational prompting:
- header: "Estratégia"
- question: "Como os repos devem ser copiados para o workspace?"
- options:
  - "Worktree (recomendado) — leve, compartilha objetos .git com repo fonte" → `worktree`
  - "Clone — cópia totalmente independente, sem conexão com repo fonte" → `clone`

**Se `--auto`:** Padrão para `worktree`.

## 5. Validar

Antes de criar qualquer coisa, validar:

1. **Caminho alvo** — não deve existir ou deve estar vazio:
```bash
if [ -d "$TARGET_PATH" ] && [ "$(ls -A "$TARGET_PATH" 2>/dev/null)" ]; then
  echo "Erro: Caminho alvo já existe e não está vazio: $TARGET_PATH"
  echo "Escolha um --name ou --path diferente."
  exit 1
fi
```

2. **Repos fonte existem e são repos git** — para cada caminho de repo:
```bash
if [ ! -d "$REPO_PATH/.git" ]; then
  echo "Erro: Não é um repo git: $REPO_PATH"
  exit 1
fi
```

3. **Disponibilidade de worktree** — se estratégia for `worktree` e `worktree_available` for false:
```
Erro: git não está disponível. Instale git ou use --strategy clone.
```

Relate todos os erros de validação de uma vez, não um por vez.

## 6. Criar Workspace

```bash
mkdir -p "$TARGET_PATH"
```

### Para cada repo:

**Estratégia worktree:**
```bash
cd "$SOURCE_REPO_PATH"
git worktree add "$TARGET_PATH/$REPO_NAME" -b "$BRANCH_NAME" 2>&1
```

Se `git worktree add` falhar porque a branch já existe, tente com branch timestamped:
```bash
TIMESTAMP=$(date +%Y%m%d%H%M%S)
git worktree add "$TARGET_PATH/$REPO_NAME" -b "${BRANCH_NAME}-${TIMESTAMP}" 2>&1
```

Se isso também falhar, relate o erro e continue com os repos restantes.

**Estratégia clone:**
```bash
git clone "$SOURCE_REPO_PATH" "$TARGET_PATH/$REPO_NAME" 2>&1
cd "$TARGET_PATH/$REPO_NAME"
git checkout -b "$BRANCH_NAME" 2>&1
```

Rastreie resultados: quais repos tiveram sucesso, quais falharam, qual branch foi usada.

## 7. Escrever WORKSPACE.md

Escreva o manifesto do workspace em `$TARGET_PATH/WORKSPACE.md`:

```markdown
# Workspace: $WORKSPACE_NAME

Criado: $DATE
Estratégia: $STRATEGY

## Repos Membros

| Repo | Fonte | Branch | Estratégia |
|------|-------|--------|------------|
| $REPO_NAME | $SOURCE_PATH | $BRANCH | $STRATEGY |
...para cada repo...

## Notas

[Adicione contexto sobre para que serve este workspace]
```

## 8. Inicializar .planning/

```bash
mkdir -p "$TARGET_PATH/.planning"
```

## 9. Relatório e Próximos Passos

**Se todos os repos tiveram sucesso:**

```
Workspace criado: $TARGET_PATH

  Repos: $REPO_COUNT
  Estratégia: $STRATEGY
  Branch: $BRANCH_NAME

Próximos passos:
  cd $TARGET_PATH
  /gsd-new-project    # Inicializar GSD no workspace
```

**Se alguns repos falharam:**

```
Workspace criado com $SUCCESS_COUNT de $TOTAL_COUNT repos: $TARGET_PATH

  Sucesso: repo1, repo2
  Falha: repo3 (branch já existe), repo4 (não é um repo git)

Próximos passos:
  cd $TARGET_PATH
  /gsd-new-project    # Inicializar GSD no workspace
```

**Oferecer inicialização do GSD (se não for `--auto`):**

Use conversational prompting:
- header: "Inicializar GSD"
- question: "Deseja inicializar um projeto GSD no novo workspace?"
- options:
  - "Sim — executar /gsd-new-project" → informar usuário para `cd $TARGET_PATH` primeiro, depois executar `/gsd-new-project`
  - "Não — vou configurar depois" → concluído

</process>

<success_criteria>
- [ ] Diretório do workspace criado no caminho alvo
- [ ] Todos os repos especificados copiados (worktree ou clone) no workspace
- [ ] Manifesto WORKSPACE.md escrito com tabela de repos correta
- [ ] Diretório `.planning/` inicializado na raiz do workspace
- [ ] Usuário informado do caminho do workspace e próximos passos
</success_criteria>
