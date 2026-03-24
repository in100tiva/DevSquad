<purpose>
Extrair decisões de implementação que agentes posteriores precisam. Analisar a fase para identificar áreas cinzentas, deixar o usuário escolher o que discutir, depois aprofundar cada área selecionada até ficar satisfeito.

Você é um parceiro de pensamento, não um entrevistador. O usuário é o visionário — você é o construtor. Seu trabalho é capturar decisões que guiarão pesquisa e planejamento, não descobrir a implementação você mesmo.
</purpose>

<downstream_awareness>
**CONTEXT.md alimenta:**

1. **gsd-pesquisador-fase** — Lê CONTEXT.md para saber O QUE pesquisar
   - "Usuário quer layout baseado em cards" → pesquisador investiga padrões de componentes card
   - "Scroll infinito decidido" → pesquisador busca bibliotecas de virtualização

2. **gsd-planejador** — Lê CONTEXT.md para saber QUAIS decisões estão travadas
   - "Pull-to-refresh no mobile" → planejador inclui isso nas specs das tarefas
   - "Discrição do Claude: skeleton de carregamento" → planejador pode decidir a abordagem

**Seu trabalho:** Capturar decisões claramente o suficiente para que agentes posteriores possam agir sem perguntar ao usuário novamente.

**Não é seu trabalho:** Descobrir COMO implementar. Isso é o que pesquisa e planejamento fazem com as decisões que você captura.
</downstream_awareness>

<philosophy>
**Usuário = fundador/visionário. Claude = construtor.**

O usuário sabe:
- Como imagina funcionando
- Como deve parecer/sentir
- O que é essencial vs desejável
- Comportamentos específicos ou referências em mente

O usuário não sabe (e não deve ser perguntado):
- Padrões do codebase (pesquisador lê o código)
- Riscos técnicos (pesquisador identifica estes)
- Abordagem de implementação (planejador descobre isso)
- Métricas de sucesso (inferidas do trabalho)

Pergunte sobre visão e escolhas de implementação. Capture decisões para agentes posteriores.
</philosophy>

<scope_guardrail>
**CRÍTICO: Sem expansão de escopo.**

O limite da fase vem do ROADMAP.md e é FIXO. A discussão esclarece COMO implementar o que está no escopo, nunca SE devemos adicionar novas capacidades.

**Permitido (esclarecendo ambiguidade):**
- "Como os posts devem ser exibidos?" (layout, densidade, informações mostradas)
- "O que acontece no estado vazio?" (dentro da funcionalidade)
- "Pull to refresh ou manual?" (escolha de comportamento)

**Não permitido (expansão de escopo):**
- "Devemos também adicionar comentários?" (nova capacidade)
- "E quanto a busca/filtragem?" (nova capacidade)
- "Talvez incluir favoritos?" (nova capacidade)

**A heurística:** Isso esclarece como implementamos o que já está na fase, ou adiciona uma nova capacidade que poderia ser sua própria fase?

**Quando o usuário sugere expansão de escopo:**
```
"[Funcionalidade X] seria uma nova capacidade — isso é sua própria fase.
Quer que eu anote para o backlog do roteiro?

Por ora, vamos focar em [domínio da fase]."
```

Capture a ideia na seção "Ideias Adiadas". Não perca, não aja sobre ela.
</scope_guardrail>

<gray_area_identification>
Áreas cinzentas são **decisões de implementação que o usuário se importa** — coisas que poderiam ir de várias maneiras e mudariam o resultado.

**Como identificar áreas cinzentas:**

1. **Ler o objetivo da fase** do ROADMAP.md
2. **Entender o domínio** — Que tipo de coisa está sendo construída?
   - Algo que usuários VEEM → apresentação visual, interações, estados importam
   - Algo que usuários CHAMAM → contratos de interface, respostas, erros importam
   - Algo que usuários EXECUTAM → invocação, saída, modos de comportamento importam
   - Algo que usuários LEEM → estrutura, tom, profundidade, fluxo importam
   - Algo sendo ORGANIZADO → critérios, agrupamento, tratamento de exceções importam
3. **Gerar áreas cinzentas específicas da fase** — Não categorias genéricas, mas decisões concretas para ESTA fase

**Não use rótulos genéricos de categoria** (UI, UX, Comportamento). Gere áreas cinzentas específicas:

```
Fase: "Autenticação de usuário"
→ Tratamento de sessão, Respostas de erro, Política multi-dispositivo, Fluxo de recuperação

Fase: "Organizar biblioteca de fotos"
→ Critérios de agrupamento, Tratamento de duplicatas, Convenção de nomenclatura, Estrutura de pastas

Fase: "CLI para backups de banco"
→ Formato de saída, Design de flags, Relatório de progresso, Recuperação de erros

Fase: "Documentação de API"
→ Estrutura/navegação, Profundidade dos exemplos de código, Abordagem de versionamento, Elementos interativos
```

**A pergunta chave:** Quais decisões mudariam o resultado nas quais o usuário deveria opinar?

**Claude trata estas (não pergunte):**
- Detalhes técnicos de implementação
- Padrões de arquitetura
- Otimização de performance
- Escopo (roteiro define isso)
</gray_area_identification>

<answer_validation>
**IMPORTANTE: Validação de resposta** — Após toda chamada de prompt conversacional, verifique se a resposta está vazia ou somente espaços em branco. Se sim:
1. Tente a pergunta novamente com os mesmos parâmetros
2. Se ainda vazia, apresente as opções como lista numerada em texto simples e peça ao usuário para digitar o número da escolha
Nunca prossiga com uma resposta vazia.

**Modo texto (`workflow.text_mode: true` no config ou flag `--text`):**
Quando o modo texto estiver ativo, **não use prompt conversacional de forma alguma**. Em vez disso, apresente toda
pergunta como lista numerada em texto simples e peça ao usuário para digitar o número da escolha.
Isto é necessário para sessões remotas do Cursor (modo `/rc`) onde o Claude App
não pode encaminhar seleções de menu TUI de volta ao host.

Habilitar modo texto:
- Por sessão: passe flag `--text` para qualquer comando (ex: `/gsd-discutir-fase --text`)
- Por projeto: `gsd-tools config-set workflow.text_mode true`

