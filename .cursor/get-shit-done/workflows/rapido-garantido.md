<purpose>
Executar tarefas pequenas e pontuais com garantias GSD (commits atômicos, rastreamento no STATE.md). Modo rápido dispara gsd-planejador (modo quick) + gsd-executor(es), rastreia tarefas em `.planning/quick/`, e atualiza a tabela "Quick Tasks Completed" do STATE.md.

Com flag `--discuss`: fase de discussão leve antes do planejamento. Levanta premissas, esclarece áreas cinzentas, captura decisões em CONTEXT.md para que o planejador as trate como travadas.

Com flag `--full`: habilita verificação de plano (máx 2 iterações) e verificação pós-execução para garantias de qualidade sem a cerimônia completa de marco.

Com flag `--research`: dispara agente de pesquisa focado antes do planejamento. Investiga abordagens de implementação, opções de biblioteca e armadilhas. Use quando não tem certeza de como abordar uma tarefa.

Flags são combináveis: `--discuss --research --full` dá discussão + pesquisa + verificação de plano + verificação.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>
**Passo 1: Analisar argumentos e obter descrição da tarefa**

Analisar `{{GSD_ARGS}}` para:
- Flag `--full` → armazenar como `$FULL_MODE` (true/false)
- Flag `--discuss` → armazenar como `$DISCUSS_MODE` (true/false)
- Flag `--research` → armazenar como `$RESEARCH_MODE` (true/false)
- Texto restante → usar como `$DESCRIPTION` se não vazio

Se `$DESCRIPTION` estiver vazio após análise, solicitar ao usuário interativamente:

```
conversational prompting(
  header: "Tarefa Rápida",
  question: "O que você quer fazer?",
  followUp: null
)
```

Armazenar resposta como `$DESCRIPTION`.

Se ainda vazio, re-solicitar: "Por favor, forneça uma descrição da tarefa."

Exibir banner baseado nas flags ativas:

Se `$DISCUSS_MODE` e `$RESEARCH_MODE` e `$FULL_MODE`:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSSÃO + PESQUISA + COMPLETO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + pesquisa + verificação de plano + verificação habilitados
```

Se `$DISCUSS_MODE` e `$FULL_MODE` (sem pesquisa):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSSÃO + COMPLETO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + verificação de plano + verificação habilitados
```

Se `$DISCUSS_MODE` e `$RESEARCH_MODE` (sem full):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSSÃO + PESQUISA)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + pesquisa habilitados
```

Se `$RESEARCH_MODE` e `$FULL_MODE` (sem discuss):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (PESQUISA + COMPLETO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Pesquisa + verificação de plano + verificação habilitados
```

Se `$DISCUSS_MODE` apenas:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSSÃO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Fase de discussão habilitada — levantando áreas cinzentas antes do planejamento
```

Se `$RESEARCH_MODE` apenas:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (PESQUISA)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Fase de pesquisa habilitada — investigando abordagens antes do planejamento
```

