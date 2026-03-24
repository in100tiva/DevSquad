---
name: gsd-gerenciador
description: "Centro de comando interativo para gerenciar múltiplas fases a partir de um terminal"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-gerenciador` ou descreve uma tarefa correspondente a esta skill.
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
Centro de comando em terminal único para gerenciar um marco. Mostra um painel de todas as fases com indicadores visuais de status, recomenda próximas ações ideais e despacha trabalho — discussão roda inline, planejar/executar rodam como agentes em background.

Projetado para usuários avançados que querem paralelizar trabalho entre fases a partir de um terminal: discutir uma fase enquanto outra planeja ou executa em background.

**Cria/Atualiza:**
- Nenhum arquivo criado diretamente — despacha para comandos GSD existentes via Skill() e agentes Task em background.
- Lê `.planning/STATE.md`, `.planning/ROADMAP.md`, diretórios de fase para status.

**Depois:** Usuário sai quando terminar de gerenciar, ou todas as fases completam e ciclo de vida do marco é sugerido.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/gerenciador.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Nenhum argumento necessário. Requer um marco ativo com ROADMAP.md e STATE.md.

Contexto do projeto, lista de fases, dependências e recomendações são resolvidos dentro do workflow usando `gsd-tools.cjs init manager`. Nenhum carregamento de contexto prévio necessário.
</context>

<process>
Execute o workflow do gerenciador de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/gerenciador.md do início ao fim.
Mantenha o loop de atualização do painel até o usuário sair ou todas as fases completarem.
</process>