Modo texto aplica-se a TODOS os workflows na sessão, não apenas ao discutir-fase.
</answer_validation>

<process>

**Caminho expresso disponível:** Se você já tem um PRD ou documento de critérios de aceitação, use `/gsd-planejar-fase {fase} --prd caminho/para/prd.md` para pular esta discussão e ir direto para o planejamento.

<step name="initialize" priority="first">
Número da fase do argumento (obrigatório).

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `has_verification`, `plan_count`, `roadmap_exists`, `planning_exists`.

**Se `phase_found` for false:**
```
Fase [X] não encontrada no roteiro.

Use /gsd-progresso ${GSD_WS} para ver fases disponíveis.
```
Sair do workflow.

**Se `phase_found` for true:** Continuar para check_existing.

**Modo auto** — Se `--auto` estiver presente em ARGUMENTS:
- Em `check_existing`: auto-selecionar "Pular" (se contexto existir) ou continuar sem perguntar (se sem contexto/planos)
- Em `present_gray_areas`: auto-selecionar TODAS as áreas cinzentas sem perguntar ao usuário
- Em `discuss_areas`: para cada pergunta de discussão, escolher a opção recomendada (primeira opção, ou a marcada como "recomendada") sem usar prompt conversacional
- Registrar cada escolha auto-selecionada inline para que o usuário possa revisar decisões no arquivo de contexto
- Após discussão completar, auto-avançar para o planejamento da fase (comportamento existente)
</step>

<step name="check_existing">
Verificar se CONTEXT.md já existe usando `has_context` do init.

```bash
ls ${phase_dir}/*-CONTEXT.md 2>/dev/null
```

**Se existir:**

**Se `--auto`:** Auto-selecionar "Atualizar" — carregar contexto existente e continuar para analyze_phase. Registrar: `[auto] Contexto existe — atualizando com decisões auto-selecionadas.`

**Caso contrário:** Use prompt conversacional:
- header: "Contexto"
- question: "Fase [X] já tem contexto. O que você quer fazer?"
- options:
  - "Atualizar" — Revisar e modificar contexto existente
  - "Visualizar" — Me mostre o que tem
  - "Pular" — Usar contexto existente como está

Se "Atualizar": Carregar existente, continuar para analyze_phase
Se "Visualizar": Exibir CONTEXT.md, depois oferecer atualizar/pular
Se "Pular": Sair do workflow

**Se não existir:**

Verificar `has_plans` e `plan_count` do init. **Se `has_plans` for true:**

**Se `--auto`:** Auto-selecionar "Continuar e replanejar depois". Registrar: `[auto] Planos existem — continuando com captura de contexto, replanejará depois.`

**Caso contrário:** Use prompt conversacional:
- header: "Planos existem"
- question: "Fase [X] já tem {plan_count} plano(s) criado(s) sem contexto do usuário. Suas decisões aqui não afetarão planos existentes a menos que replanejar."
- options:
  - "Continuar e replanejar depois" — Capturar contexto, depois executar /gsd-planejar-fase {X} ${GSD_WS} para replanejar
  - "Ver planos existentes" — Mostrar planos antes de decidir
  - "Cancelar" — Pular discutir-fase

Se "Continuar e replanejar depois": Continuar para analyze_phase.
Se "Ver planos existentes": Exibir arquivos dos planos, depois oferecer "Continuar" / "Cancelar".
Se "Cancelar": Sair do workflow.

**Se `has_plans` for false:** Continuar para load_prior_context.
</step>

<step name="load_prior_context">
Ler contexto de nível de projeto e fases anteriores para evitar re-perguntar questões já decididas e manter consistência.

**Passo 1: Ler arquivos de nível de projeto**
```bash
# Arquivos centrais do projeto
cat .planning/PROJECT.md 2>/dev/null
cat .planning/REQUIREMENTS.md 2>/dev/null
cat .planning/STATE.md 2>/dev/null
```

Extrair destes:
- **PROJECT.md** — Visão, princípios, inegociáveis, preferências do usuário
- **REQUIREMENTS.md** — Critérios de aceitação, restrições, obrigatórios vs desejáveis
- **STATE.md** — Progresso atual, quaisquer flags ou notas de sessão

**Passo 2: Ler todos os arquivos CONTEXT.md anteriores**
```bash
# Encontrar todos os arquivos CONTEXT.md de fases anteriores à atual
find .planning/phases -name "*-CONTEXT.md" 2>/dev/null | sort
```

Para cada CONTEXT.md onde número da fase < fase atual:
- Ler a seção `<decisions>` — estas são preferências travadas
- Ler `<specifics>` — referências particulares ou momentos "quero parecido com X"
- Notar padrões (ex: "usuário consistentemente prefere UI minimalista", "usuário rejeitou atalhos de tecla única")

**Passo 3: Construir contexto interno `<prior_decisions>`**

Estruturar a informação extraída:
```
<prior_decisions>
## Nível de Projeto
- [Princípio ou restrição chave do PROJECT.md]
- [Requisito que afeta esta fase do REQUIREMENTS.md]

## De Fases Anteriores
### Fase N: [Nome]
- [Decisão que pode ser relevante para a fase atual]
- [Preferência que estabelece um padrão]

### Fase M: [Nome]
- [Outra decisão relevante]
</prior_decisions>
```

**Uso em passos subsequentes:**
- `analyze_phase`: Pular áreas cinzentas já decididas em fases anteriores
- `present_gray_areas`: Anotar opções com decisões anteriores ("Você escolheu X na Fase 5")
- `discuss_areas`: Pré-preencher respostas ou sinalizar conflitos ("Isso contradiz a Fase 3 — manter igual ou diferente?")

**Se nenhum contexto anterior existir:** Continuar sem — isso é esperado para fases iniciais.
</step>

<step name="cross_reference_todos">
Verificar se alguma todo pendente é relevante para o escopo desta fase. Revela itens de backlog que poderiam ser perdidos.

**Carregar e buscar correspondências de todos:**
```bash
TODO_MATCHES=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" todo match-phase "${PHASE_NUMBER}")
```

Analisar JSON para: `todo_count`, `matches[]` (cada com `file`, `title`, `area`, `score`, `reasons`).