Se `$FULL_MODE` apenas:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (MODO COMPLETO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Verificação de plano + verificação habilitados
```

---

**Passo 2: Inicializar**

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init quick "$DESCRIPTION")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON: `planner_model`, `executor_model`, `checker_model`, `verifier_model`, `commit_docs`, `branch_name`, `quick_id`, `slug`, `date`, `timestamp`, `quick_dir`, `task_dir`, `roadmap_exists`, `planning_exists`.

**Se `roadmap_exists` for false:** Erro — Modo rápido requer um projeto ativo com ROADMAP.md. Execute `/gsd-novo-projeto` primeiro.

Tarefas rápidas podem rodar no meio de uma fase - validação apenas verifica se ROADMAP.md existe, não status da fase.

---

**Passo 2.5: Tratar branching de tarefa rápida**

**Se `branch_name` estiver vazio/null:** Pular e continuar na branch atual.

**Se `branch_name` estiver definido:** Fazer checkout da branch de tarefa rápida antes de qualquer commit de planejamento:

```bash
git checkout -b "$branch_name" 2>/dev/null || git checkout "$branch_name"
```

Todos os commits de tarefa rápida para esta execução ficam nessa branch. Usuário cuida do merge/rebase depois.

---

**Passo 3: Criar diretório da tarefa**

```bash
mkdir -p "${task_dir}"
```

---

**Passo 4: Criar diretório de tarefa rápida**

Criar o diretório para esta tarefa rápida:

```bash
QUICK_DIR=".planning/quick/${quick_id}-${slug}"
mkdir -p "$QUICK_DIR"
```

Reportar ao usuário:
```
Criando tarefa rápida ${quick_id}: ${DESCRIPTION}
Diretório: ${QUICK_DIR}
```

Armazenar `$QUICK_DIR` para uso na orquestração.

---

**Passo 4.5: Fase de discussão (apenas quando `$DISCUSS_MODE`)**

Pular esta etapa inteiramente se NÃO `$DISCUSS_MODE`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DISCUTINDO TAREFA RÁPIDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Levantando áreas cinzentas para: ${DESCRIPTION}
```

**4.5a. Identificar áreas cinzentas**

Analisar `$DESCRIPTION` para identificar 2-4 áreas cinzentas — decisões de implementação que mudariam o resultado e que o usuário deveria opinar.

Usar heurística orientada ao domínio para gerar áreas cinzentas específicas da fase (não genéricas):
- Algo que usuários **VEEM** → layout, densidade, interações, estados
- Algo que usuários **CHAMAM** → respostas, erros, autenticação, versionamento
- Algo que usuários **EXECUTAM** → formato de saída, flags, modos, tratamento de erros
- Algo que usuários **LEEM** → estrutura, tom, profundidade, fluxo
- Algo sendo **ORGANIZADO** → critérios, agrupamento, nomenclatura, exceções

Cada área cinzenta deve ser um ponto de decisão concreto, não uma categoria vaga. Exemplo: "Comportamento de carregamento" não "UX".

**4.5b. Apresentar áreas cinzentas**

```
conversational prompting(
  header: "Áreas Cinzentas",
  question: "Quais áreas precisam de esclarecimento antes do planejamento?",
  options: [
    { label: "${area_1}", description: "${por_que_importa_1}" },
    { label: "${area_2}", description: "${por_que_importa_2}" },
    { label: "${area_3}", description: "${por_que_importa_3}" },
    { label: "Tudo certo", description: "Pular discussão — eu sei o que quero" }
  ],
  multiSelect: true
)
```

Se usuário selecionar "Tudo certo" → pular para Passo 5 (nenhum CONTEXT.md escrito).

**4.5c. Discutir áreas selecionadas**

Para cada área selecionada, fazer 1-2 perguntas focadas via conversational prompting:

```
conversational prompting(
  header: "${area_name}",
  question: "${pergunta_específica_sobre_esta_área}",
  options: [
    { label: "${escolha_concreta_1}", description: "${o_que_isso_significa}" },
    { label: "${escolha_concreta_2}", description: "${o_que_isso_significa}" },
    { label: "${escolha_concreta_3}", description: "${o_que_isso_significa}" },
    { label: "Você decide", description: "Critério do Claude" }
  ],
  multiSelect: false
)
```

Regras:
- Opções devem ser escolhas concretas, não categorias abstratas
- Destacar escolha recomendada onde você tem opinião clara
- Se usuário selecionar "Outro" com texto livre, mudar para acompanhamento em texto puro (por regra de texto livre do questionamento.md)
- Se usuário selecionar "Você decide", capturar como Critério do Claude no CONTEXT.md
- Máx 2 perguntas por área — isso é leve, não um mergulho profundo

Coletar todas as decisões em `$DECISIONS`.

**4.5d. Escrever CONTEXT.md**

Escrever `${QUICK_DIR}/${quick_id}-CONTEXT.md` usando a estrutura padrão de template de contexto:

```markdown
# Tarefa Rápida ${quick_id}: ${DESCRIPTION} - Contexto

**Coletado:** ${date}
**Status:** Pronto para planejamento

<domain>
## Limite da Tarefa

${DESCRIPTION}

</domain>

<decisions>
## Decisões de Implementação

### ${area_1_name}
- ${decisão_da_discussão}

### ${area_2_name}
- ${decisão_da_discussão}

### Critério do Claude
${áreas_onde_usuário_disse_você_decide_ou_áreas_não_discutidas}

</decisions>

<specifics>
## Ideias Específicas

${referências_ou_exemplos_específicos_da_discussão}

[Se nenhuma: "Sem requisitos específicos — aberto a abordagens padrão"]

</specifics>

<canonical_refs>
## Referências Canônicas

${specs_adrs_ou_docs_referenciados_durante_discussão}

[Se nenhuma: "Sem specs externas — requisitos totalmente capturados nas decisões acima"]

</canonical_refs>
```

Nota: CONTEXT.md de tarefa rápida omite seções `<code_context>` e `<deferred>` (sem scouting de base de código, sem escopo de fase para adiar). Manter enxuto. A seção `<canonical_refs>` é incluída quando docs externos foram referenciados — omitir apenas se nenhum doc externo se aplicar.

Reportar: `Contexto capturado: ${QUICK_DIR}/${quick_id}-CONTEXT.md`

---

**Passo 4.75: Fase de pesquisa (apenas quando `$RESEARCH_MODE`)**

Pular esta etapa inteiramente se NÃO `$RESEARCH_MODE`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISANDO TAREFA RÁPIDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Investigando abordagens para: ${DESCRIPTION}
```

Disparar um único pesquisador focado (não 4 pesquisadores em paralelo como fases completas — tarefas rápidas precisam de pesquisa direcionada, não pesquisas amplas de domínio):

```
Task(
  prompt="
<research_context>

**Modo:** quick-task
**Tarefa:** ${DESCRIPTION}
**Saída:** ${QUICK_DIR}/${quick_id}-RESEARCH.md

<files_to_read>
- .planning/STATE.md (Estado do projeto — o que já foi construído)
- .planning/PROJECT.md (Contexto do projeto)
- .cursor/rules/ (se existir — diretrizes específicas do projeto)
${DISCUSS_MODE ? '- ' + QUICK_DIR + '/' + quick_id + '-CONTEXT.md (Decisões do usuário — pesquisa deve alinhar com estas)' : ''}
</files_to_read>

</research_context>

<focus>
Esta é uma tarefa rápida, não uma fase completa. Pesquisa deve ser concisa e direcionada:
1. Melhores bibliotecas/padrões para esta tarefa específica
2. Armadilhas comuns e como evitá-las
3. Pontos de integração com base de código existente
4. Quaisquer restrições ou pegadinhas que vale saber antes de planejar

NÃO produza uma pesquisa completa de domínio. Alvo de 1-2 páginas de descobertas acionáveis.
</focus>

<output>
Escrever pesquisa em: ${QUICK_DIR}/${quick_id}-RESEARCH.md
Usar formato de pesquisa padrão mas manter enxuto — pular seções que não se aplicam.
Retornar: ## PESQUISA CONCLUÍDA com caminho do arquivo
</output>
",
  subagent_type="gsd-pesquisador-fase",
  model="{planner_model}",
  description="Pesquisar: ${DESCRIPTION}"
)
```

Após pesquisador retornar:
1. Verificar se pesquisa existe em `${QUICK_DIR}/${quick_id}-RESEARCH.md`
2. Reportar: "Pesquisa concluída: ${QUICK_DIR}/${quick_id}-RESEARCH.md"

Se arquivo de pesquisa não encontrado, avisar mas continuar: "Agente de pesquisa não produziu saída — prosseguindo para planejamento sem pesquisa."

---

**Passo 5: Disparar planejador (modo quick)**

**Se `$FULL_MODE`:** Usar modo `quick-full` com restrições mais rígidas.

**Se NÃO `$FULL_MODE`:** Usar modo `quick` padrão.

```
Task(
  prompt="
<planning_context>

**Modo:** ${FULL_MODE ? 'quick-full' : 'quick'}
**Diretório:** ${QUICK_DIR}
**Descrição:** ${DESCRIPTION}

<files_to_read>
- .planning/STATE.md (Estado do Projeto)
- .cursor/rules/ (se existir — seguir diretrizes específicas do projeto)
${DISCUSS_MODE ? '- ' + QUICK_DIR + '/' + quick_id + '-CONTEXT.md (Decisões do usuário — travadas, não revisitar)' : ''}
${RESEARCH_MODE ? '- ' + QUICK_DIR + '/' + quick_id + '-RESEARCH.md (Descobertas da pesquisa — usar para informar escolhas de implementação)' : ''}
</files_to_read>

**Skills do projeto:** Verificar diretório .cursor/skills/ ou .agents/skills/ (se existir) — ler arquivos SKILL.md, planos devem considerar regras de skills do projeto

</planning_context>

<constraints>
- Criar um ÚNICO plano com 1-3 tarefas focadas
- Tarefas rápidas devem ser atômicas e auto-contidas
${RESEARCH_MODE ? '- Descobertas de pesquisa estão disponíveis — usá-las para informar escolhas de biblioteca/padrão' : '- Sem fase de pesquisa'}
${FULL_MODE ? '- Almejar ~40% de uso de contexto (estruturado para verificação)' : '- Almejar ~30% de uso de contexto (simples, focado)'}
${FULL_MODE ? '- DEVE gerar `must_haves` no frontmatter do plano (truths, artifacts, key_links)' : ''}
${FULL_MODE ? '- Cada tarefa DEVE ter campos `files`, `action`, `verify`, `done`' : ''}
</constraints>

<output>
Escrever plano em: ${QUICK_DIR}/${quick_id}-PLAN.md
Retornar: ## PLANEJAMENTO CONCLUÍDO com caminho do plano
</output>
",
  subagent_type="gsd-planejador",
  model="{planner_model}",
  description="Plano rápido: ${DESCRIPTION}"
)
```

Após planejador retornar:
1. Verificar se plano existe em `${QUICK_DIR}/${quick_id}-PLAN.md`
2. Extrair contagem de planos (tipicamente 1 para tarefas rápidas)
3. Reportar: "Plano criado: ${QUICK_DIR}/${quick_id}-PLAN.md"

Se plano não encontrado, erro: "Planejador falhou ao criar ${quick_id}-PLAN.md"

---

**Passo 5.5: Loop de verificação de plano (apenas quando `$FULL_MODE`)**

Pular esta etapa inteiramente se NÃO `$FULL_MODE`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO PLANO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Disparando verificador de plano...
```

Prompt do verificador:

```markdown
<verification_context>
**Modo:** quick-full
**Descrição da Tarefa:** ${DESCRIPTION}

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano para verificar)
</files_to_read>

**Escopo:** Esta é uma tarefa rápida, não uma fase completa. Pular verificações que requerem objetivo de fase do ROADMAP.
</verification_context>

<check_dimensions>
- Cobertura de requisitos: O plano atende à descrição da tarefa?
- Completude das tarefas: Tarefas têm campos files, action, verify, done?
- Links-chave: Arquivos referenciados são reais?
- Sanidade de escopo: Tamanho apropriado para uma tarefa rápida (1-3 tarefas)?
- Derivação de must_haves: must_haves são rastreáveis à descrição da tarefa?

Pular: deps entre planos (plano único), alinhamento com ROADMAP
${DISCUSS_MODE ? '- Conformidade de contexto: O plano honra decisões travadas do CONTEXT.md?' : '- Pular: conformidade de contexto (sem CONTEXT.md)'}
</check_dimensions>

<expected_output>
- ## VERIFICAÇÃO APROVADA — todas as verificações passaram
- ## PROBLEMAS ENCONTRADOS — lista estruturada de problemas
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-verificador-plano",
  model="{checker_model}",
  description="Verificar plano rápido: ${DESCRIPTION}"
)
```

**Tratar retorno do verificador:**

- **`## VERIFICAÇÃO APROVADA`:** Exibir confirmação, prosseguir para passo 6.
- **`## PROBLEMAS ENCONTRADOS`:** Exibir problemas, verificar contagem de iterações, entrar no loop de revisão.

**Loop de revisão (máx 2 iterações):**

Rastrear `iteration_count` (inicia em 1 após plano inicial + verificação).

**Se iteration_count < 2:**

Exibir: `Enviando de volta ao planejador para revisão... (iteração ${N}/2)`

Prompt de revisão:

```markdown
<revision_context>
**Modo:** quick-full (revisão)

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano existente)
</files_to_read>

**Problemas do verificador:** ${structured_issues_from_checker}

</revision_context>

<instructions>
Fazer atualizações direcionadas para resolver problemas do verificador.
NÃO replanejar do zero a menos que problemas sejam fundamentais.
Retornar o que mudou.
</instructions>
```

```
Task(
  prompt=revision_prompt,
  subagent_type="gsd-planejador",
  model="{planner_model}",
  description="Revisar plano rápido: ${DESCRIPTION}"
)
```

Após planejador retornar → disparar verificador novamente, incrementar iteration_count.

**Se iteration_count >= 2:**

Exibir: `Máximo de iterações alcançado. ${N} problemas restantes:` + lista de problemas

Oferecer: 1) Forçar prosseguir, 2) Abortar

