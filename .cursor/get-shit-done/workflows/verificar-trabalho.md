<purpose>
Validar funcionalidades implementadas por meio de testes conversacionais com estado persistente. Cria arquivo *-UAT.md que acompanha o progresso dos testes, sobrevive a /clear e alimenta lacunas em /gsd-planejar-fase --gaps.

O usuário testa, o Claude registra. Um teste por vez. Respostas em texto simples.
</purpose>

<philosophy>
**Mostrar o esperado, perguntar se a realidade confere.**

O Claude apresenta o que DEVERIA acontecer. O usuário confirma ou descreve o que é diferente.
- "sim" / "s" / "próximo" / vazio → aprovado
- Qualquer outra coisa → registrado como problema, gravidade inferida

Sem botões Aprovado/Reprovado. Sem perguntas de gravidade. Só: "É assim que deveria ser. Está?"
</philosophy>

<template>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/TAU.md
</template>

<process>

<step name="initialize" priority="first">
Se {{GSD_ARGS}} contiver um número de fase, carregar contexto:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init verify-work "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar JSON para: `planner_model`, `checker_model`, `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `has_verification`.
</step>

<step name="check_active_session">
**Primeiro: verificar sessões TAU ativas**

```bash
find .planning/phases -name "*-UAT.md" -type f 2>/dev/null | head -5
```

**Se existirem sessões ativas E nenhum {{GSD_ARGS}} foi informado:**

Ler o frontmatter de cada arquivo (status, phase) e a seção Teste atual.

Exibir inline:

```
## Sessões TAU ativas

| # | Fase | Status | Teste atual | Progresso |
|---|------|--------|-------------|-----------|
| 1 | 04-comments | testing | 3. Responder ao comentário | 2/6 |
| 2 | 05-auth | testing | 1. Formulário de login | 0/4 |

Responda com um número para retomar, ou informe um número de fase para iniciar nova.
```

Aguardar resposta do usuário.

- Se o usuário responder com número (1, 2) → Carregar esse arquivo, ir para `resume_from_file`
- Se o usuário responder com número de fase → Tratar como nova sessão, ir para `create_uat_file`

**Se existirem sessões ativas E {{GSD_ARGS}} foi informado:**

Verificar se existe sessão para essa fase. Se sim, oferecer retomar ou reiniciar.
Se não, seguir para `create_uat_file`.

**Se não houver sessões ativas E nenhum {{GSD_ARGS}}:**

```
Nenhuma sessão TAU ativa.

Informe um número de fase para iniciar os testes (ex.: /gsd-verificar-trabalho 4)
```

**Se não houver sessões ativas E {{GSD_ARGS}} foi informado:**

Seguir para `create_uat_file`.
</step>

<step name="find_summaries">
**Encontrar o que testar:**

Usar `phase_dir` do init (ou executar init se ainda não foi feito).

```bash
ls "$phase_dir"/*-SUMMARY.md 2>/dev/null
```

Ler cada SUMMARY.md para extrair entregáveis testáveis.
</step>

<step name="extract_tests">
**Extrair entregáveis testáveis do SUMMARY.md:**

Analisar em busca de:
1. **Conquistas** — Funcionalidades adicionadas
2. **Mudanças voltadas ao usuário** — UI, fluxos, interações

Focar em resultados OBSERVÁVEIS pelo usuário, não em detalhes de implementação.

Para cada entregável, criar um teste:
- name: Nome breve do teste
- expected: O que o usuário deve ver/experimentar (específico, observável)

Exemplos:
- Conquista: "Adicionado encadeamento de comentários com aninhamento infinito"
  → Teste: "Responder a um comentário"
  → Esperado: "Clicar em Responder abre o compositor inline abaixo do comentário. Ao enviar, a resposta aparece aninhada sob o pai com recuo visual."

Ignorar itens internos/não observáveis (refactors, mudanças de tipo, etc.).

**Injeção de teste de fumaça cold-start:**

Depois de extrair testes dos SUMMARYs, varrer os arquivos SUMMARY em busca de caminhos de arquivos modificados/criados. Se QUALQUER caminho corresponder a estes padrões:

`server.ts`, `server.js`, `app.ts`, `app.js`, `index.ts`, `index.js`, `main.ts`, `main.js`, `database/*`, `db/*`, `seed/*`, `seeds/*`, `migrations/*`, `startup*`, `docker-compose*`, `Dockerfile*`

Então **prefixar** este teste à lista de testes:

- name: "Teste de fumaça — partida a frio (cold-start)"
- expected: "Encerrar qualquer servidor/serviço em execução. Limpar estado efêmero (DBs temporários, caches, arquivos de lock). Subir a aplicação do zero. O servidor inicia sem erros, qualquer seed/migration conclui, e uma consulta primária (health check, carregamento da home ou chamada básica à API) retorna dados ao vivo."