**Se `todo_count` for 0 ou `matches` estiver vazio:** Pular silenciosamente — sem desaceleração do workflow.

**Se correspondências encontradas:**

Apresentar todos correspondentes ao usuário. Mostrar cada correspondência com título, área e motivo:

```
📋 Encontradas {N} todo(s) pendente(s) que podem ser relevantes para a Fase {X}:

{Para cada correspondência:}
- **{title}** (área: {area}, relevância: {score}) — correspondeu por {reasons}
```

Use prompt conversacional (seleção múltipla / multiSelect) perguntando quais todos incluir no escopo desta fase:

```
Quais destes todos devem ser incorporados ao escopo da Fase {X}?
(Selecione os aplicáveis, ou nenhum para pular)
```

**Para todos selecionados (incorporados):**
- Armazenar internamente como `<folded_todos>` para inclusão na seção `<decisions>` do CONTEXT.md
- Estes se tornam itens de escopo adicionais que agentes posteriores (pesquisador, planejador) verão

**Para todos não selecionados (revisados mas não incorporados):**
- Armazenar internamente como `<reviewed_todos>` para inclusão na seção `<deferred>` do CONTEXT.md
- Isso previne que fases futuras re-apresentem os mesmos todos como "perdidos"

**Modo auto (`--auto`):** Incorporar todos os todos com pontuação >= 0.4 automaticamente. Registrar a seleção.
</step>

<step name="scout_codebase">
Varredura leve do código existente para informar identificação de áreas cinzentas e discussão. Usa ~10% de contexto — aceitável para uma sessão interativa.

**Passo 1: Verificar mapas de codebase existentes**
```bash
ls .planning/codebase/*.md 2>/dev/null
```

**Se mapas de codebase existirem:** Ler os mais relevantes (CONVENTIONS.md, STRUCTURE.md, STACK.md baseado no tipo de fase). Extrair:
- Componentes/hooks/utilitários reutilizáveis
- Padrões estabelecidos (gerenciamento de estado, estilização, busca de dados)
- Pontos de integração (onde novo código conectaria)

Pular para o Passo 3 abaixo.

**Passo 2: Se sem mapas de codebase, fazer grep direcionado**

Extrair termos-chave do objetivo da fase (ex: "feed" → "post", "card", "list"; "auth" → "login", "session", "token").

```bash
# Encontrar arquivos relacionados aos termos do objetivo da fase
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" 2>/dev/null | head -10

# Encontrar componentes/hooks existentes
ls src/components/ 2>/dev/null
ls src/hooks/ 2>/dev/null
ls src/lib/ src/utils/ 2>/dev/null
```

Ler os 3-5 arquivos mais relevantes para entender padrões existentes.

**Passo 3: Construir codebase_context interno**

Da varredura, identificar:
- **Ativos reutilizáveis** — componentes, hooks, utilitários existentes que poderiam ser usados nesta fase
- **Padrões estabelecidos** — como o codebase faz gerenciamento de estado, estilização, busca de dados
- **Pontos de integração** — onde novo código conectaria ao sistema existente (rotas, nav, providers)
- **Opções criativas** — abordagens que a arquitetura existente habilita ou restringe

Armazenar como `<codebase_context>` interno para uso em analyze_phase e present_gray_areas. Isto NÃO é escrito em arquivo — é usado apenas dentro desta sessão.
</step>

<step name="analyze_phase">
Analisar a fase para identificar áreas cinzentas que valem discutir. **Usar tanto `prior_decisions` quanto `codebase_context` para fundamentar a análise.**

**Ler a descrição da fase do ROADMAP.md e determinar:**

1. **Limite do domínio** — Que capacidade esta fase está entregando? Declare claramente.

1b. **Inicializar acumulador de refs canônicas** — Começar a construir a lista `<canonical_refs>` para o CONTEXT.md. Isto acumula ao longo de toda a discussão, não apenas neste passo.

   **Fonte 1 (agora):** Copiar `Canonical refs:` do ROADMAP.md para esta fase. Expandir cada para caminho relativo completo.
   **Fonte 2 (agora):** Verificar REQUIREMENTS.md e PROJECT.md para quaisquer specs/ADRs referenciados para esta fase.
   **Fonte 3 (scout_codebase):** Se código existente referencia docs (ex: comentários citando ADRs), adicionar esses.
   **Fonte 4 (discuss_areas):** Quando o usuário disser "leia X", "verifique Y", ou referencia qualquer doc/spec/ADR durante a discussão — adicionar imediatamente. Estas são frequentemente as refs MAIS importantes porque representam docs que o usuário especificamente quer que sejam seguidos.

   Esta lista é OBRIGATÓRIA no CONTEXT.md. Toda ref deve ter um caminho relativo completo para que agentes posteriores possam lê-la diretamente. Se nenhum doc externo existir, anote isso explicitamente.

2. **Verificar decisões anteriores** — Antes de gerar áreas cinzentas, verificar se alguma já foi decidida:
   - Escanear `<prior_decisions>` para escolhas relevantes (ex: "Ctrl+C apenas, sem atalhos de tecla única")
   - Estas são **pré-respondidas** — não re-perguntar a menos que esta fase tenha necessidades conflitantes
   - Anotar decisões anteriores aplicáveis para uso na apresentação

3. **Áreas cinzentas por categoria** — Para cada categoria relevante (UI, UX, Comportamento, Estados Vazios, Conteúdo), identificar 1-2 ambiguidades específicas que mudariam a implementação. **Anotar com contexto de código onde relevante** (ex: "Você já tem um componente Card" ou "Sem padrão existente para isso").

4. **Avaliação de pulo** — Se não existirem áreas cinzentas significativas (infraestrutura pura, implementação clara, ou tudo já decidido em fases anteriores), a fase pode não precisar de discussão.

**Detecção de Modo Consultor:**

Verificar se o modo consultor deve ativar:

1. Verificar USER-PROFILE.md:
   ```bash
   PROFILE_PATH="D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md"
   ```
   ADVISOR_MODE = arquivo existe em PROFILE_PATH → true, caso contrário → false

