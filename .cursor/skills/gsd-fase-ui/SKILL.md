---
name: gsd-fase-ui
description: "Gerar contrato de design UI (UI-SPEC.md) para fases de frontend"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-fase-ui` ou descreve uma tarefa correspondente a esta skill.
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
Criar um contrato de design UI (UI-SPEC.md) para uma fase de frontend.
Orquestra gsd-pesquisador-ui e gsd-verificador-ui.
Fluxo: Validar → Pesquisar UI → Verificar UI-SPEC → Concluído
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/fase-ui.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Número da fase: {{GSD_ARGS}} — opcional, detecta automaticamente a próxima fase não planejada se omitido.
</context>

<process>
Execute @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/fase-ui.md do início ao fim.
Preserve todas as portas do workflow.
</process>
</output>
