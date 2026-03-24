---
name: gsd-adicionar-testes
description: "Gerar testes para uma fase concluída com base nos critérios TAU e implementação"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-adicionar-testes` ou descreve uma tarefa correspondente a esta skill.
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
Gerar testes unitários e E2E para uma fase concluída, usando seu SUMMARY.md, CONTEXT.md e VERIFICATION.md como especificações.

Analisa arquivos de implementação, classifica-os em categorias TDD (unitário), E2E (navegador) ou Ignorar, apresenta um plano de testes para aprovação do usuário e então gera os testes seguindo convenções RED-GREEN.

Saída: Arquivos de teste commitados com mensagem `test(phase-{N}): add unit and E2E tests from add-tests command`
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/adicionar-testes.md
</execution_context>

<context>
Fase: {{GSD_ARGS}}

@.planning/STATE.md
@.planning/ROADMAP.md
</context>

<process>
Execute o workflow adicionar-testes de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/adicionar-testes.md do início ao fim.
Preserve todas as portas do workflow (aprovação de classificação, aprovação do plano de testes, verificação RED-GREEN, relatório de lacunas).
</process>
</output>
