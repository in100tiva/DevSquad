<purpose>
Verificar se o marco atingiu sua definição de pronto agregando verificações de fase, checando integração entre fases e avaliando cobertura de requisitos. Lê arquivos VERIFICATION.md existentes (fases já verificadas durante execute-phase), agrega dívida técnica e lacunas adiadas, e então invoca o verificador de integração para fiação entre fases.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 0. Inicializar Contexto do Marco

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init milestone-op)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `milestone_version`, `milestone_name`, `phase_count`, `completed_phases`, `commit_docs`.

Resolver modelo do verificador de integração:
```bash
integration_checker_model=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-integration-checker --raw)
```

## 1. Determinar Escopo do Marco

```bash
# Obter fases no marco (ordenadas numericamente, trata decimais)
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phases list
```

- Analisar versão dos argumentos ou detectar atual do ROADMAP.md
- Identificar todos os diretórios de fase no escopo
- Extrair definição de pronto do marco do ROADMAP.md
- Extrair requisitos mapeados para este marco do REQUIREMENTS.md

## 2. Ler Todas as Verificações de Fase

Para cada diretório de fase, ler o VERIFICATION.md:

```bash
# Para cada fase, usar find-phase para resolver o diretório (trata fases arquivadas)
PHASE_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" find-phase 01 --raw)
# Extrair diretório do JSON, então ler VERIFICATION.md daquele diretório
# Repetir para cada número de fase do ROADMAP.md
```

De cada VERIFICATION.md, extrair:
- **Status:** passed | gaps_found
- **Lacunas críticas:** (se houver — são bloqueadores)
- **Lacunas não-críticas:** dívida técnica, itens adiados, avisos
- **Anti-padrões encontrados:** TODOs, stubs, placeholders
- **Cobertura de requisitos:** quais requisitos satisfeitos/bloqueados

Se uma fase estiver sem VERIFICATION.md, sinalizá-la como "fase não verificada" — isto é um bloqueador.

## 3. Invocar Verificador de Integração

Com o contexto das fases coletado:

Extrair `MILESTONE_REQ_IDS` da tabela de rastreabilidade do REQUIREMENTS.md — todos os REQ-IDs atribuídos a fases neste marco.

```
Task(
  prompt="Verificar integração entre fases e fluxos E2E.

Fases: {phase_dirs}
Exportações das fases: {dos SUMMARYs}
Rotas de API: {rotas criadas}

Requisitos do Marco:
{MILESTONE_REQ_IDS — listar cada REQ-ID com descrição e fase atribuída}

DEVE mapear cada descoberta de integração aos IDs de requisitos afetados onde aplicável.

Verificar fiação entre fases e fluxos E2E do usuário.",
  subagent_type="gsd-verificador-integracao",
  model="{integration_checker_model}"
)
```

## 4. Coletar Resultados

Combinar:
- Lacunas e dívida técnica a nível de fase (do passo 2)
- Relatório do verificador de integração (lacunas de fiação, fluxos quebrados)

## 5. Verificar Cobertura de Requisitos (Referência Cruzada de 3 Fontes)

DEVE cruzar referências de três fontes independentes para cada requisito:

### 5a. Analisar Tabela de Rastreabilidade do REQUIREMENTS.md

Extrair todos os REQ-IDs mapeados para fases do marco da tabela de rastreabilidade:
- ID do requisito, descrição, fase atribuída, status atual, estado de verificação (`[x]` vs `[ ]`)

### 5b. Analisar Tabelas de Requisitos dos VERIFICATION.md das Fases

Para cada VERIFICATION.md de fase, extrair a tabela expandida de requisitos:
- Requisito | Plano de Origem | Descrição | Status | Evidência
- Mapear cada entrada de volta ao seu REQ-ID

### 5c. Extrair Verificação Cruzada do Frontmatter do SUMMARY.md

Para cada SUMMARY.md de fase, extrair `requirements-completed` do frontmatter YAML:
```bash
for summary in .planning/phases/*-*/*-SUMMARY.md; do
  node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" summary-extract "$summary" --fields requirements_completed --pick requirements_completed
done
```