---

**Passo 6: Disparar executor**

Disparar gsd-executor com referência ao plano:

```
Task(
  prompt="
Executar tarefa rápida ${quick_id}.

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano)
- .planning/STATE.md (Estado do projeto)
- .cursor/rules/ (Instruções do projeto, se existir)
- .cursor/skills/ ou .agents/skills/ (Skills do projeto, se existir — listar skills, ler SKILL.md de cada, seguir regras relevantes durante implementação)
</files_to_read>

<constraints>
- Executar todas as tarefas do plano
- Commitar cada tarefa atomicamente
- Criar resumo em: ${QUICK_DIR}/${quick_id}-SUMMARY.md
- NÃO atualizar ROADMAP.md (tarefas rápidas são separadas das fases planejadas)
</constraints>
",
  subagent_type="gsd-executor",
  model="{executor_model}",
  isolation="worktree",
  description="Executar: ${DESCRIPTION}"
)
```

Após executor retornar:
1. Verificar se resumo existe em `${QUICK_DIR}/${quick_id}-SUMMARY.md`
2. Extrair hash do commit da saída do executor
3. Reportar status de conclusão


Se resumo não encontrado, erro: "Executor falhou ao criar ${quick_id}-SUMMARY.md"

Nota: Para tarefas rápidas produzindo múltiplos planos (raro), disparar executores em ondas paralelas conforme padrões de execute-phase.