Isso detecta bugs que só aparecem em partida limpa — condições de corrida na sequência de startup, falhas silenciosas de seed, configuração de ambiente ausente — que passam com estado aquecido mas quebram em produção.
</step>

<step name="create_uat_file">
**Criar arquivo UAT com todos os testes:**

```bash
mkdir -p "$PHASE_DIR"
```

Montar a lista de testes a partir dos entregáveis extraídos.

Criar arquivo:

```markdown
---
status: testing
phase: XX-name
source: [lista de arquivos SUMMARY.md]
started: [timestamp ISO]
updated: [timestamp ISO]
---

## Current Test
<!-- SOBRESCREVER a cada teste — indica onde estamos -->

number: 1
name: [nome do primeiro teste]
expected: |
  [o que o usuário deve observar]
awaiting: user response

## Tests

### 1. [Nome do teste]
expected: [comportamento observável]
result: [pending]

### 2. [Nome do teste]
expected: [comportamento observável]
result: [pending]

...

## Summary

total: [N]
passed: 0
issues: 0
pending: [N]
skipped: 0

## Gaps

[nenhuma ainda]
```

Gravar em `.planning/phases/XX-name/{phase_num}-UAT.md`

Seguir para `present_test`.
</step>

<step name="present_test">
**Apresentar o teste atual ao usuário:**

Ler a seção Current Test do arquivo UAT.

Exibir no formato de caixa de checkpoint:

```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: Verificação necessária                          ║
╚══════════════════════════════════════════════════════════════╝

**Teste {number}: {name}**

{expected}

──────────────────────────────────────────────────────────────
→ Digite "pass" ou descreva o que está errado
──────────────────────────────────────────────────────────────
```

Aguardar resposta do usuário (texto simples, sem prompts conversacionais).
</step>

<step name="process_response">
**Processar a resposta do usuário e atualizar o arquivo:**

**Se a resposta indicar aprovação:**
- Resposta vazia, "yes", "y", "ok", "pass", "next", "approved", "✓", "sim", "s"

Atualizar seção Tests:
```
### {N}. {name}
expected: {expected}
result: pass
```

**Se a resposta indicar pular:**
- "skip", "can't test", "n/a", "pular", "não consigo testar"

Atualizar seção Tests:
```
### {N}. {name}
expected: {expected}
result: skipped
reason: [motivo do usuário se informado]
```

**Se a resposta indicar bloqueio:**
- "blocked", "can't test - server not running", "need physical device", "need release build"
- Ou qualquer resposta contendo: "server", "blocked", "not running", "physical device", "release build"

Inferir tag blocked_by a partir da resposta:
- Contém: server, not running, gateway, API → `server`
- Contém: physical, device, hardware, real phone → `physical-device`
- Contém: release, preview, build, EAS → `release-build`
- Contém: stripe, twilio, third-party, configure → `third-party`
- Contém: depends on, prior phase, prerequisite → `prior-phase`
- Padrão: `other`

Atualizar seção Tests:
```
### {N}. {name}
expected: {expected}
result: blocked
blocked_by: {tag inferida}
reason: "{resposta literal do usuário}"
```

Nota: Testes bloqueados NÃO entram na seção Gaps (não são problemas de código — são pré-requisitos).

**Se a resposta for qualquer outra coisa:**
- Tratar como descrição do problema

Inferir gravidade a partir da descrição:
- Contém: crash, error, exception, fails, broken, unusable → blocker
- Contém: doesn't work, wrong, missing, can't → major
- Contém: slow, weird, off, minor, small → minor
- Contém: color, font, spacing, alignment, visual → cosmetic
- Padrão se não estiver claro: major

Atualizar seção Tests:
```
### {N}. {name}
expected: {expected}
result: issue
reported: "{resposta literal do usuário}"
severity: {inferida}
```

Acrescentar à seção Gaps (YAML estruturado para plan-phase --gaps):
```yaml
- truth: "{comportamento esperado do teste}"
  status: failed
  reason: "Usuário relatou: {resposta literal do usuário}"
  severity: {inferida}
  test: {N}
  artifacts: []  # Preenchido pelo diagnóstico
  missing: []    # Preenchido pelo diagnóstico
```

**Depois de qualquer resposta:**

Atualizar contagens do Summary.
Atualizar timestamp frontmatter.updated.

Se restarem testes → Atualizar Current Test, ir para `present_test`
Se não restarem testes → Ir para `complete_session`
</step>

<step name="resume_from_file">
**Retomar testes a partir do arquivo UAT:**

Ler o arquivo UAT completo.

Encontrar o primeiro teste com `result: [pending]`.

Anunciar:
```
Retomando: TAU da fase {phase}
Progresso: {passed + issues + skipped}/{total}
Problemas encontrados até agora: {contagem de issues}

Continuando a partir do teste {N}...
```