### 5d. Matriz de Determinação de Status

Para cada REQ-ID, determinar status usando todas as três fontes:

| Status VERIFICATION.md | Frontmatter SUMMARY | REQUIREMENTS.md | → Status Final |
|------------------------|---------------------|-----------------|----------------|
| passed                 | listado             | `[x]`           | **satisfeito** |
| passed                 | listado             | `[ ]`           | **satisfeito** (atualizar checkbox) |
| passed                 | ausente             | qualquer        | **parcial** (verificar manualmente) |
| gaps_found             | qualquer            | qualquer        | **não satisfeito** |
| ausente                | listado             | qualquer        | **parcial** (lacuna de verificação) |
| ausente                | ausente             | qualquer        | **não satisfeito** |

### 5e. Gate de FALHA e Detecção de Órfãos

**OBRIGATÓRIO:** Qualquer requisito `não satisfeito` DEVE forçar status `gaps_found` na auditoria do marco.

**Detecção de órfãos:** Requisitos presentes na tabela de rastreabilidade do REQUIREMENTS.md mas ausentes de TODOS os arquivos VERIFICATION.md das fases DEVEM ser sinalizados como órfãos. Requisitos órfãos são tratados como `não satisfeito` — foram atribuídos mas nunca verificados por nenhuma fase.

## 5.5. Descoberta de Conformidade Nyquist

Pular se `workflow.nyquist_validation` for explicitamente `false` (ausente = habilitado).

```bash
NYQUIST_CONFIG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.nyquist_validation --raw 2>/dev/null)
```

Se `false`: pular inteiramente.

Para cada diretório de fase, verificar `*-VALIDATION.md`. Se existir, analisar frontmatter (`nyquist_compliant`, `wave_0_complete`).

Classificar por fase:

| Status | Condição |
|--------|----------|
| CONFORME | `nyquist_compliant: true` e todas as tarefas verdes |
| PARCIAL | VALIDATION.md existe, `nyquist_compliant: false` ou vermelho/pendente |
| AUSENTE | Sem VALIDATION.md |

Adicionar ao YAML da auditoria: `nyquist: { compliant_phases, partial_phases, missing_phases, overall }`

Apenas descoberta — nunca auto-chama `/gsd-validate-phase`.

## 6. Agregar em v{version}-MILESTONE-AUDIT.md

Criar `.planning/v{version}-v{version}-MILESTONE-AUDIT.md` com:

```yaml
---
milestone: {version}
audited: {timestamp}
status: passed | gaps_found | tech_debt
scores:
  requirements: N/M
  phases: N/M
  integration: N/M
  flows: N/M
gaps:  # Bloqueadores críticos
  requirements:
    - id: "{REQ-ID}"
      status: "unsatisfied | partial | orphaned"
      phase: "{fase atribuída}"
      claimed_by_plans: ["{arquivos de plano que referenciam este requisito}"]
      completed_by_plans: ["{arquivos de plano cujo SUMMARY marca como completo}"]
      verification_status: "passed | gaps_found | missing | orphaned"
      evidence: "{evidência específica ou ausência dela}"
  integration: [...]
  flows: [...]
tech_debt:  # Não-críticos, adiados
  - phase: 01-auth
    items:
      - "TODO: adicionar rate limiting"
      - "Aviso: sem validação de força de senha"
  - phase: 03-dashboard
    items:
      - "Adiado: layout responsivo mobile"
---
```

Mais relatório completo em markdown com tabelas para requisitos, fases, integração, dívida técnica.

**Valores de status:**
- `passed` — todos os requisitos atendidos, sem lacunas críticas, dívida técnica mínima
- `gaps_found` — bloqueadores críticos existem
- `tech_debt` — sem bloqueadores mas itens adiados acumulados precisam de revisão

## 7. Apresentar Resultados

Rotear por status (ver `<offer_next>`).

</process>

<offer_next>
Exibir este markdown diretamente (não como bloco de código). Rotear baseado no status:

---

