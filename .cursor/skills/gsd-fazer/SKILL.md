---
name: gsd-fazer
description: "Rotear texto livre para o comando GSD correto automaticamente"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-fazer` ou descreve uma tarefa correspondente a esta skill.
- Trate todo o texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, solicite conversacionalmente:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Geração de Subagentes
Quando o workflow precisar gerar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

<objective>
Analisar entrada de linguagem natural em texto livre e despachar para o comando GSD mais apropriado.

Atua como um despachante inteligente — nunca faz o trabalho em si. Corresponde intenção ao melhor comando GSD usando regras de roteamento, confirma a correspondência, depois delega.

Use quando você sabe o que quer mas não sabe qual comando `/gsd-*` executar.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/fazer.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
{{GSD_ARGS}}
</context>

<process>
Execute o workflow fazer de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/fazer.md de ponta a ponta.
Roteie a intenção do usuário para o melhor comando GSD e invoque-o.
</process>
</output>
