---
name: gsd-verificar-todos
description: "Listar todos pendentes e selecionar um para trabalhar"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-verificar-todos` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, pergunte de forma conversacional:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Criação de Subagentes
Quando o workflow precisar criar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

<objective>
Listar todos os todos pendentes, permitir seleção, carregar contexto completo do todo selecionado e direcionar para a ação apropriada.

Direciona para o workflow verificar-todos que gerencia:
- Contagem e listagem de todos com filtragem por área
- Seleção interativa com carregamento completo de contexto
- Verificação de correlação com o roteiro
- Roteamento de ação (trabalhar agora, adicionar à fase, brainstorm, criar fase)
- Atualizações do STATE.md e commits git
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/verificar-todos.md
</execution_context>

<context>
Argumentos: {{GSD_ARGS}} (filtro de área opcional)

O estado dos todos e correlação com o roteiro são carregados dentro do workflow usando `init todos` e leituras direcionadas.
</context>

<process>
**Siga o workflow verificar-todos** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/verificar-todos.md`.

O workflow gerencia toda a lógica incluindo:
1. Verificação de existência de todos
2. Filtragem por área
3. Listagem interativa e seleção
4. Carregamento completo de contexto com resumos de arquivos
5. Verificação de correlação com o roteiro
6. Oferta e execução de ações
7. Atualizações do STATE.md
8. Commits git
</process>
</output>
