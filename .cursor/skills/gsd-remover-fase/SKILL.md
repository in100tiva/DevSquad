---
name: gsd-remover-fase
description: "Remover uma fase futura do roteiro e renumerar fases subsequentes"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-remover-fase` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, pergunte de forma conversacional:
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
Remover uma fase futura não iniciada do roteiro e renumerar todas as fases subsequentes para manter uma sequência limpa e linear.

Propósito: Remoção limpa de trabalho que você decidiu não fazer, sem poluir o contexto com marcadores de cancelado/adiado.
Saída: Fase deletada, todas as fases subsequentes renumeradas, commit git como registro histórico.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/remover-fase.md
</execution_context>

<context>
Fase: {{GSD_ARGS}}

O roteiro e o estado são resolvidos dentro do workflow via `init phase-op` e leituras direcionadas.
</context>

<process>
Execute o workflow remover-fase de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/remover-fase.md do início ao fim.
Preserve todas as portas de validação (verificação de fase futura, verificação de trabalho), lógica de renumeração e commit.
</process>
</output>
