---
name: gsd-executar-fase
description: "Executar todos os planos de uma fase com paralelização baseada em ondas"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-executar-fase` ou descreve uma tarefa correspondente a esta skill.
- Trate todo o texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, solicite conversacionalmente:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Geração de Subagentes
Quando o workflow precisar gerar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

<objective>
Executar todos os planos de uma fase usando execução paralela baseada em ondas.

O orquestrador permanece enxuto: descobrir planos, analisar dependências, agrupar em ondas, gerar subagentes, coletar resultados. Cada subagente carrega o contexto completo de executar-plano e gerencia seu próprio plano.

Filtro de onda opcional:
- `--wave N` executa apenas a Onda `N` para controle de ritmo, gestão de cota ou implantação escalonada
- verificação/conclusão da fase só acontece quando não restam planos incompletos após a onda selecionada terminar

Regra de tratamento de flags:
- As flags opcionais documentadas abaixo são comportamentos disponíveis, não comportamentos implicitamente ativos
- Uma flag está ativa apenas quando seu token literal aparece em `{{GSD_ARGS}}`
- Se uma flag documentada está ausente de `{{GSD_ARGS}}`, trate-a como inativa

Orçamento de contexto: ~15% orquestrador, 100% novo por subagente.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/executar-fase.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Fase: {{GSD_ARGS}}

**Flags opcionais disponíveis (apenas documentação — não automaticamente ativas):**
- `--wave N` — Executar apenas a Onda `N` na fase. Use quando quiser controlar o ritmo de execução ou ficar dentro dos limites de uso.
- `--gaps-only` — Executar apenas planos de fechamento de lacunas (planos com `gap_closure: true` no frontmatter). Use após verificar-trabalho criar planos de correção.
- `--interactive` — Executar planos sequencialmente inline (sem subagentes) com checkpoints do usuário entre tarefas. Menor uso de tokens, estilo pair-programming. Melhor para fases pequenas, correções de bugs e lacunas de verificação.

**Flags ativas devem ser derivadas de `{{GSD_ARGS}}`:**
- `--wave N` está ativa apenas se o token literal `--wave` estiver presente em `{{GSD_ARGS}}`
- `--gaps-only` está ativa apenas se o token literal `--gaps-only` estiver presente em `{{GSD_ARGS}}`
- `--interactive` está ativa apenas se o token literal `--interactive` estiver presente em `{{GSD_ARGS}}`
- Se nenhum desses tokens aparecer, execute o fluxo padrão de execução completa da fase sem filtragem por flag
- Não infira que uma flag está ativa apenas porque está documentada neste prompt

Arquivos de contexto são resolvidos dentro do workflow via `gsd-tools init execute-phase` e blocos `<files_to_read>` por subagente.
</context>

<process>
Execute o workflow executar-fase de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/executar-fase.md de ponta a ponta.
Preserve todas as portas do workflow (execução de ondas, tratamento de checkpoints, verificação, atualizações de estado, roteamento).
</process>
</output>
