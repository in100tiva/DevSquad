---
name: gsd-ajuda
description: "Mostrar comandos GSD disponíveis e guia de uso"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-ajuda` ou descreve uma tarefa correspondente a esta skill.
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
Exibir a referência completa de comandos GSD.

Exiba APENAS o conteúdo da referência abaixo. NÃO adicione:
- Análise específica do projeto
- Status do git ou contexto de arquivos
- Sugestões de próximos passos
- Qualquer comentário além da referência
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/ajuda.md
</execution_context>

<process>
Exiba a referência completa de comandos GSD de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/ajuda.md.
Exiba o conteúdo da referência diretamente — sem adições ou modificações.
</process>
</output>