---

**Passo 6.5: Verificação (apenas quando `$FULL_MODE`)**

Pular esta etapa inteiramente se NÃO `$FULL_MODE`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO RESULTADOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Disparando verificador...
```

```
Task(
  prompt="Verificar alcance do objetivo da tarefa rápida.
Diretório da tarefa: ${QUICK_DIR}
Objetivo da tarefa: ${DESCRIPTION}

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano)
</files_to_read>

Verificar must_haves contra a base de código real. Criar VERIFICATION.md em ${QUICK_DIR}/${quick_id}-VERIFICATION.md.",
  subagent_type="gsd-verificador",
  model="{verifier_model}",
  description="Verificar: ${DESCRIPTION}"
)
```

Ler status de verificação:
```bash
grep "^status:" "${QUICK_DIR}/${quick_id}-VERIFICATION.md" | cut -d: -f2 | tr -d ' '
```

Armazenar como `$VERIFICATION_STATUS`.

| Status | Ação |
|--------|--------|
| `passed` | Armazenar `$VERIFICATION_STATUS = "Verificado"`, continuar para passo 7 |
| `human_needed` | Exibir itens que precisam de verificação manual, armazenar `$VERIFICATION_STATUS = "Precisa Revisão"`, continuar |
| `gaps_found` | Exibir resumo de lacunas, oferecer: 1) Re-executar executor para corrigir lacunas, 2) Aceitar como está. Armazenar `$VERIFICATION_STATUS = "Lacunas"` |

---

**Passo 7: Atualizar STATE.md**

Atualizar STATE.md com registro de conclusão da tarefa rápida.

**7a. Verificar se seção "Quick Tasks Completed" existe:**

Ler STATE.md e verificar seção `### Quick Tasks Completed`.

