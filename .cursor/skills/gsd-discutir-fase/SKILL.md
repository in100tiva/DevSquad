---
name: gsd-discutir-fase
description: "Coletar contexto da fase através de questionamento adaptativo antes do planejamento. Use --auto para pular perguntas interativas (Claude escolhe padrões recomendados)."
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-discutir-fase` ou descreve uma tarefa correspondente a esta skill.
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
Extrair decisões de implementação que agentes downstream precisam — pesquisador e planejador usarão CONTEXT.md para saber o que investigar e quais escolhas estão definidas.

**Como funciona:**
1. Carregar contexto anterior (PROJECT.md, REQUIREMENTS.md, STATE.md, arquivos CONTEXT.md anteriores)
2. Explorar codebase em busca de assets e padrões reutilizáveis
3. Analisar fase — pular áreas cinzentas já decididas em fases anteriores
4. Apresentar áreas cinzentas restantes — usuário seleciona quais discutir
5. Aprofundar cada área selecionada até satisfação
6. Criar CONTEXT.md com decisões que guiam pesquisa e planejamento

**Saída:** `{phase_num}-CONTEXT.md` — decisões claras o suficiente para que agentes downstream possam agir sem perguntar ao usuário novamente
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/discutir-fase.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/discutir-premissas-fase.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/contexto.md
</execution_context>

<context>
Número da fase: {{GSD_ARGS}} (obrigatório)

Arquivos de contexto são resolvidos dentro do workflow usando `init phase-op` e chamadas de ferramentas de roteiro/estado.
</context>

<process>
**Roteamento de modo:**
```bash
DISCUSS_MODE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.discuss_mode 2>/dev/null || echo "discuss")
```

Se `DISCUSS_MODE` for `"assumptions"`: Leia e execute @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/discutir-premissas-fase.md de ponta a ponta.

Se `DISCUSS_MODE` for `"discuss"` (ou não definido, ou qualquer outro valor): Leia e execute @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/discutir-fase.md de ponta a ponta.

**OBRIGATÓRIO:** Os arquivos de execution_context listados acima SÃO as instruções. Leia o arquivo do workflow ANTES de tomar qualquer ação. As seções objective e success_criteria neste arquivo de comando são resumos — o arquivo do workflow contém o processo passo a passo completo com todos os comportamentos necessários, verificações de configuração e padrões de interação. Não improvise a partir do resumo.
</process>

<success_criteria>
- Contexto anterior carregado e aplicado (sem repetir perguntas já decididas)
- Áreas cinzentas identificadas através de análise inteligente
- Usuário escolheu quais áreas discutir
- Cada área selecionada explorada até satisfação
- Expansão de escopo redirecionada para ideias adiadas
- CONTEXT.md captura decisões, não visão vaga
- Usuário sabe os próximos passos
</success_criteria>
</output>
