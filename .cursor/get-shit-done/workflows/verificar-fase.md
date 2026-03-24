<purpose>
Verificar alcance do objetivo da fase através de análise objetivo-reversa. Verificar que o codebase entrega o que a fase prometeu, não apenas que as tarefas foram completadas.

Executado por um subagente de verificação invocado pelo executar-fase.md.
</purpose>

<core_principle>
**Conclusão de tarefa ≠ Alcance do objetivo**

Uma tarefa "criar componente de chat" pode ser marcada como completa quando o componente é um placeholder. A tarefa foi feita — mas o objetivo "interface de chat funcionando" não foi alcançado.

Verificação objetivo-reversa:
1. O que deve ser VERDADE para o objetivo ser alcançado?
2. O que deve EXISTIR para essas verdades se manterem?
3. O que deve estar CONECTADO para esses artefatos funcionarem?

Depois verificar cada nível contra o codebase real.
</core_principle>

<required_reading>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/padroes-verificacao.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/relatorio-verificacao.md
</required_reading>

<process>

<step name="load_context" priority="first">
Carregar contexto de operação da fase:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `phase_dir`, `phase_number`, `phase_name`, `has_plans`, `plan_count`.

Depois carregar detalhes da fase e listar planos/resumos:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}"
grep -E "^| ${phase_number}" .planning/REQUIREMENTS.md 2>/dev/null
ls "$phase_dir"/*-SUMMARY.md "$phase_dir"/*-PLAN.md 2>/dev/null
```

Extrair **objetivo da fase** do ROADMAP.md (o resultado a verificar, não tarefas) e **requisitos** do REQUIREMENTS.md se existir.
</step>

<step name="establish_must_haves">
**Opção A: Must-haves no frontmatter do PLAN**

Usar gsd-tools para extrair must_haves de cada PLAN:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  MUST_HAVES=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" frontmatter get "$plan" --field must_haves)
  echo "=== $plan ===" && echo "$MUST_HAVES"
done
```

Retorna JSON: `{ truths: [...], artifacts: [...], key_links: [...] }`

Agregar todos must_haves dos planos para verificação em nível de fase.

**Opção B: Usar Critérios de Sucesso do ROADMAP.md**

Se não houver must_haves no frontmatter (MUST_HAVES retorna erro ou vazio), verificar Critérios de Sucesso:

```bash
PHASE_DATA=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}" --raw)
```

Analisar o array `success_criteria` da saída JSON. Se não vazio:
1. Usar cada Critério de Sucesso diretamente como uma **verdade** (já estão escritos como comportamentos observáveis e testáveis)
2. Derivar **artefatos** (caminhos concretos de arquivo para cada verdade)
3. Derivar **links-chave** (conexões críticas onde stubs se escondem)
4. Documentar os must-haves antes de prosseguir

Critérios de Sucesso do ROADMAP.md são o contrato — sobrescrevem must_haves em nível de PLAN quando ambos existem.

**Opção C: Derivar do objetivo da fase (fallback)**

Se não houver must_haves no frontmatter E sem Critérios de Sucesso no ROADMAP:
1. Declarar o objetivo do ROADMAP.md
2. Derivar **verdades** (3-7 comportamentos observáveis, cada um testável)
3. Derivar **artefatos** (caminhos concretos de arquivo para cada verdade)
4. Derivar **links-chave** (conexões críticas onde stubs se escondem)
5. Documentar must-haves derivados antes de prosseguir
</step>

<step name="verify_truths">
Para cada verdade observável, determinar se o codebase a habilita.

**Status:** ✓ VERIFICADO (todos artefatos de suporte passam) | ✗ FALHOU (artefato ausente/stub/desconectado) | ? INCERTO (precisa humano)

Para cada verdade: identificar artefatos de suporte → verificar status do artefato → verificar conexão → determinar status da verdade.

**Exemplo:** Verdade "Usuário pode ver mensagens existentes" depende de Chat.tsx (renderiza), /api/chat GET (fornece), modelo Message (schema). Se Chat.tsx é stub ou API retorna [] hardcoded → FALHOU. Se todos existem, são substantivos e conectados → VERIFICADO.
</step>

<step name="verify_artifacts">
Usar gsd-tools para verificação de artefatos contra must_haves em cada PLAN:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  ARTIFACT_RESULT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" verify artifacts "$plan")
  echo "=== $plan ===" && echo "$ARTIFACT_RESULT"
done
```

Analisar resultado JSON: `{ all_passed, passed, total, artifacts: [{path, exists, issues, passed}] }`

**Status do artefato a partir do resultado:**
- `exists=false` → AUSENTE
- `issues` não vazio → STUB (verificar issues por "Only N lines" ou "Missing pattern")
- `passed=true` → VERIFICADO (Níveis 1-2 passam)

**Nível 3 — Conectado (verificação manual para artefatos que passam Níveis 1-2):**
```bash
grep -r "import.*$artifact_name" src/ --include="*.ts" --include="*.tsx"  # IMPORTADO
grep -r "$artifact_name" src/ --include="*.ts" --include="*.tsx" | grep -v "import"  # USADO
```
CONECTADO = importado E usado. ÓRFÃO = existe mas não importado/usado.

| Existe | Substantivo | Conectado | Status |
|--------|-------------|-----------|--------|
| ✓ | ✓ | ✓ | ✓ VERIFICADO |
| ✓ | ✓ | ✗ | ⚠️ ÓRFÃO |
| ✓ | ✗ | - | ✗ STUB |
| ✗ | - | - | ✗ AUSENTE |

**Verificação pontual em nível de export (severidade WARNING):**

Para artefatos que passam Nível 3, verificar pontualmente exports individuais:
- Extrair símbolos exportados-chave (funções, constantes, classes — pular tipos/interfaces)
- Para cada, buscar por uso fora do arquivo definidor
- Sinalizar exports sem nenhum ponto de chamada externo como "exportado mas não usado"

Isso captura dead stores como `setPlan()` que existem em um arquivo conectado mas nunca são realmente chamados. Reportar como WARNING — pode indicar conexão cross-plan incompleta ou código remanescente de revisões de plano.
</step>

<step name="verify_wiring">
Usar gsd-tools para verificação de links-chave contra must_haves em cada PLAN:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  LINKS_RESULT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" verify key-links "$plan")
  echo "=== $plan ===" && echo "$LINKS_RESULT"
done
```

Analisar resultado JSON: `{ all_verified, verified, total, links: [{from, to, via, verified, detail}] }`

**Status do link a partir do resultado:**
- `verified=true` → CONECTADO
- `verified=false` com "not found" → NÃO_CONECTADO
- `verified=false` com "Pattern not found" → PARCIAL

**Padrões de fallback (se key_links não estiver em must_haves):**

| Padrão | Verificação | Status |
|--------|-------------|--------|
| Componente → API | chamada fetch/axios para caminho da API, resposta usada (await/.then/setState) | CONECTADO / PARCIAL (chamada mas resposta não usada) / NÃO_CONECTADO |
| API → Banco de Dados | query Prisma/DB no modelo, resultado retornado via res.json() | CONECTADO / PARCIAL (query mas não retornado) / NÃO_CONECTADO |
| Form → Handler | onSubmit com implementação real (fetch/axios/mutate/dispatch), não console.log/vazio | CONECTADO / STUB (apenas log/vazio) / NÃO_CONECTADO |
| Estado → Renderização | variável useState aparece no JSX (`{stateVar}` ou `{stateVar.property}`) | CONECTADO / NÃO_CONECTADO |

Registrar status e evidência para cada link-chave.
</step>

<step name="verify_requirements">
Se REQUIREMENTS.md existir:
```bash
grep -E "Phase ${PHASE_NUM}" .planning/REQUIREMENTS.md 2>/dev/null
```

Para cada requisito: analisar descrição → identificar verdades/artefatos de suporte → status: ✓ SATISFEITO / ✗ BLOQUEADO / ? PRECISA HUMANO.
</step>

<step name="scan_antipatterns">
Extrair arquivos modificados nesta fase do SUMMARY.md, varrer cada um:

| Padrão | Busca | Severidade |
|--------|-------|------------|
| TODO/FIXME/XXX/HACK | `grep -n -E "TODO\|FIXME\|XXX\|HACK"` | ⚠️ Aviso |
| Conteúdo placeholder | `grep -n -iE "placeholder\|coming soon\|will be here"` | 🛑 Bloqueador |
| Retornos vazios | `grep -n -E "return null\|return \{\}\|return \[\]\|=> \{\}"` | ⚠️ Aviso |
| Funções apenas com log | Funções contendo apenas console.log | ⚠️ Aviso |

Categorizar: 🛑 Bloqueador (impede objetivo) | ⚠️ Aviso (incompleto) | ℹ️ Info (notável).
</step>

<step name="identify_human_verification">
**Sempre precisa humano:** Aparência visual, conclusão de fluxo do usuário, comportamento em tempo real (WebSocket/SSE), integração com serviço externo, sensação de performance, clareza de mensagens de erro.

**Precisa humano se incerto:** Conexão complexa que grep não consegue rastrear, comportamento dependente de estado dinâmico, casos extremos.

Formatar cada como: Nome do Teste → O que fazer → Resultado esperado → Por que não pode verificar programaticamente.
</step>

<step name="determine_status">
**passed:** Todas verdades VERIFICADAS, todos artefatos passam níveis 1-3, todos links-chave CONECTADOS, sem anti-padrões bloqueadores.

**gaps_found:** Qualquer verdade FALHOU, artefato AUSENTE/STUB, link-chave NÃO_CONECTADO, ou bloqueador encontrado.

**human_needed:** Todas verificações automatizadas passam mas itens de verificação humana permanecem.

**Pontuação:** `verdades_verificadas / total_verdades`
</step>

<step name="generate_fix_plans">
Se gaps_found:

1. **Agrupar lacunas relacionadas:** stub de API + componente desconectado → "Conectar frontend ao backend". Múltiplos ausentes → "Completar implementação principal". Apenas conexão → "Conectar componentes existentes".

2. **Gerar plano por agrupamento:** Objetivo, 2-3 tarefas (arquivos/ação/verificar cada), etapa de re-verificação. Manter focado: preocupação única por plano.

3. **Ordenar por dependência:** Corrigir ausentes → corrigir stubs → corrigir conexões → verificar.
</step>

<step name="create_report">
```bash
REPORT_PATH="$PHASE_DIR/${PHASE_NUM}-VERIFICATION.md"
```

Preencher seções do template: frontmatter (fase/timestamp/status/pontuação), alcance do objetivo, tabela de artefatos, tabela de conexões, cobertura de requisitos, anti-padrões, verificação humana, resumo de lacunas, planos de correção (se gaps_found), metadados.

Ver D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/relatorio-verificacao.md para template completo.
</step>

<step name="return_to_orchestrator">
Retornar status (`passed` | `gaps_found` | `human_needed`), pontuação (N/M must-haves), caminho do relatório.

Se gaps_found: listar lacunas + nomes de planos de correção recomendados.
Se human_needed: listar itens que requerem teste humano.

Orquestrador roteia: `passed` → update_roadmap | `gaps_found` → criar/executar correções, re-verificar | `human_needed` → apresentar ao usuário.
</step>

</process>

<success_criteria>
- [ ] Must-haves estabelecidos (do frontmatter ou derivados)
- [ ] Todas verdades verificadas com status e evidência
- [ ] Todos artefatos verificados em todos os três níveis
- [ ] Todos links-chave verificados
- [ ] Cobertura de requisitos avaliada (se aplicável)
- [ ] Anti-padrões varridos e categorizados
- [ ] Itens de verificação humana identificados
- [ ] Status geral determinado
- [ ] Planos de correção gerados (se gaps_found)
- [ ] VERIFICATION.md criado com relatório completo
- [ ] Resultados retornados ao orquestrador
</success_criteria>
</output>
