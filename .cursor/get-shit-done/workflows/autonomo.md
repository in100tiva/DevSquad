<purpose>

Executar todas as fases restantes do marco autonomamente. Para cada fase incompleta: discutir → planejar → executar usando invocações Skill() planas. Pausa apenas para decisões explícitas do usuário (aceitação de áreas cinzentas, bloqueios, solicitações de validação). Relê ROADMAP.md após cada fase para capturar fases inseridas dinamicamente.

</purpose>

<required_reading>

Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

</required_reading>

<process>

<step name="initialize" priority="first">

## 1. Inicializar

Analisar `{{GSD_ARGS}}` para flag `--from N`:

```bash
FROM_PHASE=""
if echo "{{GSD_ARGS}}" | grep -qE '\-\-from\s+[0-9]'; then
  FROM_PHASE=$(echo "{{GSD_ARGS}}" | grep -oE '\-\-from\s+[0-9]+\.?[0-9]*' | awk '{print $2}')
fi
```

Bootstrap via init de nível de marco:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init milestone-op)
```

Analise o JSON para: `milestone_version`, `milestone_name`, `phase_count`, `completed_phases`, `roadmap_exists`, `state_exists`, `commit_docs`.

**Se `roadmap_exists` for false:** Erro — "Nenhum ROADMAP.md encontrado. Execute `/gsd-new-milestone` primeiro."
**Se `state_exists` for false:** Erro — "Nenhum STATE.md encontrado. Execute `/gsd-new-milestone` primeiro."

Exibir banner de início:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTÔNOMO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Marco: {milestone_version} — {milestone_name}
 Fases: {phase_count} total, {completed_phases} completas
```

Se `FROM_PHASE` estiver definido, exibir: `Iniciando a partir da fase ${FROM_PHASE}`

</step>

<step name="discover_phases">

## 2. Descobrir Fases

Executar descoberta de fases:

```bash
ROADMAP=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Analisar o array `phases` do JSON.

**Filtrar para fases incompletas:** Manter apenas fases onde `disk_status !== "complete"` OU `roadmap_complete === false`.

**Aplicar filtro `--from N`:** Se `FROM_PHASE` foi fornecido, filtrar adicionalmente fases onde `number < FROM_PHASE` (usar comparação numérica — trata fases decimais como "5.1").

**Ordenar por `number`** em ordem numérica ascendente.

**Se nenhuma fase incompleta restar:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTÔNOMO ▸ COMPLETO 🎉
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Todas as fases completas! Nada mais a fazer.
```

Sair de forma limpa.

**Exibir plano de fases:**

```
## Plano de Fases

| # | Fase | Status |
|---|------|--------|
| 5 | Scaffolding de Skills & Descoberta de Fases | Em Progresso |
| 6 | Discuss Inteligente | Não Iniciada |
| 7 | Refinamentos Auto-Chain | Não Iniciada |
| 8 | Orquestração de Ciclo de Vida | Não Iniciada |
```

**Buscar detalhes para cada fase:**

```bash
DETAIL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM})
```

Extrair `phase_name`, `goal`, `success_criteria` de cada. Armazenar para uso em execute_phase e mensagens de transição.

</step>

<step name="execute_phase">

## 3. Executar Fase

Para a fase atual, exibir o banner de progresso:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTÔNOMO ▸ Fase {N}/{T}: {Nome} [████░░░░] {P}%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Onde N = número da fase atual (do ROADMAP, ex: 6), T = total de fases do marco (de `phase_count` analisado no passo de inicialização, ex: 8), P = percentual de todas as fases do marco completadas até agora. Calcular P como: (número de fases com `disk_status` "complete" do último `roadmap analyze` / T × 100). Usar █ para preenchido e ░ para segmentos vazios na barra de progresso (8 caracteres de largura).

**3a. Discuss Inteligente**

Verificar se CONTEXT.md já existe para esta fase:

```bash
PHASE_STATE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Analisar `has_context` do JSON.

**Se has_context for true:** Pular discuss — contexto já coletado. Exibir:

```
Fase ${PHASE_NUM}: Contexto existe — pulando discuss.
```

Prosseguir para 3b.

**Se has_context for false:** Verificar se discuss está desabilitado via configurações:

```bash
SKIP_DISCUSS=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.skip_discuss 2>/dev/null || echo "false")
```

**Se SKIP_DISCUSS for `true`:** Pular discuss inteiramente — a descrição da fase no ROADMAP é a especificação. Exibir:

```
Fase ${PHASE_NUM}: Discuss pulado (workflow.skip_discuss=true) — usando objetivo da fase do ROADMAP como especificação.
```

Escrever um CONTEXT.md mínimo para que o plan-phase downstream tenha entrada válida. Obter detalhes da fase:

```bash
DETAIL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM})
```

Extrair `goal` e `requirements` do JSON. Escrever `${phase_dir}/${padded_phase}-CONTEXT.md` com:

```markdown
# Fase {PHASE_NUM}: {Nome da Fase} - Contexto

**Coletado:** {data}
**Status:** Pronto para planejamento
**Modo:** Auto-gerado (discuss pulado via workflow.skip_discuss)

<domain>
## Limite da Fase

{objetivo da descrição da fase do ROADMAP}

</domain>

<decisions>
## Decisões de Implementação

### Discrição do Claude
Todas as escolhas de implementação estão a critério do Claude — fase de discussão foi pulada por configuração do usuário. Use objetivo da fase do ROADMAP, critérios de sucesso e convenções do codebase para guiar decisões.

</decisions>

<code_context>
## Insights do Código Existente

Contexto do codebase será coletado durante a pesquisa do plan-phase.

</code_context>

<specifics>
## Ideias Específicas

Sem requisitos específicos — fase de discussão pulada. Consultar descrição da fase e critérios de sucesso do ROADMAP.

</specifics>

<deferred>
## Ideias Adiadas

Nenhuma — fase de discussão pulada.

</deferred>
```

Commitar o contexto mínimo:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${PADDED_PHASE}): auto-generated context (discuss skipped)" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Prosseguir para 3b.

**Se SKIP_DISCUSS for `false` (ou não definido):** Executar o passo smart_discuss para esta fase.

Após smart_discuss completar, verificar que o contexto foi escrito:

```bash
PHASE_STATE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Verificar `has_context`. Se false → ir para handle_blocker: "Smart discuss para fase ${PHASE_NUM} não produziu CONTEXT.md."

**3b. Planejar**

```
Skill(skill="gsd-plan-phase", args="${PHASE_NUM}")
```

Verificar que o plano produziu saída — re-executar `init phase-op` e verificar `has_plans`. Se false → ir para handle_blocker: "Plan phase ${PHASE_NUM} não produziu nenhum plano."

**3c. Executar**

```
Skill(skill="gsd-execute-phase", args="${PHASE_NUM} --no-transition")
```

**3d. Roteamento Pós-Execução**

Após execute-phase retornar, ler o resultado da verificação:

```bash
VERIFY_STATUS=$(grep "^status:" "${PHASE_DIR}"/*-VERIFICATION.md 2>/dev/null | head -1 | cut -d: -f2 | tr -d ' ')
```

Onde `PHASE_DIR` vem da chamada `init phase-op` já feita no passo 3a. Se a variável não estiver no escopo, buscar novamente:

