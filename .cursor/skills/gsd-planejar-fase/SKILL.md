---
name: gsd-planejar-fase
description: "Criar plano detalhado de fase (PLAN.md) com ciclo de verificação"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-planejar-fase` ou descreve uma tarefa correspondente a esta skill.
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
Criar prompts executáveis de fase (arquivos PLAN.md) para uma fase do roteiro com pesquisa e verificação integradas.

**Fluxo padrão:** Pesquisa (se necessário) → Planejar → Verificar → Concluído

**Papel do orquestrador:** Analisar argumentos, validar fase, pesquisar domínio (a menos que pulado), gerar gsd-planejador, verificar com gsd-verificador-plano, iterar até aprovação ou máximo de iterações, apresentar resultados.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/planejar-fase.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Número da fase: {{GSD_ARGS}} (opcional — detecta automaticamente a próxima fase não planejada se omitido)

**Flags:**
- `--research` — Forçar re-pesquisa mesmo se RESEARCH.md existir
- `--skip-research` — Pular pesquisa, ir direto ao planejamento
- `--gaps` — Modo de fechamento de lacunas (lê VERIFICATION.md, pula pesquisa)
- `--skip-verify` — Pular ciclo de verificação
- `--prd <arquivo>` — Usar um arquivo PRD/critérios de aceitação em vez de discutir-fase. Analisa requisitos no CONTEXT.md automaticamente. Pula discutir-fase inteiramente.
- `--reviews` — Replanejar incorporando feedback de revisão cross-AI do REVIEWS.md (produzido por `/gsd-revisar`)
- `--text` — Usar listas numeradas em texto simples em vez de menus TUI (necessário para sessões `/rc` remotas)

Normalize a entrada de fase no passo 2 antes de qualquer busca de diretório.
</context>

<process>
Execute o workflow planejar-fase de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/planejar-fase.md de ponta a ponta.
Preserve todas as portas do workflow (validação, pesquisa, planejamento, ciclo de verificação, roteamento).
</process>