2. Se ADVISOR_MODE for true, resolver nível de calibração vendor_philosophy:
   - Prioridade 1: Ler config.json > preferences.vendor_philosophy (override no nível do projeto)
   - Prioridade 2: Ler avaliação de Escolhas de fornecedor / Filosofia (Vendor Choices/Philosophy) do USER-PROFILE.md (global)
   - Prioridade 3: Padrão para "standard" se nenhum tiver valor ou valor for UNSCORED

   Mapear para nível de calibração:
   - conservative OU thorough-evaluator → full_maturity
   - opinionated → minimal_decisive
   - pragmatic-fast OU qualquer outro valor OU vazio → standard

3. Resolver modelo para agentes consultores:
   ```bash
   ADVISOR_MODEL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-pesquisador-consultor --raw)
   ```

Se ADVISOR_MODE for false, pular todos os passos específicos de consultor — workflow prossegue com fluxo conversacional existente inalterado.

**Produzir sua análise internamente, depois apresentar ao usuário.**

Exemplo de análise para fase "Feed de Posts" (com contexto de código e anterior):
```
Domínio: Exibindo posts de usuários seguidos
Existente: Componente Card (src/components/ui/Card.tsx), hook useInfiniteQuery, Tailwind CSS
Decisões anteriores: "UI minimalista preferida" (Fase 2), "Sem paginação — sempre scroll infinito" (Fase 4)
Áreas cinzentas:
- UI: Estilo de layout (cards vs timeline vs grid) — Componente Card existe com variantes shadow/rounded
- UI: Densidade de informação (posts completos vs prévias) — sem padrões de densidade existentes
- Comportamento: Padrão de carregamento — JÁ DECIDIDO: scroll infinito (Fase 4)
- Estado Vazio: O que mostrar quando não há posts — componente EmptyState existe em ui/
- Conteúdo: Quais metadados exibir (hora, autor, contagem de reações)
```
</step>

<step name="present_gray_areas">
Apresentar o limite do domínio, decisões anteriores e áreas cinzentas ao usuário.

**Primeiro, declarar o limite e quaisquer decisões anteriores que se aplicam:**
```
Fase [X]: [Nome]
Domínio: [O que esta fase entrega — da sua análise]

Vamos esclarecer COMO implementar isto.
(Novas capacidades pertencem a outras fases.)

[Se decisões anteriores se aplicam:]
**Carregadas de fases anteriores:**
- [Decisão da Fase N que se aplica aqui]
- [Decisão da Fase M que se aplica aqui]
```

**Se `--auto`:** Auto-selecionar TODAS as áreas cinzentas. Registrar: `[auto] Selecionadas todas as áreas cinzentas: [listar nomes das áreas].` Pular o prompt conversacional abaixo e continuar diretamente para discuss_areas com todas as áreas selecionadas.

**Caso contrário, use prompt conversacional (seleção múltipla / multiSelect: true):**
- header: "Discutir"
- question: "Quais áreas você quer discutir para [nome da fase]?"
- options: Gerar 3-4 áreas cinzentas específicas da fase, cada com:
  - "[Área específica]" (label) — concreta, não genérica
  - [1-2 perguntas que isto cobre + anotação de contexto de código] (description)
  - **Destacar a escolha recomendada com breve explicação do motivo**

**Anotações de decisão anterior:** Quando uma área cinzenta já foi decidida em fase anterior, anotar:
```
☐ Atalhos de saída — Como os usuários devem sair?
  (Você decidiu "Ctrl+C apenas, sem atalhos de tecla única" na Fase 5 — revisitar ou manter?)
```

**Anotações de contexto de código:** Quando a exploração encontrou código existente relevante, anotar na descrição da área cinzenta:
```
☐ Estilo de layout — Cards vs lista vs timeline?
  (Você já tem um componente Card com variantes shadow/rounded. Reutilizar mantém o app consistente.)
```

**Combinando ambos:** Quando tanto decisões anteriores quanto contexto de código se aplicam:
```
☐ Comportamento de carregamento — Scroll infinito ou paginação?
  (Você escolheu scroll infinito na Fase 4. Hook useInfiniteQuery já configurado.)
```

**NÃO incluir opção "pular" ou "você decide".** O usuário executou este comando para discutir — dê escolhas reais.

**Exemplos por domínio (com contexto de código):**

Para "Feed de Posts" (funcionalidade visual):
```
☐ Estilo de layout — Cards vs lista vs timeline? (Componente Card existe com variantes)
☐ Comportamento de carregamento — Scroll infinito ou paginação? (Hook useInfiniteQuery disponível)
☐ Ordenação de conteúdo — Cronológico, algorítmico ou escolha do usuário?
☐ Metadados dos posts — Quais informações por post? Timestamps, reações, autor?
```

Para "CLI de backup de banco" (ferramenta de linha de comando):
```
☐ Formato de saída — JSON, tabela ou texto simples? Níveis de verbosidade?
☐ Design de flags — Flags curtas, longas ou ambas? Obrigatórias vs opcionais?
☐ Relatório de progresso — Silencioso, barra de progresso ou log verboso?
☐ Recuperação de erros — Falhar rápido, tentar novamente ou perguntar ação?
```

Para "Organizar biblioteca de fotos" (tarefa de organização):
```
☐ Critérios de agrupamento — Por data, localização, rostos ou eventos?
☐ Tratamento de duplicatas — Manter melhor, manter todas ou perguntar cada vez?
☐ Convenção de nomenclatura — Nomes originais, datas ou descritivos?
☐ Estrutura de pastas — Plana, aninhada por ano ou por categoria?
```

Continuar para discuss_areas com áreas selecionadas (ou advisor_research se ADVISOR_MODE for true).
</step>

<step name="advisor_research">
**Pesquisa de Consultor** (somente quando ADVISOR_MODE for true)

Após o usuário selecionar áreas cinzentas em present_gray_areas, iniciar agentes de pesquisa paralelos.

1. Exibir status breve: "Pesquisando {N} áreas..."

