---
name: gsd-nota
description: "Captura de ideias sem fricção. Adicionar, listar ou promover notas para todos."
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-nota` ou descreve uma tarefa correspondente a esta skill.
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
Captura de ideias sem fricção — uma chamada Write, uma linha de confirmação.

Três subcomandos:
- **append** (padrão): Salva um arquivo de nota com timestamp. Sem perguntas, sem formatação.
- **list**: Mostra todas as notas dos escopos projeto e global.
- **promote**: Converte uma nota em um todo estruturado.

Executa inline — sem Task, sem interação conversacional, sem Bash.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/nota.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
{{GSD_ARGS}}
</context>

<process>
Execute o workflow de nota de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/nota.md do início ao fim.
Capture a nota, liste notas ou promova para todo — dependendo dos argumentos.
</process>
</output>
