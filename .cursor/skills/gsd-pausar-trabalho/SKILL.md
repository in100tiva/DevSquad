---
name: gsd-pausar-trabalho
description: "Criar handoff de contexto ao pausar trabalho no meio de uma fase"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-pausar-trabalho` ou descreve uma tarefa correspondente a esta skill.
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
Criar arquivo de handoff `.continue-here.md` para preservar o estado completo do trabalho entre sessões.

Roteia para o workflow pausar-trabalho que gerencia:
- Detecção da fase atual a partir de arquivos recentes
- Coleta completa de estado (posição, trabalho concluído, trabalho restante, decisões, bloqueadores)
- Criação do arquivo de handoff com todas as seções de contexto
- Commit no Git como WIP
- Instruções de retomada
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/pausar-trabalho.md
</execution_context>

<context>
Estado e progresso da fase são coletados no workflow com leituras direcionadas.
</context>

<process>
**Siga o workflow pausar-trabalho** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/pausar-trabalho.md`.

O workflow gerencia toda a lógica incluindo:
1. Detecção do diretório da fase
2. Coleta de estado com esclarecimentos do usuário
3. Escrita do arquivo de handoff com timestamp
4. Commit no Git
5. Confirmação com instruções de retomada
</process>
</output>