2. Para CADA área cinzenta selecionada pelo usuário, iniciar um Task() em paralelo:

   Task(
     prompt="Primeiro, leia @D:/projetos/Estudo/devsquad/.cursor/agents/gsd-pesquisador-consultor.md para seu papel e instruções.

     <gray_area>{area_name}: {area_description da identificação de área cinzenta}</gray_area>
     <phase_context>{objetivo e descrição da fase do ROADMAP.md}</phase_context>
     <project_context>{nome e breve descrição do projeto do PROJECT.md}</project_context>
     <calibration_tier>{nível de calibração resolvido: full_maturity | standard | minimal_decisive}</calibration_tier>

     Pesquise esta área cinzenta e retorne uma tabela comparativa estruturada com justificativa.",
     subagent_type="generalPurpose",
     model="{ADVISOR_MODEL}",
     description="Pesquisar: {area_name}"
   )

   Todas as chamadas Task() iniciam simultaneamente — NÃO esperar uma antes de iniciar a próxima.

3. Após TODOS os agentes retornarem, SINTETIZAR resultados antes de apresentar:
   Para cada retorno de agente:
   a. Analisar a tabela comparativa markdown e parágrafo de justificativa
   b. Verificar que todas as 5 colunas estão presentes (Opção | Prós | Contras | Complexidade | Recomendação) — preencher colunas faltantes ao invés de mostrar tabela quebrada
   c. Verificar que contagem de opções corresponde ao nível de calibração:
      - full_maturity: 3-5 opções aceitáveis
      - standard: 2-4 opções aceitáveis
      - minimal_decisive: 1-2 opções aceitáveis
      Se agente retornou demais, cortar menos viáveis. Se poucas, aceitar como está.
   d. Reescrever parágrafo de justificativa para incorporar contexto do projeto e discussão em andamento que o agente não teve acesso
   e. Se agente retornou apenas 1 opção, converter de formato tabela para recomendação direta: "Abordagem padrão para {area}: {opção}. {justificativa}"

4. Armazenar tabelas sintetizadas para uso em discuss_areas.

**Se ADVISOR_MODE for false:** Pular este passo inteiramente — prosseguir diretamente de present_gray_areas para discuss_areas.
</step>

<step name="discuss_areas">
Discutir cada área selecionada com o usuário. Fluxo depende do modo consultor.

**Se ADVISOR_MODE for true:**

Fluxo de discussão tabela-primeiro — apresentar tabelas comparativas baseadas em pesquisa, depois capturar escolhas do usuário.

**Para cada área selecionada:**

1. **Apresentar a tabela comparativa sintetizada + parágrafo de justificativa** (do passo advisor_research)

2. **Usar prompt conversacional:**
   - header: "{area_name}"
   - question: "Qual abordagem para {area_name}?"
   - options: Extrair da coluna Opção da tabela (prompt conversacional adiciona "Outro" automaticamente)

3. **Registrar a seleção do usuário:**
   - Se usuário escolhe das opções da tabela → registrar como decisão travada para aquela área
   - Se usuário escolhe "Outro" → receber input, refletir de volta para confirmação, registrar

4. **Após registrar a escolha, Claude decide se perguntas de acompanhamento são necessárias:**
   - Se a escolha tem ambiguidade que afetaria planejamento posterior → perguntar 1-2 perguntas de acompanhamento direcionadas usando prompt conversacional
   - Se a escolha é clara e auto-contida → mover para próxima área
   - NÃO fazer as 4 perguntas padrão — a tabela já forneceu o contexto

5. **Após todas as áreas processadas:**
   - header: "Concluído"
   - question: "Isso cobre [listar áreas]. Pronto para criar contexto?"
   - options: "Criar contexto" / "Revisitar uma área"

**Tratamento de expansão de escopo (modo consultor):**
Se o usuário mencionar algo fora do domínio da fase:
```
"[Funcionalidade] parece ser uma nova capacidade — isso pertence à sua própria fase.
Vou anotar como ideia adiada.

Voltando a [área atual]: [retornar à pergunta atual]"
```

Rastrear ideias adiadas internamente.

---

**Se ADVISOR_MODE for false:**

Para cada área selecionada, conduzir um loop de discussão focada.

**Modo pesquisa-antes-das-perguntas:** Verificar se `workflow.research_before_questions` está habilitado no config (do contexto de init ou `.planning/config.json`). Quando habilitado, antes de apresentar perguntas para cada área:
1. Fazer uma breve pesquisa web por melhores práticas relacionadas ao tópico da área
2. Resumir as principais descobertas em 2-3 pontos
3. Apresentar a pesquisa junto com a pergunta para que o usuário possa tomar uma decisão mais informada

Exemplo com pesquisa habilitada:
```
Vamos falar sobre [Estratégia de Autenticação].

📊 Pesquisa de melhores práticas:
• OAuth 2.0 + PKCE é o padrão atual para SPAs (substitui fluxo implícito)
• Tokens de sessão com cookies httpOnly preferidos sobre localStorage para proteção XSS
• Considere suporte a passkey/WebAuthn — adoção está acelerando em 2025-2026

Com esse contexto: Como os usuários devem autenticar?
```

Quando desabilitado (padrão), pular a pesquisa e apresentar perguntas diretamente como antes.

**Suporte a modo texto:** Analisar `--text` opcional de `{{GSD_ARGS}}`.
- Aceitar flag `--text` OU ler `workflow.text_mode` do config (do contexto de init)
- Quando ativo, substituir TODAS as chamadas `prompt conversacional` por listas numeradas em texto simples
- Usuário digita um número para selecionar, ou digita texto livre para "Outro"
- Isto é necessário para sessões remotas do Cursor (modo `/rc`) onde menus TUI
  não funcionam através do Claude App

**Suporte ao modo lote (`--batch`):** Analisar `--batch` opcional de `{{GSD_ARGS}}`.
- Aceitar `--batch`, `--batch=N` ou `--batch N`

**Suporte a modo análise:** Analisar `--analyze` opcional de `{{GSD_ARGS}}`.
Quando `--analyze` estiver ativo, antes de apresentar cada pergunta (ou grupo de perguntas em modo lote), fornecer uma breve **análise de compensações (trade-offs)** para a decisão:
- 2-3 opções com prós/contras baseados no contexto do codebase e padrões comuns
- Uma abordagem recomendada com raciocínio
- Armadilhas conhecidas ou restrições de fases anteriores

