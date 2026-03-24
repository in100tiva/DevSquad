---
name: gsd-novo-workspace
description: "Criar um workspace isolado com cópias de repos e .planning/ independente"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-novo-workspace` ou descreve uma tarefa correspondente a esta skill.
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

<context>
**Flags:**
- `--name` (obrigatório) — Nome do workspace
- `--repos` — Caminhos ou nomes de repos separados por vírgula. Se omitido, seleção interativa dos repos git filhos no cwd
- `--path` — Diretório destino. Padrão: `~/gsd-workspaces/<name>`
- `--strategy` — `worktree` (padrão, leve) ou `clone` (totalmente independente)
- `--branch` — Branch para checkout. Padrão: `workspace/<name>`
- `--auto` — Pular perguntas interativas, usar padrões
</context>

<objective>
Criar um diretório de workspace físico contendo cópias de repos git especificados (como worktrees ou clones) com um diretório `.planning/` independente para sessões GSD isoladas.

**Casos de uso:**
- Orquestração multi-repo: trabalhar em um subconjunto de repos em paralelo com estado GSD isolado
- Isolamento de feature branch: criar uma worktree do repo atual com seu próprio `.planning/`

**Cria:**
- `<path>/WORKSPACE.md` — manifesto do workspace
- `<path>/.planning/` — diretório de planejamento independente
- `<path>/<repo>/` — git worktree ou clone para cada repo especificado

**Após este comando:** `cd` no workspace e execute `/gsd-novo-projeto` para inicializar o GSD.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/novo-workspace.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<process>
Execute o workflow novo-workspace de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/novo-workspace.md do início ao fim.
Preserve todas as portas do workflow (validação, aprovações, commits, roteamento).
</process>
</output>
