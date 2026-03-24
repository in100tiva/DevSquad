---
name: gsd-revisar
description: "Solicitar revisão por pares de IAs externas para planos de fase"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-revisar` ou descreve uma tarefa correspondente a esta skill.
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
Invocar CLIs de IAs externas (Gemini, Claude, Codex) para revisar planos de fase independentemente.
Produz um REVIEWS.md estruturado com feedback por revisor que pode ser alimentado de volta ao
planejamento via /gsd-planejar-fase --reviews.

**Fluxo:** Detectar CLIs → Construir prompt de revisão → Invocar cada CLI → Coletar respostas → Escrever REVIEWS.md
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/revisar.md
</execution_context>

<context>
Número da fase: extraído de {{GSD_ARGS}} (obrigatório)

**Flags:**
- `--gemini` — Incluir revisão do Gemini CLI
- `--claude` — Incluir revisão do Claude CLI (usa sessão separada)
- `--codex` — Incluir revisão do Codex CLI
- `--all` — Incluir todos os CLIs disponíveis
</context>

<process>
Execute o workflow de revisão de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/revisar.md do início ao fim.
</process>