Exemplo com `--analyze`:
```
**Análise de compensações (trade-offs): Estratégia de autenticação**

| Abordagem | Prós | Contras |
|-----------|------|---------|
| Cookies de sessão | Simples, httpOnly previne XSS | Requer proteção CSRF, sessões fixas |
| JWT (stateless) | Escalável, sem estado no servidor | Tamanho do token, complexidade de revogação |
| OAuth 2.0 + PKCE | Padrão da indústria para SPAs | Mais setup, UX de fluxo de redirect |

💡 Recomendado: OAuth 2.0 + PKCE — seu app tem login social nos requisitos (REQ-04) e isso alinha com o setup NextAuth existente em `src/lib/auth.ts`.

Como os usuários devem autenticar?
```

Isso dá ao usuário contexto para tomar decisões informadas sem prompting extra. Quando `--analyze` estiver ausente, apresentar perguntas diretamente como antes.

- Aceitar `--batch`, `--batch=N` ou `--batch N`
- Padrão para 4 perguntas por lote quando nenhum número fornecido
- Limitar tamanhos explícitos a 2-5 para que um lote permaneça respondível
- Se `--batch` estiver ausente, manter o fluxo existente de uma-pergunta-por-vez

**Filosofia:** permanecer adaptativo, mas deixar o usuário escolher o ritmo.
- Modo padrão: 4 turnos de pergunta única, depois verificar se deve continuar
- Modo `--batch`: 1 turno agrupado com 2-5 perguntas numeradas, depois verificar se deve continuar

Cada resposta (ou conjunto de respostas, em modo lote) deve revelar a próxima pergunta ou o próximo lote.

**Modo auto (`--auto`):** Para cada área, Claude seleciona a opção recomendada (primeira opção, ou a explicitamente marcada "recomendada") para toda pergunta sem usar prompt conversacional. Registrar cada escolha auto-selecionada:
```
[auto] [Área] — Q: "[texto da pergunta]" → Selecionado: "[opção escolhida]" (padrão recomendado)
```
Após todas as áreas serem auto-resolvidas, pular o prompt "Explorar mais áreas cinzentas" e prosseguir diretamente para write_context.

**Modo interativo (sem `--auto`):**

**Para cada área:**

1. **Anunciar a área:**
   ```
   Vamos falar sobre [Área].
   ```

2. **Fazer perguntas usando o ritmo selecionado:**

   **Padrão (sem `--batch`): Fazer 4 perguntas usando prompt conversacional**
   - header: "[Área]" (máx 12 caracteres — abreviar se necessário)
   - question: Decisão específica para esta área
   - options: 2-3 escolhas concretas (prompt conversacional adiciona "Outro" automaticamente), com a escolha recomendada destacada e breve explicação do motivo
   - **Anotar opções com contexto de código** quando relevante:
     ```
     "Como os posts devem ser exibidos?"
     - Cards (reutiliza componente Card existente — consistente com Mensagens)
     - Lista (mais simples, seria um novo padrão)
     - Timeline (precisa de novo componente Timeline — nenhum existe ainda)
     ```
   - Incluir "Você decide" como opção quando razoável — captura discrição do Claude
   - **Context7 para escolhas de biblioteca:** Quando uma área cinzenta envolve seleção de biblioteca (ex: "links mágicos" → consultar docs next-auth) ou decisões de abordagem de API, usar ferramentas `mcp__context7__*` para buscar documentação atual e informar as opções. Não usar Context7 para toda pergunta — apenas quando conhecimento específico de biblioteca melhora as opções.

   **Modo lote (`--batch`): Fazer 2-5 perguntas numeradas em um turno de texto simples**
   - Agrupar perguntas intimamente relacionadas para a área atual em uma única mensagem
   - Manter cada pergunta concreta e respondível em uma resposta
   - Quando opções são úteis, incluir escolhas inline curtas por pergunta ao invés de um prompt conversacional separado para cada item
   - Após o usuário responder, refletir as decisões capturadas, notar itens não respondidos, e perguntar apenas o acompanhamento mínimo necessário antes de prosseguir
   - Preservar adaptatividade entre lotes: usar o conjunto completo de respostas para decidir o próximo lote ou se a área está suficientemente clara

3. **Após o conjunto atual de perguntas, verificar:**
   - header: "[Área]" (máx 12 caracteres)
   - question: "Mais perguntas sobre [área], ou mover para a próxima? (Restantes: [listar outras áreas não visitadas])"
   - options: "Mais perguntas" / "Próxima área"

   Ao construir o texto da pergunta, listar as áreas não visitadas restantes para que o usuário saiba o que está por vir. Por exemplo: "Mais perguntas sobre Layout, ou mover para a próxima? (Restantes: Comportamento de carregamento, Ordenação de conteúdo)"

   Se "Mais perguntas" → fazer mais 4 perguntas únicas, ou outro lote de 2-5 perguntas quando `--batch` estiver ativo, depois verificar novamente
   Se "Próxima área" → prosseguir para a próxima área selecionada
   Se "Outro" (texto livre) → interpretar intenção: frases de continuação ("conversar mais", "continue", "sim", "mais") mapeiam para "Mais perguntas"; frases de avanço ("pronto", "seguir em frente", "próxima", "pular") mapeiam para "Próxima área". Se ambíguo, perguntar: "Continuar com mais perguntas sobre [área], ou mover para a próxima área?"

4. **Após todas as áreas inicialmente selecionadas completarem:**
   - Resumir o que foi capturado da discussão até agora
   - prompt conversacional:
     - header: "Concluído"
     - question: "Discutimos [listar áreas]. Quais áreas cinzentas permanecem incertas?"
     - options: "Explorar mais áreas cinzentas" / "Pronto para contexto"
   - Se "Explorar mais áreas cinzentas":
     - Identificar 2-4 áreas cinzentas adicionais baseadas no que foi aprendido
     - Retornar à lógica de present_gray_areas com estas novas áreas
     - Loop: discutir novas áreas, depois perguntar novamente
   - Se "Pronto para contexto": Prosseguir para write_context

**Acumulação de refs canônicas durante discussão:**
Quando o usuário referencia um doc, spec ou ADR durante qualquer resposta — ex: "leia adr-014", "verifique a spec do MCP", "conforme browse-spec.md" — imediatamente:
1. Ler o doc referenciado (ou confirmar que existe)
2. Adicionar ao acumulador de refs canônicas com caminho relativo completo
3. Usar o que aprendeu do doc para informar perguntas subsequentes

