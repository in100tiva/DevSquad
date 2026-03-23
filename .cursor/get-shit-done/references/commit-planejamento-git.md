# Commit de Planejamento Git

Commitar artefatos de planejamento usando o CLI gsd-tools, que automaticamente verifica a config `commit_docs` e o status do gitignore.

## Commit via CLI

Sempre use `gsd-tools.cjs commit` para arquivos `.planning/` — ele trata `commit_docs` e verificações de gitignore automaticamente:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs({escopo}): {descrição}" --files .planning/STATE.md .planning/ROADMAP.md
```

O CLI retornará `skipped` (com motivo) se `commit_docs` for `false` ou `.planning/` estiver no gitignore. Nenhuma verificação condicional manual necessária.

## Emendar commit anterior

Para incorporar mudanças de arquivos `.planning/` no commit anterior:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "" --files .planning/codebase/*.md --amend
```

## Padrões de Mensagem de Commit

| Comando | Escopo | Exemplo |
|---------|--------|---------|
| plan-phase | fase | `docs(phase-03): create authentication plans` |
| execute-phase | fase | `docs(phase-03): complete authentication phase` |
| new-milestone | milestone | `docs: start milestone v1.1` |
| remove-phase | chore | `chore: remove phase 17 (dashboard)` |
| insert-phase | fase | `docs: insert phase 16.1 (critical fix)` |
| add-phase | fase | `docs: add phase 07 (settings page)` |

## Quando Pular

- `commit_docs: false` na config
- `.planning/` está no gitignore
- Sem mudanças para commitar (verificar com `git status --porcelain .planning/`)
