---
name: gsd-novo-marco
description: "Iniciar um novo ciclo de marco — atualizar PROJECT.md e rotear para requisitos"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-novo-marco` ou descreve uma tarefa correspondente a esta skill.
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
Iniciar um novo marco: questionamento → pesquisa (opcional) → requisitos → roteiro.

Equivalente brownfield de novo-projeto. Projeto existe, PROJECT.md tem histórico. Coleta "qual é o próximo passo", atualiza PROJECT.md, depois executa ciclo de requisitos → roteiro.

**Cria/Atualiza:**
- `.planning/PROJECT.md` — atualizado com objetivos do novo marco
- `.planning/research/` — pesquisa de domínio (opcional, apenas funcionalidades NOVAS)
- `.planning/REQUIREMENTS.md` — requisitos com escopo para este marco
- `.planning/ROADMAP.md` — estrutura de fases (continua numeração)
- `.planning/STATE.md` — resetado para novo marco

**Após:** `/gsd-planejar-fase [N]` para iniciar execução.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/novo-marco.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/questionamento.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/projeto.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/requisitos.md
</execution_context>

<context>
Nome do marco: {{GSD_ARGS}} (opcional - solicitará se não fornecido)

Arquivos de contexto do projeto e marco são resolvidos dentro do workflow (`init new-milestone`) e delegados via blocos `<files_to_read>` onde subagentes são usados.
</context>

<process>
Execute o workflow novo-marco de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/novo-marco.md de ponta a ponta.
Preserve todas as portas do workflow (validação, questionamento, pesquisa, requisitos, aprovação do roteiro, commits).
</process>
</output>