Estes docs referenciados pelo usuário são frequentemente MAIS importantes que refs do ROADMAP.md porque representam docs que o usuário especificamente quer que agentes posteriores sigam. Nunca descartá-los.

**Design de perguntas:**
- Opções devem ser concretas, não abstratas ("Cards" não "Opção A")
- Cada resposta deve informar a próxima pergunta ou o próximo lote
- Se o usuário escolher "Outro" para fornecer input em texto livre (ex: "deixe eu descrever", "algo diferente", ou uma resposta aberta), faça seu acompanhamento como texto simples — NÃO outro prompt conversacional. Espere que digitem no prompt normal, depois reflita o input de volta e confirme antes de retomar prompt conversacional ou o próximo lote numerado.

**Tratamento de expansão de escopo:**
Se o usuário mencionar algo fora do domínio da fase:
```
"[Funcionalidade] parece ser uma nova capacidade — isso pertence à sua própria fase.
Vou anotar como ideia adiada.

Voltando a [área atual]: [retornar à pergunta atual]"
```

Rastrear ideias adiadas internamente.

**Rastrear dados do log de discussão internamente:**
Para cada pergunta feita, acumular:
- Nome da área
- Todas as opções apresentadas (label + descrição)
- Qual opção o usuário selecionou (ou sua resposta em texto livre)
- Quaisquer notas de acompanhamento ou esclarecimentos que o usuário forneceu
Estes dados são usados para gerar DISCUSSION-LOG.md no passo `write_context`.
</step>

<step name="write_context">
Criar CONTEXT.md capturando decisões tomadas.

**Também gerar DISCUSSION-LOG.md** — um rastro de auditoria completo das perguntas e respostas (Q&A) do discutir-fase. (Modelo de referência: template `log-discussao.md`.)
Este arquivo é apenas para referência humana (auditorias de software, revisões de conformidade). NÃO é
consumido por agentes posteriores (pesquisador, planejador, executor).

**Encontrar ou criar diretório da fase:**

Usar valores do init: `phase_dir`, `phase_slug`, `padded_phase`.

Se `phase_dir` for null (fase existe no roteiro mas sem diretório):
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```

**Localização do arquivo:** `${phase_dir}/${padded_phase}-CONTEXT.md`

**Estruturar o conteúdo pelo que foi discutido:**

```markdown
# Fase [X]: [Nome] - Contexto

**Coletado:** [data]
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

[Declaração clara do que esta fase entrega — a âncora de escopo]

</domain>

<decisions>
## Decisões de Implementação

### [Categoria 1 que foi discutida]
- **D-01:** [Decisão ou preferência capturada]
- **D-02:** [Outra decisão se aplicável]

### [Categoria 2 que foi discutida]
- **D-03:** [Decisão ou preferência capturada]

### Discrição do Claude
[Áreas onde o usuário disse "você decide" — notar que Claude tem flexibilidade aqui]

### Todos Incorporados
[Se algum todo foi incorporado ao escopo do passo cross_reference_todos, listá-los aqui.
Cada entrada deve incluir o título do todo, problema original e como se encaixa no escopo desta fase.
Se nenhum todo foi incorporado: omitir esta subseção inteiramente.]

</decisions>

<canonical_refs>
## Referências Canônicas

**Agentes posteriores DEVEM ler estes antes de planejar ou implementar.**

[Seção OBRIGATÓRIA. Escrever a lista COMPLETA de refs canônicas acumulada aqui.
Fontes: refs do ROADMAP.md + refs do REQUIREMENTS.md + docs referenciados pelo usuário durante
a discussão + quaisquer docs descobertos durante exploração do codebase. Agrupar por área temática.
Toda entrada precisa de caminho relativo completo — não apenas um nome.]

### [Área temática 1]
- `caminho/para/adr-ou-spec.md` — [O que decide/define que é relevante]
- `caminho/para/doc.md` §N — [Referência de seção específica]

### [Área temática 2]
- `caminho/para/feature-doc.md` — [O que este doc define]

[Se sem specs externas: "Sem specs externas — requisitos completamente capturados nas decisões acima"]

</canonical_refs>

<code_context>
## Insights do Código Existente

### Ativos Reutilizáveis
- [Componente/hook/utilitário]: [Como poderia ser usado nesta fase]

### Padrões Estabelecidos
- [Padrão]: [Como restringe/habilita esta fase]

### Pontos de Integração
- [Onde novo código conecta ao sistema existente]

</code_context>

<specifics>
## Ideias Específicas

[Quaisquer referências particulares, exemplos ou momentos "quero parecido com X" da discussão]

[Se nenhum: "Sem requisitos específicos — aberto a abordagens padrão"]

</specifics>

<deferred>
## Ideias Adiadas

[Ideias que surgiram mas pertencem a outras fases. Não perder.]

### Todos Revisados (não incorporados)
[Se algum todo foi revisado em cross_reference_todos mas não incorporado ao escopo,
listá-los aqui para que fases futuras saibam que foram considerados.
Cada entrada: título do todo + motivo do adiamento (fora do escopo, pertence à Fase Y, etc.)
Se nenhum todo revisado-mas-adiado: omitir esta subseção inteiramente.]

[Se nenhum: "Nenhum — discussão permaneceu dentro do escopo da fase"]

</deferred>

---

*Fase: XX-nome*
*Contexto coletado: [data]*
```

Escrever arquivo.
</step>

<step name="confirm_creation">
Apresentar resumo e próximos passos:

```
Criado: .planning/phases/${PADDED_PHASE}-${SLUG}/${PADDED_PHASE}-CONTEXT.md

## Decisões Capturadas

### [Categoria]
- [Decisão chave]

### [Categoria]
- [Decisão chave]

[Se ideias adiadas existirem:]
## Anotado para Depois
- [Ideia adiada] — fase futura

---

## ▶ Próximo

**Fase ${PHASE}: [Nome]** — [Objetivo do ROADMAP.md]

