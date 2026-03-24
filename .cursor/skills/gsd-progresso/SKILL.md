---
name: gsd-progresso
description: "Verificar progresso do projeto, mostrar contexto e direcionar para próxima ação (executar ou planejar)"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-progresso` ou descreve uma tarefa correspondente a esta skill.
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
Verificar progresso do projeto, resumir trabalho recente e o que vem pela frente, depois direcionar inteligentemente para a próxima ação - seja executar um plano existente ou criar o próximo.

Fornece consciência situacional antes de continuar o trabalho.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/progresso.md
</execution_context>

<process>
Execute o workflow de progresso de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/progresso.md do início ao fim.
Preserve toda lógica de roteamento (Rotas A até F) e tratamento de casos extremos.
</process>
