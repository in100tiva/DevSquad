---
name: gsd-branch-pr
description: "Criar uma branch limpa para PR filtrando commits de .planning/ — pronta para code review"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-branch-pr` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com Usuário
Quando o workflow precisar de input do usuário, pergunte de forma conversacional:
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
Criar uma branch limpa adequada para pull requests filtrando commits de .planning/
da branch atual. Revisores veem apenas mudanças de código, não artefatos de planejamento GSD.

Isso resolve o problema de diffs de PR ficarem poluídos com mudanças de PLAN.md, SUMMARY.md, STATE.md
que são irrelevantes para code review.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/branch-pr.md
</execution_context>

<process>
Execute o workflow de branch-pr de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/branch-pr.md do início ao fim.
</process>