```bash
PHASE_STATE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Analisar `phase_dir` do JSON.

**Se VERIFY_STATUS estiver vazio** (sem VERIFICATION.md ou sem campo status):

Ir para handle_blocker: "Execute phase ${PHASE_NUM} não produziu resultados de verificação."

**Se `passed`:**

Exibir:
```
Fase ${PHASE_NUM} ✅ ${PHASE_NAME} — Verificação aprovada
```

Prosseguir para o passo iterate.

**Se `human_needed`:**

Ler a seção human_verification do VERIFICATION.md para obter a contagem e itens que requerem teste manual.

Exibir os itens, depois perguntar ao usuário via conversational prompting:
- **question:** "Fase ${PHASE_NUM} tem itens que precisam de verificação manual. Validar agora ou continuar para a próxima fase?"
- **options:** "Validar agora" / "Continuar sem validação"

Em **"Validar agora"**: Apresentar os itens específicos da seção human_verification do VERIFICATION.md. Após o usuário revisar, perguntar:
- **question:** "Resultado da validação?"
- **options:** "Tudo certo — continuar" / "Encontrei problemas"

Em "Tudo certo — continuar": Exibir `Fase ${PHASE_NUM} ✅ Validação humana aprovada` e prosseguir para o passo iterate.

Em "Encontrei problemas": Ir para handle_blocker com os problemas reportados pelo usuário como descrição.

Em **"Continuar sem validação"**: Exibir `Fase ${PHASE_NUM} ⏭ Validação humana adiada` e prosseguir para o passo iterate.

**Se `gaps_found`:**

Ler resumo das lacunas do VERIFICATION.md (pontuação e itens faltantes). Exibir:
```
⚠ Fase ${PHASE_NUM}: ${PHASE_NAME} — Lacunas Encontradas
Pontuação: {N}/{M} obrigatórios verificados
```

Perguntar ao usuário via conversational prompting:
- **question:** "Lacunas encontradas na fase ${PHASE_NUM}. Como prosseguir?"
- **options:** "Executar fechamento de lacunas" / "Continuar sem corrigir" / "Parar modo autônomo"

Em **"Executar fechamento de lacunas"**: Executar ciclo de fechamento de lacunas (limite: 1 tentativa):

```
Skill(skill="gsd-plan-phase", args="${PHASE_NUM} --gaps")
```

Verificar que planos de lacuna foram criados — re-executar `init phase-op ${PHASE_NUM}` e verificar `has_plans`. Se nenhum plano de lacuna → ir para handle_blocker: "Planejamento de fechamento de lacunas para fase ${PHASE_NUM} não produziu planos."

Re-executar:
```
Skill(skill="gsd-execute-phase", args="${PHASE_NUM} --no-transition")
```

Re-ler status de verificação:
```bash
VERIFY_STATUS=$(grep "^status:" "${PHASE_DIR}"/*-VERIFICATION.md 2>/dev/null | head -1 | cut -d: -f2 | tr -d ' ')
```

Se `passed` ou `human_needed`: Rotear normalmente (continuar ou perguntar ao usuário como acima).

Se ainda `gaps_found` após esta tentativa: Exibir "Lacunas persistem após tentativa de fechamento." e perguntar via conversational prompting:
- **question:** "Fechamento de lacunas não resolveu completamente os problemas. Como prosseguir?"
- **options:** "Continuar mesmo assim" / "Parar modo autônomo"

Em "Continuar mesmo assim": Prosseguir para o passo iterate.
Em "Parar modo autônomo": Ir para handle_blocker.

Isso limita o fechamento de lacunas a 1 tentativa automática para prevenir loops infinitos.

Em **"Continuar sem corrigir"**: Exibir `Fase ${PHASE_NUM} ⏭ Lacunas adiadas` e prosseguir para o passo iterate.

Em **"Parar modo autônomo"**: Ir para handle_blocker com "Usuário parou — lacunas permanecem na fase ${PHASE_NUM}".

</step>

<step name="smart_discuss">

## Discuss Inteligente

Executar discuss inteligente para a fase atual. Propõe respostas de áreas cinzentas em tabelas de lote — o usuário aceita ou substitui por área. Produz saída CONTEXT.md idêntica ao discuss-phase regular.

> **Nota:** Discuss inteligente é uma variante otimizada para autônomo da skill `gsd-discuss-phase`. Produz saída CONTEXT.md idêntica mas usa propostas em tabelas de lote em vez de questionamento sequencial. A skill original `discuss-phase` permanece inalterada (por CTRL-03). Marcos futuros podem extrair isso para um arquivo de skill separado.

**Entradas:** `PHASE_NUM` de execute_phase. Executar init para obter caminhos da fase:

```bash
PHASE_STATE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Analisar do JSON: `phase_dir`, `phase_slug`, `padded_phase`, `phase_name`.

---

### Sub-passo 1: Carregar contexto anterior

Ler contexto de nível de projeto e fases anteriores para evitar re-perguntar questões já decididas.

**Ler arquivos do projeto:**

```bash
cat .planning/PROJECT.md 2>/dev/null
cat .planning/REQUIREMENTS.md 2>/dev/null
cat .planning/STATE.md 2>/dev/null
```

Extrair destes:
- **PROJECT.md** — Visão, princípios, inegociáveis, preferências do usuário
- **REQUIREMENTS.md** — Critérios de aceitação, restrições, obrigatórios vs desejáveis
- **STATE.md** — Progresso atual, decisões registradas até agora

**Ler todos os arquivos CONTEXT.md anteriores:**

```bash
find .planning/phases -name "*-CONTEXT.md" 2>/dev/null | sort
```

Para cada CONTEXT.md onde número da fase < fase atual:
- Ler a seção `<decisions>` — estas são preferências travadas
- Ler `<specifics>` — referências particulares ou momentos "quero parecido com X"
- Notar padrões (ex: "usuário consistentemente prefere UI minimalista", "usuário rejeitou output verboso")

**Construir contexto interno prior_decisions** (não escrever em arquivo):

```
<prior_decisions>
## Nível de Projeto
- [Princípio ou restrição chave do PROJECT.md]
- [Requisito afetando esta fase do REQUIREMENTS.md]

## De Fases Anteriores
### Fase N: [Nome]
- [Decisão relevante para a fase atual]
- [Preferência que estabelece um padrão]
</prior_decisions>
```

Se nenhum contexto anterior existir, continuar sem — esperado para fases iniciais.

---

### Sub-passo 2: Explorar Codebase

Varredura leve do codebase para informar identificação de áreas cinzentas e propostas. Manter abaixo de ~5% de contexto.

**Verificar mapas de codebase existentes:**

```bash
ls .planning/codebase/*.md 2>/dev/null
```

**Se mapas de codebase existirem:** Ler os mais relevantes (CONVENTIONS.md, STRUCTURE.md, STACK.md com base no tipo de fase). Extrair componentes reutilizáveis, padrões estabelecidos, pontos de integração. Pular para construção de contexto abaixo.

**Se sem mapas de codebase, fazer grep direcionado:**

Extrair termos-chave do objetivo da fase. Buscar arquivos relacionados:

```bash
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" 2>/dev/null | head -10
ls src/components/ src/hooks/ src/lib/ src/utils/ 2>/dev/null
```

Ler os 3-5 arquivos mais relevantes para entender padrões existentes.

**Construir codebase_context interno** (não escrever em arquivo):
- **Ativos reutilizáveis** — componentes, hooks, utilitários existentes usáveis nesta fase
- **Padrões estabelecidos** — como o codebase faz gerenciamento de estado, estilização, busca de dados
- **Pontos de integração** — onde novo código conecta (rotas, nav, providers)

---

### Sub-passo 3: Analisar Fase e Gerar Propostas

**Obter detalhes da fase:**

```bash
DETAIL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM})
```

Extrair `goal`, `requirements`, `success_criteria` da resposta JSON.

**Detecção de infraestrutura — verificar PRIMEIRO antes de gerar áreas cinzentas:**

Uma fase é infraestrutura pura quando TODOS estes são verdadeiros:
1. Palavras-chave do objetivo correspondem: "scaffolding", "plumbing", "setup", "configuration", "migration", "refactor", "rename", "restructure", "upgrade", "infrastructure"
2. E critérios de sucesso são todos técnicos: "file exists", "test passes", "config valid", "command runs"
3. E nenhum comportamento voltado ao usuário é descrito (sem "users can", "displays", "shows", "presents")

**Se somente infraestrutura:** Pular Sub-passo 4. Ir diretamente para Sub-passo 5 com CONTEXT.md mínimo. Exibir:

```
Fase ${PHASE_NUM}: Fase de infraestrutura — pulando discuss, escrevendo contexto mínimo.
```

Usar estes padrões para o CONTEXT.md:
- `<domain>`: Limite da fase do objetivo do ROADMAP
- `<decisions>`: Uma única subseção "### Discrição do Claude" — "Todas as escolhas de implementação estão a critério do Claude — fase de infraestrutura pura"
- `<code_context>`: O que a exploração do codebase encontrou
- `<specifics>`: "Sem requisitos específicos — fase de infraestrutura"
- `<deferred>`: "Nenhuma"

**Se NÃO for infraestrutura — gerar propostas de áreas cinzentas:**

Determinar tipo de domínio a partir do objetivo da fase:
- Algo que usuários **VEEM** → visual: layout, interações, estados, densidade
- Algo que usuários **CHAMAM** → interface: contratos, respostas, erros, auth
- Algo que usuários **EXECUTAM** → execução: invocação, output, modos de comportamento, flags
- Algo que usuários **LEEM** → conteúdo: estrutura, tom, profundidade, fluxo
- Algo sendo **ORGANIZADO** → organização: critérios, agrupamento, exceções, nomenclatura

Verificar prior_decisions — pular áreas cinzentas já decididas em fases anteriores.

Gerar **3-4 áreas cinzentas** com **~4 perguntas cada**. Para cada pergunta:
- **Pré-selecionar uma resposta recomendada** com base em: decisões anteriores (consistência), padrões do codebase (reutilização), convenções do domínio (abordagens padrão), critérios de sucesso do ROADMAP
- Gerar **1-2 alternativas** por pergunta
- **Anotar** com contexto de decisão anterior ("Você decidiu X na Fase N") e contexto de código ("Componente Y existe com variantes Z") quando relevante

---

### Sub-passo 4: Apresentar Propostas Por Área

Apresentar áreas cinzentas **uma por vez**. Para cada área (M de N):

Exibir uma tabela:

```
### Área Cinzenta {M}/{N}: {Nome da Área}

| # | Pergunta | ✅ Recomendado | Alternativa(s) |
|---|----------|---------------|-----------------|
| 1 | {pergunta} | {resposta} — {justificativa} | {alt1}; {alt2} |
| 2 | {pergunta} | {resposta} — {justificativa} | {alt1} |
| 3 | {pergunta} | {resposta} — {justificativa} | {alt1}; {alt2} |
| 4 | {pergunta} | {resposta} — {justificativa} | {alt1} |
```

Depois perguntar ao usuário via **conversational prompting**:
- **header:** "Área {M}/{N}"
- **question:** "Aceitar estas respostas para {Nome da Área}?"
- **options:** Construir dinamicamente — sempre "Aceitar todas" primeiro, depois "Alterar Q1" até "Alterar QN" para cada pergunta (até 4), depois "Discutir mais fundo" por último. Máximo de 6 opções explícitas (conversational prompting adiciona "Outro" automaticamente).

**Em "Aceitar todas":** Registrar todas as respostas recomendadas para esta área. Mover para a próxima área.

**Em "Alterar QN":** Use conversational prompting com as alternativas para aquela pergunta específica:
- **header:** "{Nome da Área}"
- **question:** "Q{N}: {texto da pergunta}"
- **options:** Listar as 1-2 alternativas mais "Você decide" (mapeia para Discrição do Claude)

Registrar a escolha do usuário. Re-exibir a tabela atualizada com a alteração refletida. Re-apresentar o prompt de aceitação completo para que o usuário possa fazer alterações adicionais ou aceitar.

**Em "Discutir mais fundo":** Mudar para modo interativo para esta área apenas — perguntar uma pergunta por vez usando conversational prompting com 2-3 opções concretas por pergunta mais "Você decide". Após 4 perguntas, perguntar:
- **header:** "{Nome da Área}"
- **question:** "Mais perguntas sobre {nome da área}, ou mover para a próxima?"
- **options:** "Mais perguntas" / "Próxima área"

Se "Mais perguntas", perguntar mais 4. Se "Próxima área", exibir tabela resumo final das respostas capturadas para esta área e seguir em frente.

**Em "Outro" (texto livre):** Interpretar como solicitação de alteração específica ou feedback geral. Incorporar nas decisões da área, re-exibir tabela atualizada, re-apresentar prompt de aceitação.

**Tratamento de expansão de escopo:** Se o usuário mencionar algo fora do domínio da fase:

```
"{Funcionalidade} parece ser uma nova capacidade — isso pertence à sua própria fase.
Vou anotá-la como ideia adiada.

Voltando a {área atual}: {retornar à pergunta atual}"
```

Rastrear ideias adiadas internamente para inclusão no CONTEXT.md.

---

### Sub-passo 5: Escrever CONTEXT.md

Após todas as áreas serem resolvidas (ou pulo de infraestrutura), escrever o arquivo CONTEXT.md.

**Caminho do arquivo:** `${phase_dir}/${padded_phase}-CONTEXT.md`

Usar **exatamente** esta estrutura (idêntica à saída do discuss-phase):

```markdown
# Fase {PHASE_NUM}: {Nome da Fase} - Contexto

**Coletado:** {data}
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

{Declaração do limite do domínio da análise — o que esta fase entrega}

</domain>

<decisions>
## Decisões de Implementação

### {Nome da Área 1}
- {Resposta aceita/escolhida para Q1}
- {Resposta aceita/escolhida para Q2}
- {Resposta aceita/escolhida para Q3}
- {Resposta aceita/escolhida para Q4}

### {Nome da Área 2}
- {Resposta aceita/escolhida para Q1}
- {Resposta aceita/escolhida para Q2}
...

### Discrição do Claude
{Quaisquer respostas "Você decide" coletadas — notar que Claude tem flexibilidade aqui}

</decisions>

<code_context>
## Insights do Código Existente

### Ativos Reutilizáveis
- {Da exploração do codebase — componentes, hooks, utilitários}

### Padrões Estabelecidos
- {Da exploração do codebase — gerenciamento de estado, estilização, busca de dados}

### Pontos de Integração
- {Da exploração do codebase — onde novo código conecta}

</code_context>

<specifics>
## Ideias Específicas

{Quaisquer referências específicas ou "quero parecido com X" da discussão}
{Se nenhuma: "Sem requisitos específicos — aberto a abordagens padrão"}

</specifics>

<deferred>
## Ideias Adiadas

{Ideias capturadas mas fora do escopo para esta fase}
{Se nenhuma: "Nenhuma — discussão permaneceu dentro do escopo da fase"}

</deferred>
```

Escrever o arquivo.

**Commit:**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${PADDED_PHASE}): smart discuss context" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Exibir confirmação:

```
Criado: {caminho}
Decisões capturadas: {contagem} em {area_count} áreas
```

</step>

<step name="iterate">

## 4. Iterar

Após cada fase completar, reler ROADMAP.md para capturar fases inseridas durante a execução (fases decimais como 5.1):

```bash
ROADMAP=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Re-filtrar fases incompletas usando a mesma lógica de discover_phases:
- Manter fases onde `disk_status !== "complete"` OU `roadmap_complete === false`
- Aplicar filtro `--from N` se originalmente fornecido
- Ordenar por número ascendente

Ler STATE.md atualizado:

```bash
cat .planning/STATE.md
```

Verificar bloqueadores na seção Blockers/Concerns. Se bloqueadores encontrados, ir para handle_blocker com a descrição do bloqueador.

Se fases incompletas restarem: prosseguir para a próxima fase, voltar ao loop execute_phase.

Se todas as fases estiverem completas, prosseguir para o passo lifecycle.

</step>

<step name="lifecycle">

## 5. Ciclo de Vida

Após todas as fases completarem, executar a sequência de ciclo de vida do marco: auditar → completar → limpar.

Exibir banner de transição de ciclo de vida:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTÔNOMO ▸ CICLO DE VIDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Todas as fases completas → Iniciando ciclo de vida: auditar → completar → limpar
 Marco: {milestone_version} — {milestone_name}
```

**5a. Auditoria**

```
Skill(skill="gsd-audit-milestone")
```

Após a auditoria completar, detectar o resultado:

```bash
AUDIT_FILE=".planning/v${milestone_version}-MILESTONE-AUDIT.md"
AUDIT_STATUS=$(grep "^status:" "${AUDIT_FILE}" 2>/dev/null | head -1 | cut -d: -f2 | tr -d ' ')
```

**Se AUDIT_STATUS estiver vazio** (sem arquivo de auditoria ou sem campo status):

Ir para handle_blocker: "Auditoria não produziu resultados — arquivo de auditoria ausente ou malformado."

**Se `passed`:**

Exibir:
```
Auditoria ✅ aprovada — prosseguindo para completar marco
```

Prosseguir para 5b (sem pausa do usuário — por CTRL-01).

**Se `gaps_found`:**

Ler o resumo das lacunas do arquivo de auditoria. Exibir:
```
⚠ Auditoria: Lacunas Encontradas
```

Perguntar ao usuário via conversational prompting:
- **question:** "Auditoria do marco encontrou lacunas. Como prosseguir?"
- **options:** "Continuar mesmo assim — aceitar lacunas" / "Parar — corrigir lacunas manualmente"

Em **"Continuar mesmo assim"**: Exibir `Auditoria ⏭ Lacunas aceitas — prosseguindo para completar marco` e prosseguir para 5b.

Em **"Parar"**: Ir para handle_blocker com "Usuário parou — lacunas de auditoria permanecem. Execute /gsd-audit-milestone para revisar, depois /gsd-complete-milestone quando pronto."

**Se `tech_debt`:**

Ler o resumo de dívida técnica do arquivo de auditoria. Exibir:
```
⚠ Auditoria: Dívida Técnica Identificada
```

Mostrar o resumo, depois perguntar ao usuário via conversational prompting:
- **question:** "Auditoria do marco encontrou dívida técnica. Como prosseguir?"
- **options:** "Continuar com dívida técnica" / "Parar — tratar dívida primeiro"

Em **"Continuar com dívida técnica"**: Exibir `Auditoria ⏭ Dívida técnica reconhecida — prosseguindo para completar marco` e prosseguir para 5b.

Em **"Parar"**: Ir para handle_blocker com "Usuário parou — dívida técnica a tratar. Execute /gsd-audit-milestone para revisar detalhes."

**5b. Completar Marco**

```
Skill(skill="gsd-complete-milestone", args="${milestone_version}")
```

Após complete-milestone retornar, verificar que produziu saída:

```bash
ls .planning/milestones/v${milestone_version}-ROADMAP.md 2>/dev/null
```

Se o arquivo de arquivamento não existir, ir para handle_blocker: "Completar marco não produziu os arquivos de arquivo esperados."

**5c. Limpeza**

```
Skill(skill="gsd-cleanup")
```

Limpeza mostra seu próprio dry-run e pede aprovação do usuário internamente — esta é uma pausa aceitável por CTRL-01 já que é uma decisão explícita sobre exclusão de arquivos.

**5d. Conclusão Final**

Exibir banner de conclusão final:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTÔNOMO ▸ COMPLETO 🎉
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Marco: {milestone_version} — {milestone_name}
 Status: Completo ✅
 Ciclo de vida: auditoria ✅ → completar ✅ → limpeza ✅

 Manda ver! 🚀
```

</step>

<step name="handle_blocker">

## 6. Tratar Bloqueador

Quando qualquer operação de fase falha ou um bloqueador é detectado, apresentar 3 opções via conversational prompting:

**Prompt:** "Fase {N} ({Nome}) encontrou um problema: {descrição}"

**Opções:**
1. **"Corrigir e tentar novamente"** — Re-executar o passo que falhou (discuss, plan ou execute) para esta fase
2. **"Pular esta fase"** — Marcar fase como pulada, continuar para a próxima fase incompleta
3. **"Parar modo autônomo"** — Exibir resumo do progresso até agora e sair de forma limpa

**Em "Corrigir e tentar novamente":** Voltar ao passo que falhou dentro de execute_phase. Se o mesmo passo falhar novamente após a tentativa, re-apresentar estas opções.

**Em "Pular esta fase":** Registrar `Fase {N} ⏭ {Nome} — Pulada pelo usuário` e prosseguir para iterate.

**Em "Parar modo autônomo":** Exibir resumo do progresso:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTÔNOMO ▸ PARADO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Completadas: {lista de fases completadas}
 Puladas: {lista de fases puladas}
 Restantes: {lista de fases restantes}

 Retomar com: /gsd-autonomous --from {next_phase}
```

</step>

</process>

<success_criteria>
- [ ] Todas as fases incompletas executadas em ordem (discuss inteligente → planejar → executar cada)
- [ ] Discuss inteligente propõe respostas de áreas cinzentas em tabelas, usuário aceita ou substitui por área
- [ ] Banners de progresso exibidos entre fases
- [ ] Execute-phase invocado com --no-transition (autônomo gerencia transições)
- [ ] Verificação pós-execução lê VERIFICATION.md e roteia conforme status
- [ ] Verificação aprovada → continuar automaticamente para próxima fase
- [ ] Verificação necessitando humano → usuário solicitado a validar ou pular
- [ ] Lacunas encontradas → usuário oferecido fechamento de lacunas, continuar ou parar
- [ ] Fechamento de lacunas limitado a 1 tentativa (previne loops infinitos)
- [ ] Falhas em plan-phase e execute-phase roteiam para handle_blocker
- [ ] ROADMAP.md relido após cada fase (captura fases inseridas)
- [ ] STATE.md verificado para bloqueadores antes de cada fase
- [ ] Bloqueadores tratados via escolha do usuário (tentar novamente / pular / parar)
- [ ] Conclusão final ou resumo de parada exibido
- [ ] Após todas as fases completarem, passo de ciclo de vida é invocado (não sugestão manual)
- [ ] Banner de transição de ciclo de vida exibido antes da auditoria
- [ ] Auditoria invocada via Skill(skill="gsd-audit-milestone")
- [ ] Roteamento de resultado da auditoria: passed → auto-continuar, gaps_found → usuário decide, tech_debt → usuário decide
- [ ] Falha técnica na auditoria (sem arquivo/sem status) roteia para handle_blocker
- [ ] Complete-milestone invocado via Skill() com argumento ${milestone_version}
- [ ] Limpeza invocada via Skill() — confirmação interna é aceitável (CTRL-01)
- [ ] Banner de conclusão final exibido após ciclo de vida
- [ ] Barra de progresso usa número da fase / total de fases do marco (não posição entre incompletas)
- [ ] Discuss inteligente documenta relação com discuss-phase com nota CTRL-03
</success_criteria>
</output>
