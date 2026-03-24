---
name: gsd-adicionar-todo
description: "Capturar ideia ou tarefa como todo a partir do contexto da conversa atual"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-adicionar-todo` ou descreve uma tarefa correspondente a esta skill.
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
Capturar uma ideia, tarefa ou problema que surge durante uma sessão GSD como um todo estruturado para trabalho futuro.

Direciona para o workflow adicionar-todo que gerencia:
- Criação da estrutura de diretórios
- Extração de conteúdo dos argumentos ou conversa
- Inferência de área a partir de caminhos de arquivo
- Detecção e resolução de duplicatas
- Criação de arquivo todo com frontmatter
- Atualizações do STATE.md
- Commits git
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/adicionar-todo.md
</execution_context>

<context>
Argumentos: {{GSD_ARGS}} (descrição opcional do todo)

O estado é resolvido dentro do workflow via `init todos` e leituras direcionadas.
</context>

<process>
**Siga o workflow adicionar-todo** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/adicionar-todo.md`.

O workflow gerencia toda a lógica incluindo:
1. Garantia de diretório
2. Verificação de área existente
3. Extração de conteúdo (argumentos ou conversa)
4. Inferência de área
5. Verificação de duplicatas
6. Criação de arquivo com geração de slug
7. Atualizações do STATE.md
8. Commits git
</process>
</output>
