---
name: gsd-listar-premissas-fase
description: "Revelar as premissas do Claude sobre a abordagem de uma fase antes do planejamento"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-listar-premissas-fase` ou descreve uma tarefa correspondente a esta skill.
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
Analisar uma fase e apresentar as premissas do Claude sobre abordagem técnica, ordem de implementação, limites de escopo, áreas de risco e dependências.

Propósito: Ajudar usuários a ver o que o Claude pensa ANTES do planejamento começar — permitindo correção de curso cedo quando as premissas estão erradas.
Saída: Apenas saída conversacional (sem criação de arquivo) — termina com prompt "O que você acha?"
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/listar-premissas-fase.md
</execution_context>

<context>
Número da fase: {{GSD_ARGS}} (obrigatório)

Estado do projeto e roteiro são carregados dentro do workflow usando leituras direcionadas.
</context>

<process>
1. Validar argumento de número de fase (erro se ausente ou inválido)
2. Verificar se a fase existe no roteiro
3. Seguir workflow listar-premissas-fase.md:
   - Analisar descrição do roteiro
   - Revelar premissas sobre: abordagem técnica, ordem de implementação, escopo, riscos, dependências
   - Apresentar premissas claramente
   - Perguntar "O que você acha?"
4. Coletar feedback e oferecer próximos passos
</process>

<success_criteria>

- Fase validada contra o roteiro
- Premissas reveladas em cinco áreas
- Usuário consultado para feedback
- Usuário sabe dos próximos passos (discutir contexto, planejar fase ou corrigir premissas)
  </success_criteria>
</output>
