<purpose>
Listar todos os todos pendentes, permitir seleção, carregar contexto completo do todo selecionado e rotear para ação apropriada.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="init_context">
Carregar contexto de todos:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init todos)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `todo_count`, `todos`, `pending_dir`.

Se `todo_count` for 0:
```
Nenhum todo pendente.

Todos são capturados durante sessões de trabalho com /gsd-adicionar-todo.

---

Deseja:

1. Continuar com a fase atual (/gsd-progresso)
2. Adicionar um todo agora (/gsd-adicionar-todo)
```

Encerrar.
</step>

<step name="parse_filter">
Verificar filtro de área nos argumentos:
- `/gsd-verificar-todos` → mostrar todos
- `/gsd-verificar-todos api` → filtrar apenas para area:api
</step>

<step name="list_todos">
Usar o array `todos` do contexto de init (já filtrado por área se especificado).

Analisar e exibir como lista numerada:

```
Todos Pendentes:

1. Adicionar refresh de token de auth (api, 2d atrás)
2. Corrigir z-index do modal (ui, 1d atrás)
3. Refatorar pool de conexão do banco (database, 5h atrás)

---

Responda com um número para ver detalhes, ou:
- `/gsd-verificar-todos [área]` para filtrar por área
- `q` para sair
```

Formatar idade como tempo relativo a partir do timestamp de criação.
</step>

<step name="handle_selection">
Aguardar o usuário responder com um número.

Se válido: carregar todo selecionado, prosseguir.
Se inválido: "Seleção inválida. Responda com um número (1-[N]) ou `q` para sair."
</step>

<step name="load_context">
Ler o arquivo do todo completamente. Exibir:

```
## [título]

**Área:** [área]
**Criado:** [data] ([tempo relativo] atrás)
**Arquivos:** [lista ou "Nenhum"]

### Problema
[conteúdo da seção problema]

### Solução
[conteúdo da seção solução]
```

Se o campo `files` tiver entradas, ler e resumir brevemente cada uma.
</step>

<step name="check_roadmap">
Verificar roteiro (pode usar init progress ou verificar existência do arquivo diretamente):

Se `.planning/ROADMAP.md` existir:
1. Verificar se a área do todo corresponde a uma fase futura
2. Verificar se os arquivos do todo se sobrepõem ao escopo de uma fase
3. Anotar qualquer correspondência para opções de ação
</step>

<step name="offer_actions">
**Se o todo mapeia para uma fase do roteiro:**

Usar conversational prompting:
- header: "Ação"
- question: "Este todo se relaciona à Fase [N]: [nome]. O que deseja fazer?"
- options:
  - "Trabalhar nele agora" — mover para done, começar a trabalhar
  - "Adicionar ao plano da fase" — incluir ao planejar a Fase [N]
  - "Pensar na abordagem" — refletir antes de decidir
  - "Devolver" — retornar à lista

**Se sem correspondência no roteiro:**

Usar conversational prompting:
- header: "Ação"
- question: "O que deseja fazer com este todo?"
- options:
  - "Trabalhar nele agora" — mover para done, começar a trabalhar
  - "Criar uma fase" — /gsd-adicionar-fase com este escopo
  - "Pensar na abordagem" — refletir antes de decidir
  - "Devolver" — retornar à lista
</step>

<step name="execute_action">
**Trabalhar nele agora:**
```bash
mv ".planning/todos/pending/[nome_arquivo]" ".planning/todos/done/"
```
Atualizar contagem de todos no STATE.md. Apresentar contexto problema/solução. Começar trabalho ou perguntar como prosseguir.

**Adicionar ao plano da fase:**
Anotar referência do todo nas notas de planejamento da fase. Manter em pending. Retornar à lista ou sair.

**Criar uma fase:**
Exibir: `/gsd-adicionar-fase [descrição do todo]`
Manter em pending. Usuário executa o comando em contexto limpo.

**Pensar na abordagem:**
Manter em pending. Iniciar discussão sobre o problema e abordagens.

**Devolver:**
Retornar ao passo list_todos.
</step>

<step name="update_state">
Após qualquer ação que mude a contagem de todos:

Re-executar `init todos` para obter contagem atualizada, então atualizar seção "### Todos Pendentes" no STATE.md se existir.
</step>

<step name="git_commit">
Se o todo foi movido para done/, commitar a mudança:

```bash
git rm --cached .planning/todos/pending/[nome_arquivo] 2>/dev/null || true
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: start work on todo - [título]" --files .planning/todos/done/[nome_arquivo] .planning/STATE.md
```

A ferramenta respeita a config `commit_docs` e gitignore automaticamente.

Confirmar: "Commitado: docs: start work on todo - [título]"
</step>

</process>

<success_criteria>
- [ ] Todos os todos pendentes listados com título, área, idade
- [ ] Filtro de área aplicado se especificado
- [ ] Contexto completo do todo selecionado carregado
- [ ] Contexto do roteiro verificado para correspondência de fase
- [ ] Ações apropriadas oferecidas
- [ ] Ação selecionada executada
- [ ] STATE.md atualizado se contagem de todos mudou
- [ ] Mudanças commitadas no git (se todo movido para done/)
</success_criteria>
