---
name: gsd-rapido-garantido
description: "Executar uma tarefa rápida com garantias GSD (commits atômicos, rastreamento de estado) mas pular agentes opcionais"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-rapido-garantido` ou descreve uma tarefa correspondente a esta skill.
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
Executar tarefas pequenas e ad-hoc com garantias GSD (commits atômicos, rastreamento STATE.md).

Modo rápido é o mesmo sistema com um caminho mais curto:
- Cria gsd-planejador (modo rápido) + gsd-executor(es)
- Tarefas rápidas ficam em `.planning/quick/` separadas das fases planejadas
- Atualiza tabela "Tarefas Rápidas Concluídas" do STATE.md (NÃO o ROADMAP.md)

**Padrão:** Pula pesquisa, discussão, verificador de plano, verificador. Use quando você sabe exatamente o que fazer.

**Flag `--discuss`:** Fase de discussão leve antes do planejamento. Expõe premissas, clarifica áreas cinzentas, captura decisões no CONTEXT.md. Use quando a tarefa tem ambiguidade que vale resolver antecipadamente.

**Flag `--full`:** Habilita verificação de plano (máximo 2 iterações) e verificação pós-execução. Use quando você quer garantias de qualidade sem a cerimônia completa de marco.

**Flag `--research`:** Cria um agente de pesquisa focado antes do planejamento. Investiga abordagens de implementação, opções de bibliotecas e armadilhas para a tarefa. Use quando não tem certeza da melhor abordagem.

Flags são combináveis: `--discuss --research --full` fornece discussão + pesquisa + verificação de plano + verificação.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/rapido-garantido.md
</execution_context>

<context>
{{GSD_ARGS}}

Arquivos de contexto são resolvidos dentro do workflow (`init quick`) e delegados via blocos `<files_to_read>`.
</context>

<process>
Execute o workflow rápido garantido de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/rapido-garantido.md do início ao fim.
Preserve todos os gates do workflow (validação, descrição da tarefa, planejamento, execução, atualizações de estado, commits).
</process>
