---
name: gsd-autonomo
description: "Executar todas as fases restantes autonomamente — discutir→planejar→executar por fase"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-autonomo` ou descreve uma tarefa correspondente a esta skill.
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
Executar todas as fases restantes do marco autonomamente. Para cada fase: discutir → planejar → executar. Pausa apenas para decisões do usuário (aceitação de área cinzenta, bloqueios, solicitações de validação).

Usa descoberta de fases do ROADMAP.md e invocações Skill() planas para cada comando de fase. Após todas as fases completarem: auditoria de marco → completar → limpeza.

**Cria/Atualiza:**
- `.planning/STATE.md` — atualizado após cada fase
- `.planning/ROADMAP.md` — progresso atualizado após cada fase
- Artefatos de fase — CONTEXT.md, PLANs, SUMMARYs por fase

**Após:** Marco está completo e limpo.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/autonomo.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Flag opcional: `--from N` — iniciar da fase N em vez da primeira fase incompleta.

Contexto do projeto, lista de fases e estado são resolvidos dentro do workflow usando comandos init (`gsd-tools.cjs init milestone-op`, `gsd-tools.cjs roadmap analyze`). Nenhum carregamento de contexto prévio necessário.
</context>

<process>
Execute o workflow autonomo de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/autonomo.md de ponta a ponta.
Preserve todas as portas do workflow (descoberta de fases, execução por fase, tratamento de bloqueios, exibição de progresso).
</process>
</output>
