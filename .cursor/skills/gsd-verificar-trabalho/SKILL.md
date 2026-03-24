---
name: gsd-verificar-trabalho
description: "Validar funcionalidades construídas através de TAU conversacional"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-verificar-trabalho` ou descreve uma tarefa correspondente a esta skill.
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
Validar funcionalidades construídas através de testes conversacionais com estado persistente.

Propósito: Confirmar que o que o Claude construiu realmente funciona da perspectiva do usuário. Um teste por vez, respostas em texto simples, sem interrogatório. Quando problemas são encontrados, diagnosticar automaticamente, planejar correções e preparar para execução.

Saída: {phase_num}-TAU.md rastreando todos os resultados de teste. Se problemas encontrados: lacunas diagnosticadas, planos de correção verificados prontos para /gsd-executar-fase
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/verificar-trabalho.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/TAU.md
</execution_context>

<context>
Fase: {{GSD_ARGS}} (opcional)
- Se fornecido: Testar fase específica (ex: "4")
- Se não fornecido: Verificar sessões ativas ou solicitar fase

Arquivos de contexto são resolvidos dentro do workflow (`init verify-work`) e delegados via blocos `<files_to_read>`.
</context>

<process>
Execute o workflow verificar-trabalho de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/verificar-trabalho.md de ponta a ponta.
Preserve todas as portas do workflow (gerenciamento de sessão, apresentação de testes, diagnóstico, planejamento de correção, roteamento).
</process>
</output>
