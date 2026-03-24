---
name: gsd-inserir-fase
description: "Inserir trabalho urgente como fase decimal (ex: 72.1) entre fases existentes"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-inserir-fase` ou descreve uma tarefa correspondente a esta skill.
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
Inserir uma fase decimal para trabalho urgente descoberto no meio do marco que precisa ser concluído entre fases inteiras existentes.

Usa numeração decimal (72.1, 72.2, etc.) para preservar a sequência lógica das fases planejadas enquanto acomoda inserções urgentes.

Propósito: Lidar com trabalho urgente descoberto durante a execução sem renumerar todo o roteiro.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/inserir-fase.md
</execution_context>

<context>
Argumentos: {{GSD_ARGS}} (formato: <número-da-fase-anterior> <descrição>)

O roteiro e o estado são resolvidos dentro do workflow via `init phase-op` e chamadas de ferramentas direcionadas.
</context>

<process>
Execute o workflow inserir-fase de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/inserir-fase.md do início ao fim.
Preserve todas as portas de validação (análise de argumentos, verificação de fase, cálculo decimal, atualizações do roteiro).
</process>
</output>
