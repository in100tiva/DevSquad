---
name: gsd-saude
description: "Diagnosticar saúde do diretório de planejamento e opcionalmente reparar problemas"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-saude` ou descreve uma tarefa correspondente a esta skill.
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
Validar integridade do diretório `.planning/` e reportar problemas acionáveis. Verifica arquivos ausentes, configurações inválidas, estado inconsistente e planos órfãos.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/saude.md
</execution_context>

<process>
Execute o workflow de saúde de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/saude.md do início ao fim.
Parse a flag --repair dos argumentos e passe para o workflow.
</process>
