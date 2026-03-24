<planning_config>

Opções de configuração para comportamento do diretório `.planning/`.

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"git": {
  "branching_strategy": "none",
  "phase_branch_template": "gsd/phase-{phase}-{slug}",
  "milestone_branch_template": "gsd/{milestone}-{slug}",
  "quick_branch_template": null
}
```

| Opção | Padrão | Descrição |
|-------|--------|-----------|
| `commit_docs` | `true` | Se deve commitar artefatos de planejamento no git |
| `search_gitignored` | `false` | Adicionar `--no-ignore` a buscas amplas com rg |
| `git.branching_strategy` | `"none"` | Abordagem de branching Git: `"none"`, `"phase"`, ou `"milestone"` |
| `git.phase_branch_template` | `"gsd/phase-{phase}-{slug}"` | Template de branch para estratégia por fase |
| `git.milestone_branch_template` | `"gsd/{milestone}-{slug}"` | Template de branch para estratégia por milestone |
| `git.quick_branch_template` | `null` | Template opcional de branch para execuções de tarefas rápidas |
</config_schema>

<commit_docs_behavior>

**Quando `commit_docs: true` (padrão):**
- Arquivos de planejamento commitados normalmente
- SUMMARY.md, STATE.md, ROADMAP.md rastreados no git
- Histórico completo de decisões de planejamento preservado

**Quando `commit_docs: false`:**
- Pular todo `git add`/`git commit` para arquivos `.planning/`
- Usuário deve adicionar `.planning/` ao `.gitignore`
- Útil para: contribuições OSS, projetos de clientes, manter planejamento privado

**Usando gsd-tools.cjs (preferido):**

```bash
# Commit com verificações automáticas de commit_docs + gitignore:
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: update state" --files .planning/STATE.md

# Carregar config via state load (retorna JSON):
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docs está disponível na saída JSON

# Ou use comandos init que incluem commit_docs:
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docs está incluído em todas as saídas de comandos init
```

**Auto-detecção:** Se `.planning/` está no gitignore, `commit_docs` é automaticamente `false` independente do config.json. Isso previne erros git quando usuários têm `.planning/` no `.gitignore`.

**Commit via CLI (trata verificações automaticamente):**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: update state" --files .planning/STATE.md
```

O CLI verifica status de `commit_docs` e gitignore internamente — nenhuma condicional manual necessária.

</commit_docs_behavior>

<search_behavior>

**Quando `search_gitignored: false` (padrão):**
- Comportamento padrão do rg (respeita .gitignore)
- Buscas com caminho direto funcionam: `rg "padrão" .planning/` encontra arquivos
- Buscas amplas pulam ignorados: `rg "padrão"` pula `.planning/`

**Quando `search_gitignored: true`:**
- Adicionar `--no-ignore` a buscas amplas com rg que devem incluir `.planning/`
- Somente necessário quando buscando no repositório inteiro e esperando matches em `.planning/`

**Nota:** A maioria das operações GSD usa leitura direta de arquivos ou caminhos explícitos, que funcionam independente do status do gitignore.

</search_behavior>

<setup_uncommitted_mode>

Para usar modo não-commitado:

1. **Definir config:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **Adicionar ao .gitignore:**
   ```
   .planning/
   ```

3. **Arquivos já rastreados:** Se `.planning/` já foi rastreado anteriormente:
   ```bash
   git rm -r --cached .planning/
   git commit -m "chore: stop tracking planning docs"
   ```

4. **Merges de branch:** Quando usando `branching_strategy: phase` ou `milestone`, o workflow `complete-milestone` automaticamente remove arquivos `.planning/` do staging antes de commits de merge quando `commit_docs: false`.

</setup_uncommitted_mode>

<branching_strategy_behavior>

**Estratégias de Branching:**

| Estratégia | Quando branch é criada | Escopo da branch | Ponto de merge |
|------------|------------------------|------------------|----------------|
| `none` | Nunca | N/A | N/A |
| `phase` | No início de `execute-phase` | Fase única | Usuário faz merge após fase |
| `milestone` | No primeiro `execute-phase` do milestone | Milestone inteiro | No `complete-milestone` |

**Quando `git.branching_strategy: "none"` (padrão):**
- Todo trabalho commita na branch atual
- Comportamento padrão GSD

**Quando `git.branching_strategy: "phase"`:**
- `execute-phase` cria/troca para uma branch antes da execução
- Nome da branch do `phase_branch_template` (ex: `gsd/phase-03-authentication`)
- Todos os commits do plano vão para essa branch
- Usuário faz merge das branches manualmente após conclusão da fase
- `complete-milestone` oferece fazer merge de todas as branches de fase

**Quando `git.branching_strategy: "milestone"`:**
- Primeiro `execute-phase` do milestone cria a branch do milestone
- Nome da branch do `milestone_branch_template` (ex: `gsd/v1.0-mvp`)
- Todas as fases no milestone commitam na mesma branch
- `complete-milestone` oferece fazer merge da branch do milestone para main

**Variáveis de template:**

| Variável | Disponível em | Descrição |
|----------|---------------|-----------|
| `{phase}` | phase_branch_template | Número da fase com zero à esquerda (ex: "03") |
| `{slug}` | Ambos | Nome em minúsculas, separado por hífen |
| `{milestone}` | milestone_branch_template | Versão do milestone (ex: "v1.0") |

**Verificando a config:**

Use `init execute-phase` que retorna toda config como JSON:
```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# Saída JSON inclui: branching_strategy, phase_branch_template, milestone_branch_template
```

Ou use `state load` para os valores de config:
```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# Parsear branching_strategy, phase_branch_template, milestone_branch_template do JSON
```

**Criação de branch:**

```bash
# Para estratégia por fase
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# Para estratégia por milestone
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**Opções de merge no complete-milestone:**

| Opção | Comando Git | Resultado |
|-------|-------------|-----------|
| Squash merge (recomendado) | `git merge --squash` | Commit limpo único por branch |
| Merge com histórico | `git merge --no-ff` | Preserva todos os commits individuais |
| Deletar sem merge | `git branch -D` | Descartar trabalho da branch |
| Manter branches | (nenhum) | Tratamento manual depois |

Squash merge é recomendado — mantém o histórico da branch main limpo enquanto preserva o histórico completo de desenvolvimento na branch (até ser deletada).

**Casos de uso:**

| Estratégia | Melhor para |
|------------|-------------|
| `none` | Desenvolvimento solo, projetos simples |
| `phase` | Code review por fase, rollback granular, colaboração em equipe |
| `milestone` | Branches de release, ambientes de staging, PR por versão |

</branching_strategy_behavior>

</planning_config>