Atualizar a seção Current Test com o teste pendente.
Seguir para `present_test`.
</step>

<step name="complete_session">
**Concluir testes e fazer commit:**

**Determinar status final:**

Contar resultados:
- `pending_count`: testes com `result: [pending]`
- `blocked_count`: testes com `result: blocked`
- `skipped_no_reason`: testes com `result: skipped` e sem campo `reason`

```
if pending_count > 0 OR blocked_count > 0 OR skipped_no_reason > 0:
  status: partial
  # Sessão encerrada mas nem todos os testes resolvidos
else:
  status: complete
  # Todos os testes têm resultado definitivo (pass, issue ou skipped com motivo)
```

Atualizar frontmatter:
- status: {status calculado}
- updated: [agora]

Limpar seção Current Test:
```
## Current Test

[testes concluídos]
```

Fazer commit do arquivo UAT:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "test({phase_num}): TAU concluído - {passed} aprovados, {issues} problemas" --files ".planning/phases/XX-name/{phase_num}-UAT.md"
```

Apresentar resumo:
```
## TAU concluída: fase {phase}

| Resultado | Contagem |
|-----------|----------|
| Aprovados | {N}      |
| Problemas | {N}      |
| Pulados   | {N}      |

[Se issues > 0:]
### Problemas encontrados

[Lista da seção de problemas]
```

**Se issues > 0:** Seguir para `diagnose_issues`

**Se issues == 0:**
```
Todos os testes passaram. Pronto para continuar.

- `/gsd-planejar-fase {next}` — Planejar próxima fase
- `/gsd-executar-fase {next}` — Executar próxima fase
- `/gsd-revisao-ui {phase}` — auditoria visual de qualidade (se arquivos de frontend foram alterados)
```
</step>

<step name="diagnose_issues">
**Diagnosticar causas raiz antes de planejar correções:**

```
---

{N} problemas encontrados. Diagnosticando causas raiz...

Disparando agentes de depuração em paralelo para investigar cada problema.
```

- Carregar workflow diagnosticar-problemas
- Seguir @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/diagnosticar-problemas.md
- Disparar agentes de depuração em paralelo para cada problema
- Coletar causas raiz
- Atualizar UAT.md com as causas raiz
- Seguir para `plan_gap_closure`

O diagnóstico roda automaticamente — sem prompt ao usuário. Agentes em paralelo investigam ao mesmo tempo, então o custo extra é mínimo e as correções ficam mais precisas.
</step>

<step name="plan_gap_closure">
**Planejar correções automaticamente a partir das lacunas diagnosticadas:**

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PLANEJANDO CORREÇÕES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Disparando planejador para fechamento de lacunas...
```

Disparar gsd-planejador no modo --gaps:

```
Task(
  prompt="""
<planning_context>

**Fase:** {phase_number}
**Modo:** gap_closure

<files_to_read>
- {phase_dir}/{phase_num}-UAT.md (UAT com diagnósticos)
- .planning/STATE.md (estado do projeto)
- .planning/ROADMAP.md (roteiro)
</files_to_read>

</planning_context>

<downstream_consumer>
Saída consumida por /gsd-executar-fase
Os planos devem ser prompts executáveis.
</downstream_consumer>
""",
  subagent_type="gsd-planejador",
  model="{planner_model}",
  description="Planejar correções de lacunas para a fase {phase}"
)
```

Ao retornar:
- **PLANEJAMENTO CONCLUÍDO:** Seguir para `verify_gap_plans`
- **PLANEJAMENTO INCONCLUSIVO:** Relatar e oferecer intervenção manual
</step>

