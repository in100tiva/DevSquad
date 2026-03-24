---
name: gsd-proximo
description: "Avançar automaticamente para o próximo passo lógico no workflow GSD"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-proximo` ou descreve uma tarefa correspondente a esta skill.
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
Detectar o estado atual do projeto e invocar automaticamente o próximo passo lógico do workflow GSD.
Sem argumentos necessários — lê STATE.md, ROADMAP.md e diretórios de fase para determinar o que vem a seguir.

Projetado para workflows rápidos multi-projeto onde lembrar em qual fase/passo você está é overhead.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/proximo.md
</execution_context>

<process>
Execute o workflow proximo de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/proximo.md de ponta a ponta.
</process>
</output>
