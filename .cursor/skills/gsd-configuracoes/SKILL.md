---
name: gsd-configuracoes
description: "Configurar toggles de workflow GSD e perfil de modelo"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-configuracoes` ou descreve uma tarefa correspondente a esta skill.
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
Configuração interativa dos agentes de workflow GSD e perfil de modelo via prompt de múltiplas perguntas.

Direciona para o workflow de configurações que gerencia:
- Garantia de existência do config
- Leitura e parsing das configurações atuais
- Prompt interativo de 5 perguntas (modelo, pesquisa, verificação_plano, verificador, branching)
- Mesclagem e escrita do config
- Exibição de confirmação com referências rápidas de comando
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/configuracoes.md
</execution_context>

<process>
**Siga o workflow de configurações** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/configuracoes.md`.

O workflow gerencia toda a lógica incluindo:
1. Criação do arquivo de config com padrões se ausente
2. Leitura do config atual
3. Apresentação interativa de configurações com pré-seleção
4. Parsing de respostas e mesclagem de config
5. Escrita do arquivo
6. Exibição de confirmação
</process>
