---
name: gsd-remover-workspace
description: "Remover um workspace GSD e limpar worktrees"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-remover-workspace` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de input do usuário, pergunte conversacionalmente:
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

<context>
**Argumentos:**
- `<nome-workspace>` (obrigatório) — Nome do workspace a remover
</context>

<objective>
Remover um diretório de workspace após confirmação. Para estratégia worktree, executa `git worktree remove` para cada repo membro primeiro. Recusa se algum repo tiver alterações não commitadas.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/remover-workspace.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<process>
Execute o workflow remover-workspace de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/remover-workspace.md do início ao fim.
</process>
</output>
