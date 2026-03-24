<overview>
Integração Git para o framework GSD.
</overview>

<core_principle>

**Faça commit de resultados, não de processos.**

O log do git deve ser lido como um changelog do que foi entregue, não um diário de atividade de planejamento.
</core_principle>

<commit_points>

| Evento                  | Commit? | Por quê                                          |
| ----------------------- | ------- | ------------------------------------------------ |
| BRIEF + ROADMAP criados | SIM     | Inicialização do projeto                         |
| PLAN.md criado          | NÃO     | Intermediário - commit com conclusão do plano    |
| RESEARCH.md criado      | NÃO     | Intermediário                                    |
| DISCOVERY.md criado     | NÃO     | Intermediário                                    |
| **Tarefa completada**   | SIM     | Unidade atômica de trabalho (1 commit por tarefa)|
| **Plano completado**    | SIM     | Commit de metadados (SUMMARY + STATE + ROADMAP)  |
| Handoff criado          | SIM     | Estado WIP preservado                            |

</commit_points>

<git_check>

```bash
[ -d .git ] && echo "GIT_EXISTS" || echo "NO_GIT"
```

Se NO_GIT: Execute `git init` silenciosamente. Projetos GSD sempre ganham seu próprio repositório.
</git_check>

<commit_formats>

<format name="initialization">
## Inicialização do Projeto (brief + roadmap juntos)

```
docs: initialize [nome-do-projeto] ([N] fases)

[Uma linha do PROJECT.md]

Fases:
1. [nome-da-fase]: [objetivo]
2. [nome-da-fase]: [objetivo]
3. [nome-da-fase]: [objetivo]
```

O que commitar:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: initialize [nome-do-projeto] ([N] fases)" --files .planning/
```

</format>

<format name="task-completion">
## Conclusão de Tarefa (Durante Execução do Plano)

Cada tarefa ganha seu próprio commit imediatamente após conclusão.

> **Agentes paralelos:** Quando rodando como executor paralelo (criado pelo execute-phase),
> use `--no-verify` em todos os commits para evitar contenção de lock do pre-commit hook.
> O orquestrador valida hooks uma vez após todos os agentes completarem.

```
{tipo}({fase}-{plano}): {nome-da-tarefa}

- [Mudança chave 1]
- [Mudança chave 2]
- [Mudança chave 3]
```

**Tipos de commit:**
- `feat` - Nova funcionalidade/recurso
- `fix` - Correção de bug
- `test` - Somente teste (fase RED do TDD)
- `refactor` - Limpeza de código (fase REFACTOR do TDD)
- `perf` - Melhoria de performance
- `chore` - Dependências, config, ferramentas

**Exemplos:**

```bash
# Tarefa padrão
git add src/api/auth.ts src/types/user.ts
git commit -m "feat(08-02): create user registration endpoint

- POST /auth/register valida email e senha
- Verifica usuários duplicados
- Retorna token JWT em caso de sucesso
"

# Tarefa TDD - fase RED
git add src/__tests__/jwt.test.ts
git commit -m "test(07-02): add failing test for JWT generation

- Testa se token contém claim de user ID
- Testa se token expira em 1 hora
- Testa verificação de assinatura
"

# Tarefa TDD - fase GREEN
git add src/utils/jwt.ts
git commit -m "feat(07-02): implement JWT generation

- Usa biblioteca jose para assinatura
- Inclui claims de user ID e expiração
- Assina com algoritmo HS256
"
```

</format>

<format name="plan-completion">
## Conclusão do Plano (Após Todas as Tarefas Concluídas)

Após todas as tarefas commitadas, um commit final de metadados captura a conclusão do plano.

```
docs({fase}-{plano}): complete [nome-do-plano] plan

Tarefas completadas: [N]/[N]
- [Nome da tarefa 1]
- [Nome da tarefa 2]
- [Nome da tarefa 3]

SUMMARY: .planning/phases/XX-name/{fase}-{plano}-SUMMARY.md
```

O que commitar:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs({fase}-{plano}): complete [nome-do-plano] plan" --files .planning/phases/XX-name/{fase}-{plano}-PLAN.md .planning/phases/XX-name/{fase}-{plano}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md
```

**Nota:** Arquivos de código NÃO incluídos - já commitados por tarefa.

</format>

<format name="handoff">
## Handoff (WIP)

```
wip: [nome-da-fase] pausado na tarefa [X]/[Y]

Atual: [nome da tarefa]
[Se bloqueado:] Bloqueado: [motivo]
```

O que commitar:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "wip: [nome-da-fase] pausado na tarefa [X]/[Y]" --files .planning/
```

</format>
</commit_formats>

<example_log>

**Abordagem antiga (commits por plano):**
```
a7f2d1 feat(checkout): Stripe payments with webhook verification
3e9c4b feat(products): catalog with search, filters, and pagination
8a1b2c feat(auth): JWT with refresh rotation using jose
5c3d7e feat(foundation): Next.js 15 + Prisma + Tailwind scaffold
2f4a8d docs: initialize ecommerce-app (5 phases)
```

**Nova abordagem (commits por tarefa):**
```
# Fase 04 - Checkout
1a2b3c docs(04-01): complete checkout flow plan
4d5e6f feat(04-01): add webhook signature verification
7g8h9i feat(04-01): implement payment session creation
0j1k2l feat(04-01): create checkout page component

