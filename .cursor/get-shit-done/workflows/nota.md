<purpose>
Captura de ideias sem fricção. Uma chamada Write, uma linha de confirmação. Sem perguntas, sem prompts.
Executa inline — sem Task, sem conversational prompting, sem Bash.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="storage_format">
**Formato de armazenamento de notas.**

Notas são armazenadas como arquivos markdown individuais:

- **Escopo do projeto**: `.planning/notes/{YYYY-MM-DD}-{slug}.md` — usado quando `.planning/` existe no cwd
- **Escopo global**: `D:/projetos/Estudo/devsquad/.cursor/notes/{YYYY-MM-DD}-{slug}.md` — fallback quando não há `.planning/`, ou quando a flag `--global` está presente

Cada arquivo de nota:

```markdown
---
date: "YYYY-MM-DD HH:mm"
promoted: false
---

{texto da nota literal}
```

**Flag `--global`**: Remova `--global` de qualquer posição em `{{GSD_ARGS}}` antes de analisar. Quando presente, force escopo global independentemente da existência de `.planning/`.

**Importante**: NÃO crie `.planning/` se não existir. Use escopo global silenciosamente.
</step>

<step name="parse_subcommand">
**Analise o subcomando de {{GSD_ARGS}} (após remover --global).**

| Condição | Subcomando |
|----------|------------|
| Argumentos são exatamente `list` (case-insensitive) | **list** |
| Argumentos são exatamente `promote <N>` onde N é um número | **promote** |
| Argumentos estão vazios (sem texto algum) | **list** |
| Qualquer outra coisa | **append** (o texto É a nota) |

**Crítico**: `list` só é um subcomando quando é o argumento INTEIRO. `/gsd-nota list of groceries` salva uma nota com texto "list of groceries". O mesmo para `promote` — só é subcomando quando seguido por exatamente um número.
</step>

<step name="append">
**Subcomando: append — criar um arquivo de nota com timestamp.**

1. Determine o escopo (projeto ou global) conforme formato de armazenamento acima
2. Garanta que o diretório de notas existe (`.planning/notes/` ou `D:/projetos/Estudo/devsquad/.cursor/notes/`)
3. Gere o slug: primeiras ~4 palavras significativas do texto da nota, minúsculas, separadas por hífen (remova artigos/preposições do início)
4. Gere o nome do arquivo: `{YYYY-MM-DD}-{slug}.md`
   - Se um arquivo com esse nome já existir, adicione `-2`, `-3`, etc.
5. Escreva o arquivo com frontmatter e texto da nota (veja formato de armazenamento)
6. Confirme com exatamente uma linha: `Anotado ({escopo}): {texto da nota}`
   - Onde `{escopo}` é "projeto" ou "global"

**Restrições:**
- **Nunca modifique o texto da nota** — capture literalmente, incluindo erros de digitação
- **Nunca faça perguntas** — apenas escreva e confirme
- **Formato de timestamp**: Use hora local, `YYYY-MM-DD HH:mm` (24 horas, sem segundos)
</step>

<step name="list">
**Subcomando: list — mostrar notas de ambos os escopos.**

1. Glob `.planning/notes/*.md` (se o diretório existir) — notas do projeto
2. Glob `D:/projetos/Estudo/devsquad/.cursor/notes/*.md` (se o diretório existir) — notas globais
3. Para cada arquivo, leia o frontmatter para obter `date` e status `promoted`
4. Exclua arquivos onde `promoted: true` das contagens ativas (mas ainda mostre-os, esmaecidos)
5. Ordene por data, numere todas as entradas ativas sequencialmente começando em 1
6. Se total de entradas ativas > 20, mostre apenas as últimas 10 com uma nota sobre quantas foram omitidas

**Formato de exibição:**

```
Notas:

Projeto (.planning/notes/):
  1. [2026-02-08 14:32] refatorar o sistema de hooks para suportar validadores assíncronos
  2. [promovida] [2026-02-08 14:40] adicionar rate limiting aos endpoints da API
  3. [2026-02-08 15:10] considerar adicionar uma flag --dry-run ao build

Global (D:/projetos/Estudo/devsquad/.cursor/notes/):
  4. [2026-02-08 10:00] ideia cross-projeto sobre configuração compartilhada

{contagem} nota(s) ativa(s). Use `/gsd-nota promote <N>` para converter em todo.
```

Se um escopo não tem diretório ou entradas, mostre: `(sem notas)`
</step>

<step name="promote">
**Subcomando: promote — converter uma nota em todo.**

1. Execute a lógica de **list** para construir o índice numerado (ambos os escopos)
2. Encontre a entrada N da lista numerada
3. Se N for inválido ou referir a uma nota já promovida, informe o usuário e pare
4. **Requer diretório `.planning/`** — se não existir, avise: "Todos requerem um projeto GSD. Execute `/gsd-novo-projeto` para inicializar um."
5. Garanta que o diretório `.planning/todos/pending/` existe
6. Gere ID do todo: `{NNN}-{slug}` onde NNN é o próximo número sequencial (escaneie ambos `.planning/todos/pending/` e `.planning/todos/done/` para o número mais alto existente, incremente em 1, preencha com zeros até 3 dígitos) e slug são as primeiras ~4 palavras significativas do texto da nota
7. Extraia o texto da nota do arquivo fonte (corpo após frontmatter)
8. Crie `.planning/todos/pending/{id}.md`:

```yaml
---
title: "{texto da nota}"
status: pending
priority: P2
source: "promovida de /gsd-nota"
created: {YYYY-MM-DD}
theme: general
---

## Objetivo

{texto da nota}

## Contexto

Promovida de nota rápida capturada em {data original}.

## Critérios de Aceitação

- [ ] {critério principal derivado do texto da nota}
```

9. Marque o arquivo fonte da nota como promovido: atualize o frontmatter para `promoted: true`
10. Confirme: `Nota {N} promovida para todo {id}: {texto da nota}`
</step>

</process>

<edge_cases>
1. **"list" como texto de nota**: `/gsd-nota list of things` salva nota "list of things" (subcomando apenas quando `list` é o argumento inteiro)
2. **Sem `.planning/`**: Usa `D:/projetos/Estudo/devsquad/.cursor/notes/` global — funciona em qualquer diretório
3. **Promover sem projeto**: Avisa que todos requerem `.planning/`, sugere `/gsd-novo-projeto`
4. **Arquivos grandes**: `list` mostra últimos 10 quando >20 entradas ativas
5. **Slugs duplicados**: Adicione `-2`, `-3` etc. ao nome do arquivo se o slug já foi usado na mesma data
6. **Posição de `--global`**: Removida de qualquer lugar — `--global minha ideia` e `minha ideia --global` ambos salvam "minha ideia" globalmente
7. **Promover já promovida**: Informe ao usuário "Nota {N} já foi promovida" e pare
8. **Texto da nota vazio após remover flags**: Trate como subcomando `list`
</edge_cases>

<success_criteria>
- [ ] Append: Arquivo de nota escrito com frontmatter correto e texto literal
- [ ] Append: Nenhuma pergunta feita — captura instantânea
- [ ] List: Ambos os escopos mostrados com numeração sequencial
- [ ] List: Notas promovidas mostradas mas esmaecidas
- [ ] Promote: Todo criado com formato correto
- [ ] Promote: Nota fonte marcada como promovida
- [ ] Fallback global: Funciona quando `.planning/` não existe
</success_criteria>
