---
name: gsd-resumo-marco
description: "Gerar um resumo abrangente do projeto a partir dos artefatos do marco para integração da equipe e revisão"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-resumo-marco` ou descreve uma tarefa correspondente a esta skill.
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
Gerar um resumo estruturado do marco para integração da equipe e revisão do projeto. Lê artefatos do marco concluído (ROADMAP, REQUIREMENTS, CONTEXT, SUMMARY, arquivos VERIFICATION) e produz uma visão geral amigável do que foi construído, como e por quê.

Propósito: Permitir que novos membros da equipe entendam um projeto concluído lendo um único documento e fazendo perguntas de acompanhamento.
Saída: MILESTONE_SUMMARY escrito em `.planning/reports/`, apresentado inline, Q&A interativo opcional.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/resumo-marco.md
</execution_context>

<context>
**Arquivos do projeto:**
- `.planning/ROADMAP.md`
- `.planning/PROJECT.md`
- `.planning/STATE.md`
- `.planning/RETROSPECTIVE.md`
- `.planning/milestones/v{version}-ROADMAP.md` (se arquivado)
- `.planning/milestones/v{version}-REQUIREMENTS.md` (se arquivado)
- `.planning/phases/*-*/` (SUMMARY.md, VERIFICATION.md, CONTEXT.md, RESEARCH.md)

**Entrada do usuário:**
- Versão: {{GSD_ARGS}} (opcional — padrão é o marco atual/mais recente)
</context>

<process>
Leia e execute o workflow resumo-marco de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/resumo-marco.md do início ao fim.
</process>

<success_criteria>
- Versão do marco resolvida (dos argumentos, STATE.md ou varredura de arquivo)
- Todos os artefatos disponíveis lidos (ROADMAP, REQUIREMENTS, CONTEXT, SUMMARY, VERIFICATION, RESEARCH, RETROSPECTIVE)
- Documento de resumo escrito em `.planning/reports/MILESTONE_SUMMARY-v{version}.md`
- Todas as 7 seções geradas (Visão Geral, Arquitetura, Fases, Decisões, Requisitos, Débito Técnico, Primeiros Passos)
- Resumo apresentado inline ao usuário
- Q&A interativo oferecido
- STATE.md atualizado
</success_criteria>
</output>