**Se passed:**

## ✓ Marco {version} — Auditoria Aprovada

**Pontuação:** {N}/{M} requisitos satisfeitos
**Relatório:** .planning/v{version}-MILESTONE-AUDIT.md

Todos os requisitos cobertos. Integração entre fases verificada. Fluxos E2E completos.

───────────────────────────────────────────────────────────────

## ▶ Próximo Passo

**Completar marco** — arquivar e etiquetar

/gsd-complete-milestone {version}

<sub>/clear primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────

---

**Se gaps_found:**

## ⚠ Marco {version} — Lacunas Encontradas

**Pontuação:** {N}/{M} requisitos satisfeitos
**Relatório:** .planning/v{version}-MILESTONE-AUDIT.md

### Requisitos Não Satisfeitos

{Para cada requisito não satisfeito:}
- **{REQ-ID}: {descrição}** (Fase {X})
  - {razão}

### Problemas Entre Fases

{Para cada lacuna de integração:}
- **{de} → {para}:** {problema}

### Fluxos Quebrados

{Para cada lacuna de fluxo:}
- **{nome do fluxo}:** quebra em {etapa}

### Cobertura Nyquist

|| Fase | VALIDATION.md | Conforme | Ação |
||------|---------------|----------|------|
|| {fase} | existe/ausente | true/false/parcial | `/gsd-validate-phase {N}` |

Fases que precisam de validação: executar `/gsd-validate-phase {N}` para cada fase sinalizada.

───────────────────────────────────────────────────────────────

## ▶ Próximo Passo

**Planejar fechamento de lacunas** — criar fases para completar o marco

/gsd-plan-milestone-gaps

<sub>/clear primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────

**Também disponível:**
- cat .planning/v{version}-MILESTONE-AUDIT.md — ver relatório completo
- /gsd-complete-milestone {version} — prosseguir mesmo assim (aceitar dívida técnica)

───────────────────────────────────────────────────────────────

---

**Se tech_debt (sem bloqueadores mas dívida acumulada):**

## ⚡ Marco {version} — Revisão de Dívida Técnica

**Pontuação:** {N}/{M} requisitos satisfeitos
**Relatório:** .planning/v{version}-MILESTONE-AUDIT.md

Todos os requisitos atendidos. Sem bloqueadores críticos. Dívida técnica acumulada precisa de revisão.

### Dívida Técnica por Fase

{Para cada fase com dívida:}
**Fase {X}: {nome}**
- {item 1}
- {item 2}

### Total: {N} itens em {M} fases

───────────────────────────────────────────────────────────────

## ▶ Opções

**A. Completar marco** — aceitar dívida, rastrear no backlog

/gsd-complete-milestone {version}

**B. Planejar fase de limpeza** — tratar dívida antes de completar

/gsd-plan-milestone-gaps

<sub>/clear primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────
</offer_next>

<success_criteria>
- [ ] Escopo do marco identificado
- [ ] Todos os arquivos VERIFICATION.md das fases lidos
- [ ] Frontmatter `requirements-completed` do SUMMARY.md extraído para cada fase
- [ ] Tabela de rastreabilidade do REQUIREMENTS.md analisada para todos os REQ-IDs do marco
- [ ] Referência cruzada de 3 fontes completada (VERIFICATION + SUMMARY + rastreabilidade)
- [ ] Requisitos órfãos detectados (na rastreabilidade mas ausentes de todos os VERIFICATIONs)
- [ ] Dívida técnica e lacunas adiadas agregadas
- [ ] Verificador de integração invocado com IDs de requisitos do marco
- [ ] v{version}-MILESTONE-AUDIT.md criado com objetos estruturados de lacunas de requisitos
- [ ] Gate de FALHA aplicado — qualquer requisito não satisfeito força status gaps_found
- [ ] Conformidade Nyquist verificada para todas as fases do marco (se habilitado)
- [ ] Fases com VALIDATION.md ausente sinalizadas com sugestão de validate-phase
- [ ] Resultados apresentados com próximos passos acionáveis
</success_criteria>
