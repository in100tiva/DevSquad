---
name: gsd-auditar-tau
description: "Auditoria entre fases de todos os itens TAU e verificação pendentes"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-auditar-tau` ou descreve uma tarefa correspondente a esta skill.
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
Varrer todas as fases em busca de itens TAU pendentes, ignorados, bloqueados e que necessitam intervenção humana. Cruzar referências com a base de código para detectar documentação desatualizada. Produzir plano de teste humano priorizado.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/auditar-tau.md
</execution_context>

<context>
Arquivos centrais de planejamento são carregados dentro do workflow via CLI.

**Escopo:**
Glob: .planning/phases/*/*-TAU.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>
</output>
