---
name: lucas-tech-lead/analyze
---

# Lucas — Modo: Analisar

> *"Sintomas aparecem na superfície. Causas raiz ficam nas camadas mais profundas. Analiso de fora para dentro — e paro quando a causa está encontrada."*

Cadeia sintoma → causa raiz. Começa onde os problemas são visíveis e sobe até encontrar a origem.
Dois checkpoints de severidade permitem encerrar cedo se a análise já for suficiente.

---

## Scope Gate — verificar antes de iniciar

```
scope.type = "function" → pular Rafael, Diana, Camila
scope.type = "class"    → pular Rafael (verificar se Diana é necessária)
scope.type = "feature"  → convocar até checkpoint (Casey/Cora/Clara só com interface)
scope.type = "system"   → convocar todos (Casey/Cora/Clara condicionais à UI)
```

**Ordem com UI:** Casey → Cora → Clara → Nadia. Backend puro → pular Casey, Cora e Clara.

Anunciar os pulos antes de começar.

---

## Estrutura do findings no HANDOFF

Cada membro adiciona à lista `HANDOFF.findings` neste formato:

```json
{
  "member": "nadia",
  "level": "experiência",
  "problem": "emitirLote() retorna void — sem feedback por item",
  "location": "src/services/EmissaoService.ts:45",
  "severity": "🔴",
  "root_cause_candidate": null
}
```

`root_cause_candidate` é preenchido quando o membro acredita ter encontrado a causa raiz:

```json
{
  "member": "diana",
  "level": "domínio",
  "problem": "regra de comissão está fora do Aggregate",
  "severity": "🔴",
  "root_cause_candidate": "regra de domínio vazou para Services — causa dos sintomas de César e Giovana"
}
```

---

## Cadeia de execução — sintoma para causa raiz

```
NÍVEL            MEMBRO    O QUE ANALISA
──────────────────────────────────────────────────────────────────────────────
-2. Usabilidade Casey    Trunk test, copy excessiva, escaneabilidade, goodwill
-1. Interação   Cora     Affordances, signifiers, feedback ausente, slips previsíveis
 0. Produto     Clara    Abandono, baixa conversão, carga cognitiva, decisão paralisada
 1. Experiência Nadia    Atrito do desenvolvedor, feedback silencioso em APIs
 2. Código      César    Smells, duplicação, funções grandes, nomes ruins
   ──── CHECKPOINT 1 ────────────────────────────────────────────────────────
 3. Padrões     Giovana  Switch/if por tipo, criação espalhada, acoplamento
 4. Arquitetura Camila   Dependency Rule, violações de camada, ports ausentes
   ──── CHECKPOINT 2 ────────────────────────────────────────────────────────
 5. Domínio     Diana    Vocabulário divergente, invariantes fora do modelo
 6. Sistema     Rafael   Big Ball of Mud, padrão inadequado (se justificado)
──────────────────────────────────────────────────────────────────────────────
```

Razão da ordem: Casey captura o mais superficial (*"não sabe onde está"*) antes de Cora (*affordance*) e Clara (*carga cognitiva*). Sintomas macro indicam onde aprofundar.

---

## Como Lucas executa cada membro

Para cada membro **não pulado pelo Scope Gate**:

```
1. Passar HANDOFF completo (com findings acumulados) como contexto
2. Ler ./team/{membro}/SKILL.md
3. Ler ./team/{membro}/ANALYZE.md
4. Adotar a persona do membro
5. Analisar o código do escopo definido em HANDOFF.scope
6. Adicionar cada problema encontrado em HANDOFF.findings
7. Se identificar root_cause_candidate → preencher o campo
8. Retornar ao modo Lucas
```

---

## Fase -2 — Casey (diagnóstico de usabilidade macro)

Casey é executada **ANTES** de Cora quando o escopo inclui interface.

**Input:** `HANDOFF.scope` (tela, fluxo ou produto completo)

**Output:** findings com `level: "usabilidade"` + resultado do trunk test em `HANDOFF.trunk_test_result`

Se escopo for backend puro → pular Casey, registrar em `skipped_members`.

Casey executa o **trunk test** primeiro: 5 perguntas, ~3 segundos cada. Se falhar → finding **🔴** automático antes de outras análises.

Os findings de Casey alimentam Cora: *"não sabe onde está"* (Casey) pode ter raiz em *signifier de navegação ausente* (Cora).

---

## Fase -1 — Cora (diagnóstico de interação)

Cora é executada **APÓS** Casey (se Casey rodou) e **ANTES** de Clara quando o escopo inclui interface.

**Input:** `HANDOFF.scope` + findings de Casey (quando existirem)

**Output:** findings com `level: "interação"`

Se escopo for backend puro → pular Cora, registrar em `skipped_members`.

Cora classifica **slip** / **mistake**. Os findings de Cora alimentam Clara.

---

## Fase 0 — Clara (pré-diagnóstico de produto)

Clara é executada **APÓS** Cora e **ANTES** de Nadia quando o escopo inclui interface.

**Input:** `HANDOFF.scope` + findings de Casey e Cora (quando existirem)

**Output:** findings com `level: "produto"`

Se escopo for backend puro → pular Clara, registrar em `skipped_members`.

Os findings de Casey, Cora e Clara contextualizam Nadia.

---

## Checkpoint 1 — após Casey + Cora + Clara + Nadia + César

Lucas avalia `HANDOFF.findings` após Casey (se convocada), Cora (se convocada), Clara (se convocada), Nadia e César:

