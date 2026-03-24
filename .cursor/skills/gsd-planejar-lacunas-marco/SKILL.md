---
name: gsd-planejar-lacunas-marco
description: "Criar fases para fechar todas as lacunas identificadas pela auditoria de marco"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-planejar-lacunas-marco` ou descreve uma tarefa correspondente a esta skill.
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
Criar todas as fases necessárias para fechar lacunas identificadas pelo `/gsd-auditar-marco`.

Lê MILESTONE-AUDIT.md, agrupa lacunas em fases lógicas, cria entradas de fase no ROADMAP.md e oferece planejar cada fase.

Um comando cria todas as fases de correção — sem necessidade de `/gsd-adicionar-fase` manual por lacuna.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/planejar-lacunas-marco.md
</execution_context>

<context>
**Resultados da auditoria:**
Glob: .planning/v*-MILESTONE-AUDIT.md (use o mais recente)

A intenção original e o estado atual de planejamento são carregados sob demanda dentro do workflow.
</context>

<process>
Execute o workflow planejar-lacunas-marco de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/planejar-lacunas-marco.md do início ao fim.
Preserve todas as portas do workflow (carregamento de auditoria, priorização, agrupamento de fases, confirmação do usuário, atualizações do roteiro).
</process>
</output>
