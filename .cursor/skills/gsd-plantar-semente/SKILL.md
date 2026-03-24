---
name: gsd-plantar-semente
description: "Capturar uma ideia voltada ao futuro com condições de gatilho — aparece automaticamente no marco certo"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-plantar-semente` ou descreve uma tarefa correspondente a esta skill.
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
Capturar uma ideia que é grande demais para agora mas deve aparecer automaticamente quando o
marco certo chegar. Sementes resolvem a deterioração de contexto: em vez de uma linha em Adiados que
ninguém lê, uma semente preserva o POR QUÊ completo, QUANDO surfacear, e trilhas para os detalhes.

Cria: .planning/seeds/SEED-NNN-slug.md
Consumido por: /gsd-novo-marco (escaneia sementes e apresenta correspondências)
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/plantar-semente.md
</execution_context>

<process>
Execute o workflow plantar-semente de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/plantar-semente.md do início ao fim.
</process>
</output>
