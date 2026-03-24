---
name: gsd-adicionar-fase
description: "Adicionar fase ao final do marco atual no roteiro"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-adicionar-fase` ou descreve uma tarefa correspondente a esta skill.
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
Adicionar uma nova fase inteira ao final do marco atual no roteiro.

Direciona para o workflow adicionar-fase que gerencia:
- Cálculo do número da fase (próximo inteiro sequencial)
- Criação do diretório com geração de slug
- Atualizações na estrutura do roteiro
- Rastreamento de evolução do STATE.md
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/adicionar-fase.md
</execution_context>

<context>
Argumentos: {{GSD_ARGS}} (descrição da fase)

O roteiro e o estado são resolvidos dentro do workflow via `init phase-op` e chamadas de ferramentas direcionadas.
</context>

<process>
**Siga o workflow adicionar-fase** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/adicionar-fase.md`.

O workflow gerencia toda a lógica incluindo:
1. Análise e validação de argumentos
2. Verificação de existência do roteiro
3. Identificação do marco atual
4. Cálculo do próximo número de fase (ignorando decimais)
5. Geração de slug a partir da descrição
6. Criação do diretório da fase
7. Inserção da entrada no roteiro
8. Atualizações do STATE.md
</process>
</output>