<step name="verify_gap_plans">
**Verificar planos de correção com o verificador:**

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO PLANOS DE CORREÇÃO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Disparando verificador de planos...
```

Inicializar: `iteration_count = 1`

Disparar gsd-verificador-plano:

```
Task(
  prompt="""
<verification_context>

**Fase:** {phase_number}
**Objetivo da fase:** Fechar lacunas diagnosticadas a partir do UAT

<files_to_read>
- {phase_dir}/*-PLAN.md (planos a verificar)
</files_to_read>

</verification_context>

<expected_output>
Retornar um dos:
- ## VERIFICATION PASSED — todas as verificações passaram
- ## ISSUES FOUND — lista estruturada de problemas
</expected_output>
""",
  subagent_type="gsd-verificador-plano",
  model="{checker_model}",
  description="Verificar planos de correção da fase {phase}"
)
```

Ao retornar:
- **VERIFICATION PASSED** (verificação aprovada): seguir para `present_ready`
- **ISSUES FOUND** (problemas encontrados): seguir para `revision_loop`
</step>

<step name="revision_loop">
**Iterar planejador ↔ verificador até os planos passarem (máx. 3):**

**Se iteration_count < 3:**

Exibir: `Enviando de volta ao planejador para revisão... (iteração {N}/3)`

Disparar gsd-planejador com contexto de revisão:

```
Task(
  prompt="""
<revision_context>

**Fase:** {phase_number}
**Modo:** revision

<files_to_read>
- {phase_dir}/*-PLAN.md (planos existentes)
</files_to_read>

**Problemas apontados pelo verificador:**
{structured_issues_from_checker}

</revision_context>

<instructions>
Ler os arquivos PLAN.md existentes. Fazer atualizações pontuais para corrigir os problemas do verificador.
NÃO replanejar do zero salvo se os problemas forem fundamentais.
</instructions>
""",
  subagent_type="gsd-planejador",
  model="{planner_model}",
  description="Revisar planos da fase {phase}"
)
```

Depois que o planejador retornar → disparar o verificador novamente (lógica de verify_gap_plans)
Incrementar iteration_count

**Se iteration_count >= 3:**

Exibir: `Máximo de iterações atingido. Restam {N} problemas.`

Oferecer opções:
1. Forçar prosseguir (executar apesar dos problemas)
2. Fornecer orientação (usuário indica direção, tentar de novo)
3. Abandonar (sair; usuário executa /gsd-planejar-fase manualmente)

Aguardar resposta do usuário.
</step>

<step name="present_ready">
**Apresentar conclusão e próximos passos:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CORREÇÕES PRONTAS ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {X}: {Name}** — {N} lacuna(s) diagnosticada(s), {M} plano(s) de correção criado(s)

| Lacuna | Causa raiz | Plano de correção |
|--------|------------|-------------------|
| {truth 1} | {root_cause} | {phase}-04 |
| {truth 2} | {root_cause} | {phase}-04 |

Planos verificados e prontos para execução.

───────────────────────────────────────────────────────────────

## ▶ Próximo passo

**Executar correções** — rodar planos de correção

`/clear` depois `/gsd-executar-fase {phase} --gaps-only`

───────────────────────────────────────────────────────────────
```
</step>

</process>

<update_rules>
**Gravações em lote para eficiência:**

Manter resultados em memória. Gravar no arquivo somente quando:
1. **Problema encontrado** — Preservar o problema imediatamente
2. **Sessão concluída** — Gravação final antes do commit
3. **Checkpoint** — A cada 5 testes aprovados (rede de segurança)

| Seção | Regra | Quando gravar |
|---------|------|--------------|
| Frontmatter.status | SOBRESCREVER | Início, conclusão |
| Frontmatter.updated | SOBRESCREVER | Em qualquer gravação no arquivo |
| Current Test | SOBRESCREVER | Em qualquer gravação no arquivo |
| Tests.{N}.result | SOBRESCREVER | Em qualquer gravação no arquivo |
| Summary | SOBRESCREVER | Em qualquer gravação no arquivo |
| Gaps | ACRESCENTAR | Quando houver problema |

Em reset de contexto: O arquivo mostra o último checkpoint. Retomar dali.
</update_rules>

<severity_inference>
**Inferir gravidade a partir da linguagem natural do usuário:**

| O usuário diz (PT ou EN) | Inferir |
|-----------|-------|
| "trava", "erro", "exceção", "falha completamente" / "crashes", "error", "exception", "fails completely" | blocker |
| "não funciona", "nada acontece", "comportamento errado" / "doesn't work", "nothing happens", "wrong behavior" | major |
| "funciona mas...", "lento", "estranho", "problema pequeno" / "works but...", "slow", "weird", "minor issue" | minor |
| "cor", "espaçamento", "alinhamento", "está estranho visualmente" / "color", "spacing", "alignment", "looks off" | cosmetic |

Padrão **major** se não estiver claro. O usuário pode corrigir depois se precisar.

**Nunca perguntar "quão grave é isso?"** — apenas inferir e seguir.
</severity_inference>

<success_criteria>
- [ ] Arquivo UAT criado com todos os testes do SUMMARY.md
- [ ] Testes apresentados um de cada vez com comportamento esperado
- [ ] Respostas do usuário processadas como aprovado/problema/pular
- [ ] Gravidade inferida a partir da descrição (nunca perguntada)
- [ ] Gravações em lote: em problema, a cada 5 aprovações, ou ao concluir
- [ ] Commit ao concluir
- [ ] Se houver problemas: agentes de depuração em paralelo diagnosticam causas raiz
- [ ] Se houver problemas: gsd-planejador cria planos de correção (modo gap_closure)
- [ ] Se houver problemas: gsd-verificador-plano verifica os planos de correção
- [ ] Se houver problemas: loop de revisão até os planos passarem (máx. 3 iterações)
- [ ] Pronto para `/gsd-executar-fase --gaps-only` quando concluir
</success_criteria>
