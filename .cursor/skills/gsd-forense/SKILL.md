---
name: gsd-forense
description: "Investigação post-mortem para workflows GSD falhos — analisa histórico git, artefatos e estado para diagnosticar o que deu errado"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-forense` ou descreve uma tarefa correspondente a esta skill.
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
Investigar o que deu errado durante a execução de um workflow GSD. Analisa histórico git, artefatos `.planning/` e estado do sistema de arquivos para detectar anomalias e gerar um relatório diagnóstico estruturado.

Propósito: Diagnosticar workflows falhos ou travados para que o usuário possa entender a causa raiz e tomar ação corretiva.
Saída: Relatório forense salvo em `.planning/forensics/`, apresentado inline, com criação opcional de issue.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/forense.md
</execution_context>

<context>
**Fontes de dados:**
- `git log` (commits recentes, padrões, intervalos de tempo)
- `git status` / `git diff` (trabalho não commitado, conflitos)
- `.planning/STATE.md` (posição atual, histórico de sessão)
- `.planning/ROADMAP.md` (escopo e progresso das fases)
- `.planning/phases/*/` (PLAN.md, SUMMARY.md, VERIFICATION.md, CONTEXT.md)
- `.planning/reports/SESSION_REPORT.md` (resultados da última sessão)

**Entrada do usuário:**
- Descrição do problema: {{GSD_ARGS}} (opcional — perguntará se não fornecido)
</context>

<process>
Leia e execute o workflow forense de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/forense.md do início ao fim.
</process>

<success_criteria>
- Evidências coletadas de todas as fontes de dados disponíveis
- Pelo menos 4 tipos de anomalia verificados (loop travado, artefatos ausentes, trabalho abandonado, crash/interrupção)
- Relatório forense estruturado escrito em `.planning/forensics/report-{timestamp}.md`
- Relatório apresentado inline com descobertas, anomalias e recomendações
- Investigação interativa oferecida para análise mais profunda
- Criação de issue no GitHub oferecida se existirem descobertas acionáveis
</success_criteria>

<critical_rules>
- **Investigação somente leitura:** Não modifique arquivos fonte do projeto durante a forense. Apenas escreva o relatório forense e atualize o rastreamento de sessão do STATE.md.
- **Redatar dados sensíveis:** Remova caminhos absolutos, chaves de API, tokens de relatórios e issues.
- **Fundamentar descobertas em evidências:** Toda anomalia deve citar commits, arquivos ou dados de estado específicos.
- **Sem especulação sem evidência:** Se os dados são insuficientes, diga isso — não fabrique causas raiz.
</critical_rules>
</output>
