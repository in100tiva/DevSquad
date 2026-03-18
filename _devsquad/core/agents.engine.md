# DevSquad Agents Engine — `_devsquad/core/agents.engine.md`

## Onde estão os membros

**Caminho base:** `_devsquad/agents/tech-lead/team/`

Os membros da equipe do Lucas vivem em subpastas de `team/`. O runner e o Lucas referenciam sempre esse caminho.

---

## Listar membros

1. Varrer as **subpastas** de `_devsquad/agents/tech-lead/team/`.
2. Para cada subpasta que contém **SKILL.md**:
   - Ler o **frontmatter** do SKILL.md (name, description).
   - Verificar se existem **CREATE.md**, **ANALYZE.md**, **REFACTOR.md** (modos disponíveis).
   - Exibir: **nome** + **descrição** + **modos** (Criar / Analisar / Refatorar conforme os .md presentes).

**Exemplo de saída:**

| Membro | Descrição | Modos |
|--------|-----------|-------|
| clean-code (César) | Clean Code — nomes, funções, smells | CREATE, ANALYZE, REFACTOR |
| clean-architecture (Camila) | Clean Architecture — Dependency Rule, ports | CREATE, ANALYZE, REFACTOR |
| … | … | … |

---

## Adicionar novo membro

1. Criar a pasta `_devsquad/agents/tech-lead/team/{nome}/`.
2. Incluir os **4 arquivos**: SKILL.md, CREATE.md, ANALYZE.md, REFACTOR.md (no mesmo padrão dos membros existentes).
3. Lucas passa a poder convocá-lo ao incluir o membro na task (CREATE/ANALYZE/REFACTOR) conforme o escopo.
4. **Não** é necessário alterar o entry skill (`.claude/skills/devsquad/SKILL.md`) nem o runner.

---

## Uso pelo runner

- O **runner** conhece apenas `_devsquad/agents/tech-lead/`; os membros são carregados via caminho `tech-lead/team/{membro}/`.
- O comando **/devsquad team** deve listar os membros seguindo este agents.engine (varrer `tech-lead/team/` e exibir nome, descrição e modos).