**7b. Se seção não existir, criá-la:**

Inserir após seção `### Blockers/Concerns`:

**Se `$FULL_MODE`:**
```markdown
### Quick Tasks Completed

| # | Descrição | Data | Commit | Status | Diretório |
|---|-----------|------|--------|--------|-----------|
```

**Se NÃO `$FULL_MODE`:**
```markdown
### Quick Tasks Completed

| # | Descrição | Data | Commit | Diretório |
|---|-----------|------|--------|-----------|
```

**Nota:** Se a tabela já existir, corresponder seu formato de colunas existente. Se adicionando `--full` a um projeto que já tem tarefas rápidas sem coluna Status, adicionar a coluna Status às linhas de cabeçalho e separador, e deixar Status vazio para os predecessores da nova linha.

**7c. Adicionar nova linha à tabela:**

Usar `date` do init:

**Se `$FULL_MODE` (ou tabela tem coluna Status):**
```markdown
| ${quick_id} | ${DESCRIPTION} | ${date} | ${commit_hash} | ${VERIFICATION_STATUS} | [${quick_id}-${slug}](./quick/${quick_id}-${slug}/) |
```

**Se NÃO `$FULL_MODE` (e tabela não tem coluna Status):**
```markdown
| ${quick_id} | ${DESCRIPTION} | ${date} | ${commit_hash} | [${quick_id}-${slug}](./quick/${quick_id}-${slug}/) |
```