`/gsd-planejar-fase ${PHASE} ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-planejar-fase ${PHASE} --skip-research ${GSD_WS}` — planejar sem pesquisa
- `/gsd-fase-ui ${PHASE} ${GSD_WS}` — gerar contrato de design UI antes de planejar (se fase tem trabalho frontend)
- Revisar/editar CONTEXT.md antes de continuar

---
```
</step>

<step name="git_commit">
**Escrever DISCUSSION-LOG.md antes de commitar:**

**Localização do arquivo:** `${phase_dir}/${padded_phase}-DISCUSSION-LOG.md`

```markdown
# Fase [X]: [Nome] - Log de Discussão

> **Somente rastro de auditoria.** Não usar como entrada para agentes de planejamento, pesquisa ou execução.
> Decisões estão capturadas no CONTEXT.md — este log preserva as alternativas consideradas.

**Data:** [data ISO]
**Fase:** [número da fase]-[nome da fase]
**Áreas discutidas:** [lista separada por vírgula]

---

[Para cada área cinzenta discutida:]

## [Nome da Área]

| Opção | Descrição | Selecionada |
|-------|-----------|-------------|
| [Opção 1] | [Descrição do prompt conversacional] | |
| [Opção 2] | [Descrição] | ✓ |
| [Opção 3] | [Descrição] | |

**Escolha do usuário:** [Opção selecionada ou resposta em texto livre]
**Notas:** [Quaisquer esclarecimentos, contexto de acompanhamento ou justificativa que o usuário forneceu]

---

[Repetir para cada área]

## Discrição do Claude

[Listar áreas onde o usuário disse "você decide" ou delegou ao Claude]

## Ideias Adiadas

[Ideias mencionadas durante a discussão que foram anotadas para fases futuras]
```

Escrever arquivo.

Commitar contexto da fase e log de discussão:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): capture phase context" --files "${phase_dir}/${padded_phase}-CONTEXT.md" "${phase_dir}/${padded_phase}-DISCUSSION-LOG.md"
```

Confirmar: "Commitado: docs(${padded_phase}): capture phase context"
</step>

<step name="update_state">
Atualizar STATE.md com informações da sessão:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Phase ${PHASE} context gathered" \
  --resume-file "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Commitar STATE.md:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(state): record phase ${PHASE} context session" --files .planning/STATE.md
```
</step>

<step name="auto_advance">
Verificar gatilho de auto-avanço:

1. Analisar flag `--auto` de {{GSD_ARGS}}
2. **Sincronizar flag de cadeia com intenção** — se o usuário invocou manualmente (sem `--auto`), limpar a flag efêmera de cadeia de qualquer cadeia `--auto` interrompida anteriormente. Isto NÃO toca em `workflow.auto_advance` (a preferência persistente do usuário nas configurações):
   ```bash
   if [[ ! "{{GSD_ARGS}}" =~ --auto ]]; then
     node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. Ler tanto a flag de cadeia quanto a preferência do usuário:
   ```bash
   AUTO_CHAIN=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se flag `--auto` presente E `AUTO_CHAIN` não for true:** Persistir flag de cadeia no config (trata uso direto de `--auto` sem new-project):
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

**Se flag `--auto` presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true:**

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTO-AVANÇANDO PARA PLANEJAR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Contexto capturado. Iniciando planejamento da fase...
```

Iniciar o planejamento da fase usando a ferramenta Skill para evitar sessões Task aninhadas (que causam congelamentos de runtime devido a aninhamento profundo de agentes — veja #686):
```
Skill(skill="gsd-plan-phase", args="${PHASE} --auto ${GSD_WS}")
```

Isso mantém a cadeia de auto-avanço plana — discutir, planejar e executar rodam todos no mesmo nível de aninhamento, em vez de gerar agentes Task cada vez mais profundos.

**Tratar retorno do planejamento da fase:**
- **PHASE COMPLETE** → Cadeia completa teve sucesso. Exibir:
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► FASE ${PHASE} COMPLETA
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Pipeline de auto-avanço finalizado: discutir → planejar → executar

  Próximo: /gsd-discutir-fase ${NEXT_PHASE} --auto ${GSD_WS}
  <sub>/clear primeiro → janela de contexto limpa</sub>
  ```
- **PLANNING COMPLETE** → Planejamento concluído, execução não completou:
  ```
  Auto-avanço parcial: Planejamento completo, execução não finalizou.
  Continuar: /gsd-executar-fase ${PHASE} ${GSD_WS}
  ```
- **PLANNING INCONCLUSIVE / CHECKPOINT** → Parar cadeia:
  ```
  Auto-avanço parado: Planejamento precisa de input.
  Continuar: /gsd-planejar-fase ${PHASE} ${GSD_WS}
  ```
- **GAPS FOUND** → Parar cadeia:
  ```
  Auto-avanço parado: Lacunas encontradas durante execução.
  Continuar: /gsd-planejar-fase ${PHASE} --gaps ${GSD_WS}
  ```

**Se nem `--auto` nem config habilitado:**
Rotear para o passo `confirm_creation` (comportamento existente — mostrar próximos passos manuais).
</step>

</process>

<success_criteria>
- Fase validada contra o roteiro
- Contexto anterior carregado (PROJECT.md, REQUIREMENTS.md, STATE.md, arquivos CONTEXT.md anteriores)
- Perguntas já decididas não são re-feitas (carregadas de fases anteriores)
- Codebase explorado para ativos reutilizáveis, padrões e pontos de integração
- Áreas cinzentas identificadas através de análise inteligente com anotações de código e decisões anteriores
- Usuário selecionou quais áreas discutir
- Cada área selecionada explorada até o usuário ficar satisfeito (com opções informadas por código e decisões anteriores)
- Expansão de escopo redirecionada para ideias adiadas
- CONTEXT.md captura decisões reais, não visão vaga
- CONTEXT.md inclui seção canonical_refs com caminhos completos de arquivo para cada spec/ADR/doc que agentes posteriores precisam (OBRIGATÓRIO — nunca omitir)
- CONTEXT.md inclui seção code_context com ativos reutilizáveis e padrões
- Ideias adiadas preservadas para fases futuras
- STATE.md atualizado com informações da sessão
- Usuário sabe os próximos passos
</success_criteria>
