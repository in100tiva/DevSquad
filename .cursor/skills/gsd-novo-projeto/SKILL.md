---
name: gsd-novo-projeto
description: "Inicializar um novo projeto com coleta profunda de contexto e PROJECT.md"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-novo-projeto` ou descreve uma tarefa correspondente a esta skill.
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

<context>
**Flags:**
- `--auto` — Modo automático. Após perguntas de configuração, executa pesquisa → requisitos → roteiro sem mais interação. Espera documento de ideia via referência @.
</context>

<objective>
Inicializar um novo projeto através de fluxo unificado: questionamento → pesquisa (opcional) → requisitos → roteiro.

**Cria:**
- `.planning/PROJECT.md` — contexto do projeto
- `.planning/config.json` — preferências de workflow
- `.planning/research/` — pesquisa de domínio (opcional)
- `.planning/REQUIREMENTS.md` — requisitos com escopo definido
- `.planning/ROADMAP.md` — estrutura de fases
- `.planning/STATE.md` — memória do projeto

**Após este comando:** Execute `/gsd-planejar-fase 1` para iniciar a execução.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/novo-projeto.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/questionamento.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/projeto.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/requisitos.md
</execution_context>

<process>
Execute o workflow novo-projeto de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/novo-projeto.md de ponta a ponta.
Preserve todas as portas do workflow (validação, aprovações, commits, roteamento).
</process>
</output>