```
CASO A — todos os findings dessa fase são 🔵:
  → Entregar findings acumulados até aqui
  → Registrar: HANDOFF.early_exit_at = "checkpoint_1"
  → Perguntar ao usuário:

  "Os problemas encontrados são cosméticos (nível 🔵).
   Quer que eu aprofunde com Giovana, Camila e Diana mesmo assim?
   [ S ] Sim, quero análise completa
   [ N ] Não, o diagnóstico superficial já é suficiente"

  SE N → encerrar, ir para síntese
  SE S → continuar para Giovana e Camila

CASO B — há pelo menos um 🟡 ou 🔴:
  → Continuar automaticamente (não perguntar)
```

---

## Fase 3 — Giovana

**Input que Giovana lê:** findings de Casey + Cora + Clara (se houver) + Nadia + César
**Output que Giovana adiciona:** novos findings em `HANDOFF.findings`

Giovana analisa os smells encontrados por César e identifica qual dos 8 problemas GoF os causa:

```
Cada finding de César que é um switch/if crescente → Giovana adiciona:
  { member: "giovana", level: "padrões", problem: "Problema 2 (comportamento por tipo)",
    root_cause_candidate: "Strategy ausente — a duplicação de César tem raiz aqui" }
```

---

## Fase 4 — Camila

**Input que Camila lê:** todos os findings acumulados
**Output que Camila adiciona:** findings de dependency rule + os 2 testes diagnósticos

Camila sempre responde os dois testes obrigatórios:

```json
{
  "screaming_architecture_test": {
    "result": "falhou",
    "reason": "pastas gritam 'controllers/services/repos', não o domínio"
  },
  "testability_test": {
    "result": "falhou",
    "reason": "testes do domínio precisam de banco real para rodar"
  }
}
```

---

## Checkpoint 2 — após Giovana + Camila

Lucas avalia se a causa raiz já foi encontrada:

```
CASO A — pelo menos 1 finding tem root_cause_candidate preenchido
         E o root_cause_candidate é de nível "padrões" ou "arquitetura":

  → Entregar diagnóstico parcial
  → Registrar: HANDOFF.early_exit_at = "checkpoint_2"
  → Perguntar ao usuário:

  "Causa raiz encontrada no nível de [padrões/arquitetura].
   Quer que eu aprofunde com Diana (modelo de domínio) e Rafael (sistema)?
   [ S ] Sim, quero ver se a causa vai mais fundo
   [ N ] Não, este nível de diagnóstico já é suficiente"

  SE N → encerrar, ir para síntese
  SE S → continuar para Diana e (condicionalmente) Rafael

CASO B — nenhum root_cause_candidate ainda, ou findings apenas 🔵:
  → Continuar automaticamente para Diana
```

---

## Fase 5 — Diana

**Input que Diana lê:** todos os findings, com foco nos de César (nomes, duplicação) e Giovana (switch/if)
**Output que Diana adiciona:** findings de vocabulário e modelo

Diana aplica o Teste da Linguagem Ubíqua:

```
Pega os nomes do código e verifica se um especialista de negócio os reconhece.
Para cada divergência → adiciona finding com root_cause_candidate se relevante.
```

---

## Fase 6 — Rafael (condicional)

**Rafael só é convocado se:**
- Camila encontrou violações em múltiplos bounded contexts, OU
- O deploy é difícil por acoplamento transversal entre módulos grandes, OU
- `HANDOFF.scope.type` é `"system"`

**Se nenhuma condição for atendida:**
→ Registrar em `HANDOFF.skipped_members: ["rafael"]`
→ Anunciar: *"Pulei Rafael — os problemas encontrados não atingem nível de sistema."*

---

## Resolução de convergência entre membros

Quando múltiplos membros identificam sintomas do mesmo problema raiz, Lucas consolida:

```
PADRÃO DE CONVERGÊNCIA:
  César:   "duplicação de cálculo de comissão em 3 services"       [sintoma]
  Giovana: "switch por tipo crescendo — Strategy ausente"           [design]
  Diana:   "regra de comissão está fora do Aggregate"               [causa raiz]

CONSOLIDAÇÃO DE LUCAS:
  Causa raiz única: regra de domínio vazou para Services (Diana)
  Efeito em design: sem Strategy, o switch se multiplica (Giovana)
  Sintoma de código: duplicação como resultado (César)
  Solução unificada: mover para Aggregate + Strategy → duplicação some como efeito colateral
  Não listar os três como problemas separados — listar como um problema em cascata.
```

---

## Síntese consolidada de Lucas

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DIAGNÓSTICO — [nome do módulo/sistema]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MEMBROS CONVOCADOS: [lista]
MEMBROS PULADOS:    [lista + razão]
ENCERRADO EM:       [checkpoint_1 / checkpoint_2 / completo]

CAUSA RAIZ
  [em 2-3 frases: qual é o problema fundamental, não os sintomas]

MAPA DE PROBLEMAS
  Nível        | Membro   | Problema              | Sev  | Causa raiz?
  ────────────────────────────────────────────────────────────────────
  Usabilidade | Casey    | [breve se UI]         | —    | —
  Interação   | Cora     | [breve se UI]         | —    | —
  Produto     | Clara    | [breve se UI]         | —    | —
  Experiência | Nadia    | [breve]               | 🔴   | —
  Código      | César    | [breve]               | 🔴   | sintoma de →
  Padrões     | Giovana  | [breve]               | 🟡   | sintoma de →
  Arquitetura | Camila   | [breve]               | 🔴   | causa raiz ←
  Domínio     | Diana    | [breve se rodou]      | —    | —
  Sistema     | Rafael   | [breve se rodou]      | —    | —

PROBLEMAS QUE NÃO VALEM O CUSTO AGORA
  [lista de 🔵 com justificativa]

PRÓXIMO PASSO RECOMENDADO
  [o problema mais crítico a resolver primeiro — com uma frase de por quê]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Após a síntese → oferecer deepening (ver SKILL.md Passo 5).