# Fase 03 - Produtos
3m4n5o docs(03-02): complete product listing plan
6p7q8r feat(03-02): add pagination controls
9s0t1u feat(03-02): implement search and filters
2v3w4x feat(03-01): create product catalog schema

# Fase 02 - Auth
5y6z7a docs(02-02): complete token refresh plan
8b9c0d feat(02-02): implement refresh token rotation
1e2f3g test(02-02): add failing test for token refresh
4h5i6j docs(02-01): complete JWT setup plan
7k8l9m feat(02-01): add JWT generation and validation
0n1o2p chore(02-01): install jose library

# Fase 01 - Fundação
3q4r5s docs(01-01): complete scaffold plan
6t7u8v feat(01-01): configure Tailwind and globals
9w0x1y feat(01-01): set up Prisma with database
2z3a4b feat(01-01): create Next.js 15 project

# Inicialização
5c6d7e docs: initialize ecommerce-app (5 phases)
```

Cada plano produz 2-4 commits (tarefas + metadados). Claro, granular, bisecável.

</example_log>

<anti_patterns>

**Ainda não commitar (artefatos intermediários):**
- Criação de PLAN.md (commit com conclusão do plano)
- RESEARCH.md (intermediário)
- DISCOVERY.md (intermediário)
- Ajustes menores de planejamento
- "Corrigido typo no roadmap"

**Commitar (resultados):**
- Conclusão de cada tarefa (feat/fix/test/refactor)
- Metadados de conclusão do plano (docs)
- Inicialização do projeto (docs)

**Princípio chave:** Commitar código funcional e resultados entregues, não processo de planejamento.

</anti_patterns>

<commit_strategy_rationale>

## Por Que Commits Por Tarefa?

**Engenharia de contexto para IA:**
- Histórico Git se torna fonte primária de contexto para futuras sessões do Claude
- `git log --grep="{fase}-{plano}"` mostra todo trabalho de um plano
- `git diff <hash>^..<hash>` mostra mudanças exatas por tarefa
- Menos dependência de parsing de SUMMARY.md = mais contexto para trabalho real

**Recuperação de falhas:**
- Tarefa 1 commitada ✅, Tarefa 2 falhou ❌
- Claude na próxima sessão: vê tarefa 1 completa, pode tentar novamente tarefa 2
- Pode `git reset --hard` para última tarefa bem-sucedida

**Debugging:**
- `git bisect` encontra tarefa exata que falhou, não apenas plano que falhou
- `git blame` rastreia linha até contexto específico da tarefa
- Cada commit é independentemente reversível

**Observabilidade:**
- Fluxo de trabalho de desenvolvedor solo + Claude se beneficia de atribuição granular
- Commits atômicos são melhores práticas git
- "Ruído de commits" irrelevante quando consumidor é Claude, não humanos

</commit_strategy_rationale>

<sub_repos_support>

## Suporte a Workspace Multi-Repo (sub_repos)

Para workspaces com repositórios git separados (ex: `backend/`, `frontend/`, `shared/`), GSD roteia commits para cada repo independentemente.

### Configuração

Em `.planning/config.json`, liste diretórios de sub-repo em `planning.sub_repos`:

```json
{
  "planning": {
    "commit_docs": false,
    "sub_repos": ["backend", "frontend", "shared"]
  }
}
```

Defina `commit_docs: false` para que docs de planejamento fiquem locais e não sejam commitados em nenhum sub-repo.

### Como Funciona

1. **Auto-detecção:** Durante `/gsd-new-project`, diretórios com sua própria pasta `.git` são detectados e oferecidos para seleção como sub-repos. Em execuções subsequentes, `loadConfig` sincroniza automaticamente a lista `sub_repos` com o sistema de arquivos — adicionando repos recém-criados e removendo os deletados. Isso significa que `config.json` pode ser reescrito automaticamente quando repos mudam no disco.
2. **Agrupamento de arquivos:** Arquivos de código são agrupados pelo prefixo do sub-repo (ex: `backend/src/api/users.ts` pertence ao repo `backend/`).
3. **Commits independentes:** Cada sub-repo recebe seu próprio commit atômico via `gsd-tools.cjs commit-to-subrepo`. Caminhos de arquivo são relativos à raiz do sub-repo antes de staging.
4. **Planejamento fica local:** O diretório `.planning/` não é commitado; ele atua como coordenação entre repos.

### Roteamento de Commit

Em vez do comando padrão `commit`, use `commit-to-subrepo` quando `sub_repos` está configurado:

```bash
node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs commit-to-subrepo "feat(02-01): add user API" \
  --files backend/src/api/users.ts backend/src/types/user.ts frontend/src/components/UserForm.tsx
```

Isso faz staging de `src/api/users.ts` e `src/types/user.ts` no repo `backend/`, e `src/components/UserForm.tsx` no repo `frontend/`, depois commita cada um independentemente com a mesma mensagem.

Arquivos que não correspondem a nenhum sub-repo configurado são reportados como não correspondidos.

</sub_repos_support>
