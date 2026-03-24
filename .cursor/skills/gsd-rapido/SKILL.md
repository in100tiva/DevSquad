---
name: gsd-rapido
description: "Executar uma tarefa trivial inline — sem subagentes, sem overhead de planejamento"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-rapido` ou descreve uma tarefa correspondente a esta skill.
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
Executar uma tarefa trivial diretamente no contexto atual sem gerar subagentes
ou criar arquivos PLAN.md. Para tarefas pequenas demais para justificar overhead de planejamento:
correções de typo, mudanças de configuração, refatorações pequenas, commits esquecidos, adições simples.

Este NÃO é um substituto para /gsd-rapido-garantido — use /gsd-rapido-garantido para qualquer coisa que
precise de pesquisa, planejamento multi-passos ou verificação. /gsd-rapido é para tarefas
que você poderia descrever em uma frase e executar em menos de 2 minutos.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/rapido.md
</execution_context>

<process>
Execute o workflow rapido de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/rapido.md de ponta a ponta.
</process>
</output>
