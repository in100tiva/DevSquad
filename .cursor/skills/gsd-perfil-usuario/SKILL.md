---
name: gsd-perfil-usuario
description: "Gerar perfil comportamental do desenvolvedor e criar artefatos descobríveis pelo Claude"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-perfil-usuario` ou descreve uma tarefa correspondente a esta skill.
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
Gerar um perfil comportamental do desenvolvedor a partir de análise de sessão (ou questionário) e produzir artefatos (USER-PROFILE.md, /gsd-preferencias-dev, seção .cursor/rules/) que personalizam as respostas do Claude.

Direciona para o workflow de perfil-usuario que orquestra o fluxo completo: gate de consentimento, análise de sessão ou fallback por questionário, geração de perfil, exibição de resultado e seleção de artefatos.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/perfil-usuario.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</execution_context>

<context>
Flags de {{GSD_ARGS}}:
- `--questionnaire` -- Pular análise de sessão completamente, usar caminho somente por questionário
- `--refresh` -- Reconstruir perfil mesmo quando já existe, fazer backup do perfil antigo, mostrar diff de dimensões
</context>

<process>
Execute o workflow de perfil-usuario do início ao fim.

O workflow gerencia toda a lógica incluindo:
1. Inicialização e detecção de perfil existente
2. Gate de consentimento antes da análise de sessão
3. Varredura de sessão e verificações de suficiência de dados
4. Análise de sessão (agente perfilador) ou fallback por questionário
5. Resolução de divisão entre projetos
6. Escrita do perfil em USER-PROFILE.md
7. Exibição de resultado com boletim e destaques
8. Seleção de artefatos (preferencias-dev, seções .cursor/rules/)
9. Geração sequencial de artefatos
10. Resumo com diff de atualização (se aplicável)
</process>
