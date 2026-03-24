---
name: gsd-retomar-trabalho
description: "Retomar trabalho da sessão anterior com restauração completa de contexto"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-retomar-trabalho` ou descreve uma tarefa correspondente a esta skill.
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
Restaurar contexto completo do projeto e retomar trabalho de forma contínua a partir da sessão anterior.

Roteia para o workflow retomar-projeto que gerencia:

- Carregamento do STATE.md (ou reconstrução se ausente)
- Detecção de checkpoint (arquivos .continue-here)
- Detecção de trabalho incompleto (PLAN sem SUMMARY)
- Apresentação do status
- Roteamento de próxima ação com reconhecimento de contexto
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/retomar-projeto.md
</execution_context>

<process>
**Siga o workflow retomar-projeto** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/retomar-projeto.md`.

O workflow gerencia toda a lógica de retomada incluindo:

1. Verificação de existência do projeto
2. Carregamento ou reconstrução do STATE.md
3. Detecção de checkpoint e trabalho incompleto
4. Apresentação visual do status
5. Oferta de opções com reconhecimento de contexto (verifica CONTEXT.md antes de sugerir planejar vs discutir)
6. Roteamento para o próximo comando apropriado
7. Atualizações de continuidade da sessão
</process>
</output>
