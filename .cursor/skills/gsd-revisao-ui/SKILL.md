---
name: gsd-revisao-ui
description: "Auditoria visual retroativa de 6 pilares do código frontend implementado"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-revisao-ui` ou descreve uma tarefa correspondente a esta skill.
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

<objective>
Conduzir uma auditoria visual retroativa de 6 pilares. Produz UI-REVIEW.md com
avaliação graduada (1-4 por pilar). Funciona em qualquer projeto.
Saída: {num_fase}-UI-REVIEW.md
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/revisao-ui.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Fase: {{GSD_ARGS}} — opcional, padrão é a última fase concluída.
</context>

<process>
Execute @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/revisao-ui.md do início ao fim.
Preserve todas as portas do workflow.
</process>
</output>