**7d. Atualizar linha "Last activity":**

Usar `date` do init:
```
Last activity: ${date} - Tarefa rápida ${quick_id} concluída: ${DESCRIPTION}
```

Usar ferramenta Edit para fazer estas mudanças atomicamente

---

**Passo 8: Commit final e conclusão**

Adicionar e commitar artefatos de tarefa rápida:

Construir lista de arquivos:
- `${QUICK_DIR}/${quick_id}-PLAN.md`
- `${QUICK_DIR}/${quick_id}-SUMMARY.md`
- `.planning/STATE.md`
- Se `$DISCUSS_MODE` e arquivo de contexto existir: `${QUICK_DIR}/${quick_id}-CONTEXT.md`
- Se `$RESEARCH_MODE` e arquivo de pesquisa existir: `${QUICK_DIR}/${quick_id}-RESEARCH.md`
- Se `$FULL_MODE` e arquivo de verificação existir: `${QUICK_DIR}/${quick_id}-VERIFICATION.md`

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(quick-${quick_id}): ${DESCRIPTION}" --files ${file_list}
```

Obter hash do commit final:
```bash
commit_hash=$(git rev-parse --short HEAD)
```

Exibir saída de conclusão:

**Se `$FULL_MODE`:**
```
---

GSD > TAREFA RÁPIDA CONCLUÍDA (MODO COMPLETO)

Tarefa Rápida ${quick_id}: ${DESCRIPTION}

${RESEARCH_MODE ? 'Pesquisa: ' + QUICK_DIR + '/' + quick_id + '-RESEARCH.md' : ''}
Resumo: ${QUICK_DIR}/${quick_id}-SUMMARY.md
Verificação: ${QUICK_DIR}/${quick_id}-VERIFICATION.md (${VERIFICATION_STATUS})
Commit: ${commit_hash}

---

Pronto para próxima tarefa: /gsd-rapido-garantido ${GSD_WS}
```

**Se NÃO `$FULL_MODE`:**
```
---

GSD > TAREFA RÁPIDA CONCLUÍDA

Tarefa Rápida ${quick_id}: ${DESCRIPTION}

${RESEARCH_MODE ? 'Pesquisa: ' + QUICK_DIR + '/' + quick_id + '-RESEARCH.md' : ''}
Resumo: ${QUICK_DIR}/${quick_id}-SUMMARY.md
Commit: ${commit_hash}

---

Pronto para próxima tarefa: /gsd-rapido-garantido ${GSD_WS}
```

</process>

<success_criteria>
- [ ] Validação de ROADMAP.md aprovada
- [ ] Usuário forneceu descrição da tarefa
- [ ] Flags `--full`, `--discuss` e `--research` analisadas dos argumentos quando presentes
- [ ] Slug gerado (minúsculas, hífens, máx 40 caracteres)
- [ ] Quick ID gerado (formato YYMMDD-xxx, precisão Base36 de 2s)
- [ ] Diretório criado em `.planning/quick/YYMMDD-xxx-slug/`
- [ ] (--discuss) Áreas cinzentas identificadas e apresentadas, decisões capturadas em `${quick_id}-CONTEXT.md`
- [ ] (--research) Agente de pesquisa disparado, `${quick_id}-RESEARCH.md` criado
- [ ] `${quick_id}-PLAN.md` criado pelo planejador (honra decisões do CONTEXT.md quando --discuss, usa descobertas do RESEARCH.md quando --research)
- [ ] (--full) Verificador de plano valida plano, loop de revisão limitado a 2
- [ ] `${quick_id}-SUMMARY.md` criado pelo executor
- [ ] (--full) `${quick_id}-VERIFICATION.md` criado pelo verificador
- [ ] STATE.md atualizado com linha de tarefa rápida (coluna Status quando --full)
- [ ] Artefatos commitados
</success_criteria>
