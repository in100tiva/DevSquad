<purpose>
Inicializar um novo projeto através de fluxo unificado: questionamento, pesquisa (opcional), requisitos, roteiro. Este é o momento de maior alavancagem em qualquer projeto — questionamento profundo aqui significa melhores planos, melhor execução, melhores resultados. Um único fluxo leva você da ideia até pronto-para-planejar.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<auto_mode>

## Detecção de Modo Auto

Verifique se a flag `--auto` está presente em {{GSD_ARGS}}.

**Se modo auto:**

- Pular oferta de mapeamento brownfield (assumir greenfield)
- Pular questionamento profundo (extrair contexto do documento fornecido)
- Config: modo YOLO é implícito (pular essa pergunta), mas perguntar granularidade/git/agentes PRIMEIRO (Passo 2a)
- Após config: executar Passos 6-9 automaticamente com padrões inteligentes:
  - Pesquisa: Sempre sim
  - Requisitos: Incluir todos os itens essenciais + funcionalidades do documento fornecido
  - Aprovação de requisitos: Auto-aprovar
  - Aprovação do roteiro: Auto-aprovar

**Requisito de documento:**
Modo auto requer um documento de ideia — seja:

- Referência de arquivo: `/gsd-novo-projeto --auto @prd.md`
- Texto colado/escrito no prompt

Se nenhum conteúdo de documento fornecido, erro:

```
Erro: --auto requer um documento de ideia.

Uso:
  /gsd-novo-projeto --auto @sua-ideia.md
  /gsd-novo-projeto --auto [cole ou escreva sua ideia aqui]

O documento deve descrever o que você quer construir.
```

</auto_mode>

<process>

## 1. Configuração

**PRIMEIRO PASSO OBRIGATÓRIO — Execute estas verificações antes de QUALQUER interação com o usuário:**

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init new-project)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `researcher_model`, `synthesizer_model`, `roadmapper_model`, `commit_docs`, `project_exists`, `has_codebase_map`, `planning_exists`, `has_existing_code`, `has_package_file`, `is_brownfield`, `needs_codebase_map`, `has_git`, `project_path`.

**Se `project_exists` for true:** Erro — projeto já inicializado. Use `/gsd-progresso`.

**Se `has_git` for false:** Inicializar git:

```bash
git init
```

## 2. Oferta Brownfield

**Se modo auto:** Pular para o Passo 4 (assumir greenfield, sintetizar PROJECT.md a partir do documento fornecido).

**Se `needs_codebase_map` for true** (do init — código existente detectado, mas sem mapa da base de código):

Use um prompt conversacional:

- header: "Base de código"
- question: "Detectei código existente neste diretório. Quer mapear a base de código primeiro?"
- options:
  - "Mapear a base de código primeiro" — Executar /gsd-mapear-codigo para entender a arquitetura existente (Recomendado)
  - "Pular mapeamento" — Prosseguir com a inicialização do projeto

**Se "Mapear a base de código primeiro":**

```
Execute `/gsd-mapear-codigo` primeiro, depois retorne a `/gsd-novo-projeto`
```

Sair do comando.

**Se "Pular mapeamento" OU `needs_codebase_map` for false:** Continuar para o Passo 3.

## 2a. Config do Modo Auto (somente modo auto)

**Se modo auto:** Coletar configurações antecipadamente antes de processar o documento de ideia.

Modo YOLO é implícito (auto = YOLO). Perguntar as configurações restantes:

**Rodada 1 — Configurações principais (3 perguntas, sem pergunta de Modo):**

```
prompt conversacional([
  {
    header: "Granularidade",
    question: "Quão finamente o escopo deve ser dividido em fases?",
    multiSelect: false,
    options: [
      { label: "Grossa (Recomendado)", description: "Menos fases, mais amplas (3-5 fases, 1-3 planos cada)" },
      { label: "Padrão", description: "Tamanho equilibrado de fases (5-8 fases, 3-5 planos cada)" },
      { label: "Fina", description: "Muitas fases focadas (8-12 fases, 5-10 planos cada)" }
    ]
  },
  {
    header: "Execução",
    question: "Executar planos em paralelo?",
    multiSelect: false,
    options: [
      { label: "Paralelo (Recomendado)", description: "Planos independentes executam simultaneamente" },
      { label: "Sequencial", description: "Um plano por vez" }
    ]
  },
  {
    header: "Rastreamento Git",
    question: "Fazer commit dos docs de planejamento no git?",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Docs de planejamento rastreados no controle de versão" },
      { label: "Não", description: "Manter .planning/ somente local (adicionar ao .gitignore)" }
    ]
  }
])
```

**Rodada 2 — Agentes do fluxo de trabalho (igual ao Passo 5):**

```
prompt conversacional([
  {
    header: "Pesquisa",
    question: "Pesquisar antes de planejar cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Investigar domínio, encontrar padrões, identificar armadilhas" },
      { label: "Não", description: "Planejar diretamente a partir dos requisitos" }
    ]
  },
  {
    header: "Verificação de Plano",
    question: "Verificar se os planos alcançarão seus objetivos? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Detectar lacunas antes do início da execução" },
      { label: "Não", description: "Executar planos sem verificação" }
    ]
  },
  {
    header: "Verificador",
    question: "Verificar se o trabalho satisfaz os requisitos após cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Confirmar que entregas correspondem aos objetivos da fase" },
      { label: "Não", description: "Confiar na execução, pular verificação" }
    ]
  },
  {
    header: "Modelos de IA",
    question: "Quais modelos de IA para agentes de planejamento?",
    multiSelect: false,
    options: [
      { label: "Equilibrado (Recomendado)", description: "Sonnet para a maioria dos agentes — boa relação qualidade/custo" },
      { label: "Qualidade", description: "Opus para pesquisa/roteiro — custo maior, análise mais profunda" },
      { label: "Econômico", description: "Haiku onde possível — mais rápido, menor custo" },
      { label: "Herdar", description: "Usar o modelo da sessão atual para todos os agentes (OpenCode /model)" }
    ]
  }
])
```

Criar `.planning/config.json` com todas as configurações (CLI preenche os padrões restantes automaticamente):

```bash
mkdir -p .planning
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-new-project '{"mode":"yolo","granularity":"[selected]","parallelization":true|false,"commit_docs":true|false,"model_profile":"quality|balanced|budget|inherit","workflow":{"research":true|false,"plan_check":true|false,"verifier":true|false,"nyquist_validation":true|false,"auto_advance":true}}'
```

**Se commit_docs = Não:** Adicionar `.planning/` ao `.gitignore`.

**Commit do config.json:**

```bash
mkdir -p .planning
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "chore: adicionar config do projeto" --files .planning/config.json
```

**Persistir flag de cadeia auto-advance no config (sobrevive à compactação de contexto):**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

Prosseguir para o Passo 4 (pular Passos 3 e 5).

## 3. Questionamento Profundo

**Se modo auto:** Pular (já tratado no Passo 2a). Extrair contexto do projeto a partir do documento fornecido e prosseguir para o Passo 4.

**Exibir banner de etapa:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► QUESTIONAMENTO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Abrir a conversa:**

Pergunte inline (texto livre, sem prompt conversacional):

"O que você quer construir?"

Aguarde a resposta. Isso dá o contexto necessário para fazer perguntas de acompanhamento inteligentes.

**Modo pesquisa-antes-das-perguntas:** Verifique se `workflow.research_before_questions` está habilitado em `.planning/config.json` (ou no config do contexto do init). Quando habilitado, antes de fazer perguntas de acompanhamento sobre uma área:

1. Faça uma breve pesquisa web por melhores práticas relacionadas ao que o usuário descreveu
2. Mencione descobertas-chave naturalmente ao fazer perguntas (ex: "A maioria dos projetos como este usa X — é isso que você está pensando, ou algo diferente?")
3. Isso torna as perguntas mais informadas sem alterar o fluxo conversacional

Quando desabilitado (padrão), faça perguntas diretamente como antes.

**Siga o fio:**

Com base no que disseram, faça perguntas de acompanhamento que aprofundem a resposta. Use um prompt conversacional com opções que investiguem o que mencionaram — interpretações, esclarecimentos, exemplos concretos.

Continue seguindo os fios. Cada resposta abre novos fios para explorar. Pergunte sobre:

- O que os empolgou
- Que problema motivou isso
- O que querem dizer com termos vagos
- Como seria na prática
- O que já está decidido

Consulte `questionamento.md` para técnicas:

- Desafiar vagueza
- Tornar abstrato concreto
- Revelar suposições
- Encontrar limites
- Revelar motivação

**Verificar contexto (internamente, não em voz alta):**

Conforme avança, verifique mentalmente o checklist de contexto de `questionamento.md`. Se houver lacunas, incorpore perguntas naturalmente. Não mude repentinamente para modo checklist.

**Portal de decisão:**

Quando puder escrever um PROJECT.md claro, use um prompt conversacional:

- header: "Pronto?"
- question: "Acho que entendi o que você busca. Pronto para criar o PROJECT.md?"
- options:
  - "Criar PROJECT.md" — Vamos seguir em frente
  - "Continuar explorando" — Quero compartilhar mais / me pergunte mais

Se "Continuar explorando" — pergunte o que querem adicionar, ou identifique lacunas e investigue naturalmente.

Loop até "Criar PROJECT.md" ser selecionado.

## 4. Escrever PROJECT.md

**Se modo auto:** Sintetizar a partir do documento fornecido. Nenhum portal "Pronto?" foi mostrado — prosseguir diretamente para o commit.

Sintetize todo o contexto em `.planning/PROJECT.md` usando o template de `templates/projeto.md`.

**Para projetos greenfield:**

Inicializar requisitos como hipóteses:

```markdown
## Requisitos

### Validados

(Nenhum ainda — lance para validar)

### Ativos

- [ ] [Requisito 1]
- [ ] [Requisito 2]
- [ ] [Requisito 3]

### Fora do Escopo

- [Exclusão 1] — [motivo]
- [Exclusão 2] — [motivo]
```

Todos os requisitos Ativos são hipóteses até serem lançados e validados.

**Para projetos brownfield (mapa da base de código presente):**

Inferir requisitos Validados a partir do código existente:

1. Ler `.planning/codigo/ARCHITECTURE.md` e `STACK.md`
2. Identificar o que a base de código já faz
3. Estes se tornam o conjunto Validado inicial

```markdown
## Requisitos

### Validados

- ✓ [Capacidade existente 1] — existente
- ✓ [Capacidade existente 2] — existente
- ✓ [Capacidade existente 3] — existente

### Ativos

- [ ] [Novo requisito 1]
- [ ] [Novo requisito 2]

### Fora do Escopo

- [Exclusão 1] — [motivo]
```

**Decisões Chave:**

Inicializar com quaisquer decisões tomadas durante o questionamento:

```markdown
## Decisões Chave

| Decisão | Justificativa | Resultado |
|---------|---------------|-----------|
| [Escolha do questionamento] | [Motivo] | — Pendente |
```

**Rodapé de última atualização:**

```markdown
---
*Última atualização: [data] após inicialização*
```

**Seção de evolução** (incluir no final do PROJECT.md, antes do rodapé):

```markdown
## Evolução

Este documento evolui nas transições de fase e limites de marco.

**Após cada transição de fase** (via `/gsd-transicao`):
1. Requisitos invalidados? → Mover para Fora do Escopo com motivo
2. Requisitos validados? → Mover para Validados com referência da fase
3. Novos requisitos emergiram? → Adicionar a Ativos
4. Decisões a registrar? → Adicionar a Decisões Chave
5. "O Que É Isso" ainda preciso? → Atualizar se divergiu

**Após cada marco** (via `/gsd-completar-marco`):
1. Revisão completa de todas as seções
2. Verificação do Valor Central — ainda a prioridade certa?
3. Auditar Fora do Escopo — razões ainda válidas?
4. Atualizar Contexto com estado atual
```

Não comprimir. Capturar tudo que foi coletado.

**Commit do PROJECT.md:**

```bash
mkdir -p .planning
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: inicializar projeto" --files .planning/PROJECT.md
```

## 5. Preferências de fluxo de trabalho

**Se modo auto:** Pular — a config foi coletada no Passo 2a. Prosseguir para o Passo 5.1.5.

**Verificar padrões globais** em `~/.gsd/defaults.json`. Se o arquivo existir, oferecer usar padrões salvos:

```
prompt conversacional([
  {
    question: "Usar suas configurações padrão salvas? (de ~/.gsd/defaults.json)",
    header: "Padrões",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Usar padrões salvos, pular perguntas de configuração" },
      { label: "Não", description: "Configurar manualmente" }
    ]
  }
])
```

Se "Sim": ler `~/.gsd/defaults.json`, usar esses valores para config.json, e pular diretamente para **Commit do config.json** abaixo.

Se "Não" ou `~/.gsd/defaults.json` não existe: prosseguir com as perguntas abaixo.

**Rodada 1 — Configurações principais do fluxo de trabalho (4 perguntas):**

```
perguntas: [
  {
    header: "Modo",
    question: "Como você quer trabalhar?",
    multiSelect: false,
    options: [
      { label: "YOLO (Recomendado)", description: "Auto-aprovar, apenas executar" },
      { label: "Interativo", description: "Confirmar em cada passo" }
    ]
  },
  {
    header: "Granularidade",
    question: "Quão finamente o escopo deve ser dividido em fases?",
    multiSelect: false,
    options: [
      { label: "Grossa", description: "Menos fases, mais amplas (3-5 fases, 1-3 planos cada)" },
      { label: "Padrão", description: "Tamanho equilibrado de fases (5-8 fases, 3-5 planos cada)" },
      { label: "Fina", description: "Muitas fases focadas (8-12 fases, 5-10 planos cada)" }
    ]
  },
  {
    header: "Execução",
    question: "Executar planos em paralelo?",
    multiSelect: false,
    options: [
      { label: "Paralelo (Recomendado)", description: "Planos independentes executam simultaneamente" },
      { label: "Sequencial", description: "Um plano por vez" }
    ]
  },
  {
    header: "Rastreamento Git",
    question: "Fazer commit dos docs de planejamento no git?",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Docs de planejamento rastreados no controle de versão" },
      { label: "Não", description: "Manter .planning/ somente local (adicionar ao .gitignore)" }
    ]
  }
]
```

**Rodada 2 — Agentes do fluxo de trabalho:**

Estes geram agentes adicionais durante planejamento/execução. Adicionam tokens e tempo mas melhoram a qualidade.

| Agente | Quando executa | O que faz |
|--------|----------------|-----------|
| **Pesquisador** | Antes de planejar cada fase | Investiga domínio, encontra padrões, identifica armadilhas |
| **Verificador de Plano** | Após plano ser criado | Verifica se o plano realmente alcança o objetivo da fase |
| **Verificador** | Após execução da fase | Confirma que itens obrigatórios foram entregues |

Todos recomendados para projetos importantes. Pular para experimentos rápidos.

```
perguntas: [
  {
    header: "Pesquisa",
    question: "Pesquisar antes de planejar cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Investigar domínio, encontrar padrões, identificar armadilhas" },
      { label: "Não", description: "Planejar diretamente a partir dos requisitos" }
    ]
  },
  {
    header: "Verificação de Plano",
    question: "Verificar se os planos alcançarão seus objetivos? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Detectar lacunas antes do início da execução" },
      { label: "Não", description: "Executar planos sem verificação" }
    ]
  },
  {
    header: "Verificador",
    question: "Verificar se o trabalho satisfaz os requisitos após cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Confirmar que entregas correspondem aos objetivos da fase" },
      { label: "Não", description: "Confiar na execução, pular verificação" }
    ]
  },
  {
    header: "Modelos de IA",
    question: "Quais modelos de IA para agentes de planejamento?",
    multiSelect: false,
    options: [
      { label: "Equilibrado (Recomendado)", description: "Sonnet para a maioria dos agentes — boa relação qualidade/custo" },
      { label: "Qualidade", description: "Opus para pesquisa/roteiro — custo maior, análise mais profunda" },
      { label: "Econômico", description: "Haiku onde possível — mais rápido, menor custo" },
      { label: "Herdar", description: "Usar o modelo da sessão atual para todos os agentes (OpenCode /model)" }
    ]
  }
]
```

Criar `.planning/config.json` com todas as configurações (CLI preenche os padrões restantes automaticamente):

```bash
mkdir -p .planning
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-new-project '{"mode":"[yolo|interactive]","granularity":"[selected]","parallelization":true|false,"commit_docs":true|false,"model_profile":"quality|balanced|budget|inherit","workflow":{"research":true|false,"plan_check":true|false,"verifier":true|false,"nyquist_validation":[false se granularidade=grossa, true caso contrário]}}'
```

**Nota:** Execute `/gsd-configuracoes` a qualquer momento para atualizar perfil de modelo, agentes do fluxo de trabalho, estratégia de branches e outras preferências.

**Se commit_docs = Não:**

- Definir `commit_docs: false` no config.json
- Adicionar `.planning/` ao `.gitignore` (criar se necessário)

**Se commit_docs = Sim:**

- Nenhuma entrada adicional no gitignore necessária

**Commit do config.json:**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "chore: adicionar config do projeto" --files .planning/config.json
```

## 5.1. Detecção de Sub-Repositórios

**Detectar workspace multi-repo:**

Verificar diretórios com suas próprias pastas `.git` (repositórios separados dentro do workspace):

```bash
find . -maxdepth 1 -type d -not -name ".*" -not -name "node_modules" -exec test -d "{}/.git" \; -print
```

**Se sub-repos encontrados:**

Remover o prefixo `./` para obter nomes de diretórios (ex: `./backend` → `backend`).

Use um prompt conversacional:

- header: "Workspace multi-repositório"
- question: "Detectei repositórios git separados neste workspace. Quais diretórios contêm código que o GSD deve commitar?"
- multiSelect: true
- options: uma opção por diretório detectado
  - "[nome do diretório]" — Repositório git separado

**Se usuário selecionar um ou mais diretórios:**

- Definir `planning.sub_repos` no config.json com o array de nomes de diretórios selecionados (ex: `["backend", "frontend"]`)
- Auto-definir `planning.commit_docs` para `false` (docs de planejamento ficam locais em workspaces multi-repo)
- Adicionar `.planning/` ao `.gitignore` se ainda não presente

Alterações de config são salvas localmente — nenhum commit necessário pois `commit_docs` é `false` em modo multi-repo.

**Se nenhum sub-repo encontrado ou usuário não selecionar nenhum:** Continuar sem alterações no config.

## 5.5. Resolver Perfil de Modelo

Usar modelos do init: `researcher_model`, `synthesizer_model`, `roadmapper_model`.

## 6. Decisão de Pesquisa

**Se modo auto:** Padrão para "Pesquisar primeiro" sem perguntar.

Use um prompt conversacional:

- header: "Pesquisa"
- question: "Pesquisar o ecossistema do domínio antes de definir requisitos?"
- options:
  - "Pesquisar primeiro (Recomendado)" — Descobrir stacks padrão, funcionalidades esperadas, padrões de arquitetura
  - "Pular pesquisa" — Conheço bem este domínio, ir direto para requisitos

**Se "Pesquisar primeiro":**

Exibir banner de etapa:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISANDO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pesquisando ecossistema de [domínio]...
```

Criar diretório de pesquisa:

```bash
mkdir -p .planning/research
```

**Determinar contexto do marco:**

Verificar se é greenfield ou marco subsequente:

- Se não houver requisitos "Validados" no PROJECT.md → Greenfield (construindo do zero)
- Se requisitos "Validados" existirem → Marco subsequente (adicionando a app existente)

Exibir indicador de spawn:

```
◆ Iniciando 4 pesquisadores em paralelo...
  → Pesquisa de stack
  → Pesquisa de funcionalidades
  → Pesquisa de arquitetura
  → Pesquisa de armadilhas
```

Iniciar 4 agentes gsd-pesquisador-projeto em paralelo com referências de caminho:

```
Task(prompt="<research_type>
Pesquisa de Projeto — Dimensão Stack para [domínio].
</research_type>

<milestone_context>
[greenfield OU subsequente]

Greenfield: Pesquisar o stack padrão para construir [domínio] do zero.
Subsequente: Pesquisar o que é necessário para adicionar [funcionalidades alvo] a um app [domínio] existente. Não re-pesquisar o sistema existente.
</milestone_context>

<question>
Qual é o stack padrão 2025 para [domínio]?
</question>

<files_to_read>
- {project_path} (Contexto e objetivos do projeto)
</files_to_read>

<downstream_consumer>
Seu STACK.md alimenta a criação do roteiro. Seja prescritivo:
- Bibliotecas específicas com versões
- Justificativa clara para cada escolha
- O que NÃO usar e por quê
</downstream_consumer>

<quality_gate>
- [ ] Versões são atuais (verificar com Context7/docs oficiais, não dados de treinamento)
- [ ] Justificativa explica POR QUÊ, não apenas O QUÊ
- [ ] Níveis de confiança atribuídos a cada recomendação
</quality_gate>

<output>
Escrever em: .planning/research/STACK.md
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/STACK.md
</output>
", subagent_type="gsd-pesquisador-projeto", model="{researcher_model}", description="Pesquisa de stack")

Task(prompt="<research_type>
Pesquisa de Projeto — Dimensão Funcionalidades para [domínio].
</research_type>

<milestone_context>
[greenfield OU subsequente]

Greenfield: Que funcionalidades produtos de [domínio] têm? O que é essencial vs diferenciador?
Subsequente: Como [funcionalidades alvo] tipicamente funcionam? Qual é o comportamento esperado?
</milestone_context>

<question>
Que funcionalidades produtos de [domínio] têm? O que é essencial vs diferenciador?
</question>

<files_to_read>
- {project_path} (Contexto do projeto)
</files_to_read>

<downstream_consumer>
Seu FEATURES.md alimenta a definição de requisitos. Categorize claramente:
- Essenciais (obrigatório ou usuários abandonam)
- Diferenciadores (vantagem competitiva)
- Anti-funcionalidades (coisas para deliberadamente NÃO construir)
</downstream_consumer>

<quality_gate>
- [ ] Categorias são claras (essenciais vs diferenciadores vs anti-funcionalidades)
- [ ] Complexidade anotada para cada funcionalidade
- [ ] Dependências entre funcionalidades identificadas
</quality_gate>

<output>
Escrever em: .planning/research/FEATURES.md
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/FEATURES.md
</output>
", subagent_type="gsd-pesquisador-projeto", model="{researcher_model}", description="Pesquisa de funcionalidades")

Task(prompt="<research_type>
Pesquisa de Projeto — Dimensão Arquitetura para [domínio].
</research_type>

<milestone_context>
[greenfield OU subsequente]

Greenfield: Como sistemas de [domínio] são tipicamente estruturados? Quais são os componentes principais?
Subsequente: Como [funcionalidades alvo] se integram com a arquitetura existente de [domínio]?
</milestone_context>

<question>
Como sistemas de [domínio] são tipicamente estruturados? Quais são os componentes principais?
</question>

<files_to_read>
- {project_path} (Contexto do projeto)
</files_to_read>

<downstream_consumer>
Seu ARCHITECTURE.md informa a estrutura de fases no roteiro. Inclua:
- Limites de componentes (o que fala com o quê)
- Fluxo de dados (como a informação se move)
- Ordem de construção sugerida (dependências entre componentes)
</downstream_consumer>

<quality_gate>
- [ ] Componentes claramente definidos com limites
- [ ] Direção do fluxo de dados explícita
- [ ] Implicações de ordem de construção anotadas
</quality_gate>

<output>
Escrever em: .planning/research/ARCHITECTURE.md
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/ARCHITECTURE.md
</output>
", subagent_type="gsd-pesquisador-projeto", model="{researcher_model}", description="Pesquisa de arquitetura")

Task(prompt="<research_type>
Pesquisa de Projeto — Dimensão Armadilhas para [domínio].
</research_type>

<milestone_context>
[greenfield OU subsequente]

Greenfield: O que projetos de [domínio] comumente erram? Erros críticos?
Subsequente: Quais são erros comuns ao adicionar [funcionalidades alvo] a [domínio]?
</milestone_context>

<question>
O que projetos de [domínio] comumente erram? Erros críticos?
</question>

<files_to_read>
- {project_path} (Contexto do projeto)
</files_to_read>

<downstream_consumer>
Seu PITFALLS.md previne erros no roteiro/planejamento. Para cada armadilha:
- Sinais de alerta (como detectar cedo)
- Estratégia de prevenção (como evitar)
- Qual fase deve tratar
</downstream_consumer>

<quality_gate>
- [ ] Armadilhas são específicas deste domínio (não conselhos genéricos)
- [ ] Estratégias de prevenção são acionáveis
- [ ] Mapeamento de fase incluído onde relevante
</quality_gate>

<output>
Escrever em: .planning/research/PITFALLS.md
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/PITFALLS.md
</output>
", subagent_type="gsd-pesquisador-projeto", model="{researcher_model}", description="Pesquisa de armadilhas")
```

Após todos os 4 agentes completarem, iniciar sintetizador para criar SUMMARY.md:

```
Task(prompt="
<task>
Sintetizar saídas de pesquisa em SUMMARY.md.
</task>

<files_to_read>
- .planning/research/STACK.md
- .planning/research/FEATURES.md
- .planning/research/ARCHITECTURE.md
- .planning/research/PITFALLS.md
</files_to_read>

<output>
Escrever em: .planning/research/SUMMARY.md
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/SUMMARY.md
Commitar após escrever.
</output>
", subagent_type="gsd-sintetizador-pesquisa", model="{synthesizer_model}", description="Sintetizar pesquisa")
```

Exibir banner de pesquisa completa e descobertas-chave:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISA COMPLETA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Descobertas Chave

**Stack:** [do SUMMARY.md]
**Essenciais:** [do SUMMARY.md]
**Cuidado Com:** [do SUMMARY.md]

Arquivos: `.planning/research/`
```

**Se "Pular pesquisa":** Continuar para o Passo 7.

## 7. Definir Requisitos

Exibir banner de etapa:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DEFININDO REQUISITOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Carregar contexto:**

Ler PROJECT.md e extrair:

- Valor central (a ÚNICA coisa que deve funcionar)
- Restrições declaradas (orçamento, prazo, limitações técnicas)
- Quaisquer limites explícitos de escopo

**Se pesquisa existe:** Ler research/FEATURES.md e extrair categorias de funcionalidades.

**Se modo auto:**

- Auto-incluir todas as funcionalidades essenciais (usuários esperam estas)
- Incluir funcionalidades explicitamente mencionadas no documento fornecido
- Auto-adiar diferenciadores não mencionados no documento
- Pular loops de prompt conversacional por categoria
- Pular pergunta "Alguma adição?"
- Pular portal de aprovação de requisitos
- Gerar REQUIREMENTS.md e commitar diretamente

**Apresentar funcionalidades por categoria (somente modo interativo):**

```
Aqui estão as funcionalidades para [domínio]:

## Autenticação
**Essenciais:**
- Cadastro com email/senha
- Verificação de email
- Recuperação de senha
- Gerenciamento de sessão

**Diferenciadores:**
- Login com link mágico
- OAuth (Google, GitHub)
- 2FA

**Notas de pesquisa:** [notas relevantes]

---

## [Próxima Categoria]
...
```

**Se sem pesquisa:** Coletar requisitos através de conversa.

Pergunte: "Quais são as principais coisas que os usuários precisam poder fazer?"

Para cada capacidade mencionada:

- Faça perguntas de esclarecimento para torná-la específica
- Investigue capacidades relacionadas
- Agrupe em categorias

**Definir escopo de cada categoria:**

Para cada categoria, use um prompt conversacional:

- header: "[Categoria]" (máx 12 caracteres)
- question: "Quais funcionalidades de [categoria] estão na v1?"
- multiSelect: true
- options:
  - "[Funcionalidade 1]" — [breve descrição]
  - "[Funcionalidade 2]" — [breve descrição]
  - "[Funcionalidade 3]" — [breve descrição]
  - "Nenhuma para v1" — Adiar categoria inteira

Rastrear respostas:

- Funcionalidades selecionadas → requisitos v1
- Essenciais não selecionados → v2 (usuários esperam estes)
- Diferenciadores não selecionados → fora do escopo

**Identificar lacunas:**

Use um prompt conversacional:

- header: "Adições"
- question: "Algum requisito que a pesquisa não cobriu? (Funcionalidades específicas da sua visão)"
- options:
  - "Não, a pesquisa cobriu" — Prosseguir
  - "Sim, quero adicionar" — Capturar adições

**Validar valor central:**

Cruzar requisitos com Valor Central do PROJECT.md. Se lacunas detectadas, apresentá-las.

**Gerar REQUIREMENTS.md:**

Criar `.planning/REQUIREMENTS.md` com:

- Requisitos v1 agrupados por categoria (checkboxes, REQ-IDs)
- Requisitos v2 (adiados)
- Fora do Escopo (exclusões explícitas com justificativa)
- Seção de rastreabilidade (vazia, preenchida pelo roteiro)

**Formato REQ-ID:** `[CATEGORIA]-[NÚMERO]` (AUTH-01, CONTENT-02)

**Critérios de qualidade de requisitos:**

Bons requisitos são:

- **Específicos e testáveis:** "Usuário pode redefinir senha via link de email" (não "Tratar redefinição de senha")
- **Centrados no usuário:** "Usuário pode X" (não "Sistema faz Y")
- **Atômicos:** Uma capacidade por requisito (não "Usuário pode fazer login e gerenciar perfil")
- **Independentes:** Dependências mínimas de outros requisitos

Rejeitar requisitos vagos. Exigir especificidade:

- "Tratar autenticação" → "Usuário pode fazer login com email/senha e permanecer logado entre sessões"
- "Suportar compartilhamento" → "Usuário pode compartilhar post via link que abre no navegador do destinatário"

**Apresentar lista completa de requisitos (somente modo interativo):**

Mostrar todos os requisitos (não contagens) para confirmação do usuário:

```
## Requisitos v1

### Autenticação
- [ ] **AUTH-01**: Usuário pode criar conta com email/senha
- [ ] **AUTH-02**: Usuário pode fazer login e permanecer logado entre sessões
- [ ] **AUTH-03**: Usuário pode fazer logout de qualquer página

### Conteúdo
- [ ] **CONT-01**: Usuário pode criar posts com texto
- [ ] **CONT-02**: Usuário pode editar seus próprios posts

[... lista completa ...]

---

Isso captura o que você está construindo? (sim / ajustar)
```

Se "ajustar": Retornar ao escopo.

**Commit dos requisitos:**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: definir requisitos v1" --files .planning/REQUIREMENTS.md
```

## 8. Criar Roteiro

Exibir banner de etapa:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CRIANDO ROTEIRO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando roteirista...
```

Iniciar agente gsd-roteirista com referências de caminho:

```
Task(prompt="
<planning_context>

<files_to_read>
- .planning/PROJECT.md (Contexto do projeto)
- .planning/REQUIREMENTS.md (Requisitos v1)
- .planning/research/SUMMARY.md (Descobertas da pesquisa - se existir)
- .planning/config.json (Configurações de granularidade e modo)
</files_to_read>

</planning_context>

<instructions>
Criar roteiro:
1. Derivar fases a partir dos requisitos (não impor estrutura)
2. Mapear cada requisito v1 para exatamente uma fase
3. Derivar 2-5 critérios de sucesso por fase (comportamentos observáveis do usuário)
4. Validar cobertura de 100%
5. Escrever arquivos imediatamente (ROADMAP.md, STATE.md, atualizar rastreabilidade em REQUIREMENTS.md)
6. Retornar ROADMAP CREATED com resumo

Escrever arquivos primeiro, depois retornar. Isso garante que artefatos persistam mesmo se o contexto for perdido.
</instructions>
", subagent_type="gsd-roteirista", model="{roadmapper_model}", description="Criar roteiro")
```

**Tratar retorno do roteirista:**

**Se `## ROADMAP BLOCKED`:**

- Apresentar informações do bloqueio
- Trabalhar com usuário para resolver
- Re-iniciar quando resolvido

**Se `## ROADMAP CREATED`:**

Ler o ROADMAP.md criado e apresentá-lo inline de forma agradável:

```
---

## Roteiro Proposto

**[N] fases** | **[X] requisitos mapeados** | Todos os requisitos v1 cobertos ✓

| # | Fase | Objetivo | Requisitos | Critérios de Sucesso |
|---|------|----------|------------|---------------------|
| 1 | [Nome] | [Objetivo] | [REQ-IDs] | [contagem] |
| 2 | [Nome] | [Objetivo] | [REQ-IDs] | [contagem] |
| 3 | [Nome] | [Objetivo] | [REQ-IDs] | [contagem] |
...

### Detalhes das Fases

**Fase 1: [Nome]**
Objetivo: [objetivo]
Requisitos: [REQ-IDs]
Critérios de sucesso:
1. [critério]
2. [critério]
3. [critério]

**Fase 2: [Nome]**
Objetivo: [objetivo]
Requisitos: [REQ-IDs]
Critérios de sucesso:
1. [critério]
2. [critério]

[... continuar para todas as fases ...]

---
```

**Se modo auto:** Pular portal de aprovação — auto-aprovar e commitar diretamente.

**CRÍTICO: Pedir aprovação antes de commitar (somente modo interativo):**

Use um prompt conversacional:

- header: "Roteiro"
- question: "Essa estrutura de roteiro funciona para você?"
- options:
  - "Aprovar" — Commitar e continuar
  - "Ajustar fases" — Me diga o que mudar
  - "Revisar arquivo completo" — Mostrar ROADMAP.md bruto

**Se "Aprovar":** Continuar para commit.

**Se "Ajustar fases":**

- Obter notas de ajuste do usuário
- Re-iniciar roteirista com contexto de revisão:

  ```
  Task(prompt="
  <revision>
  Feedback do usuário sobre o roteiro:
  [notas do usuário]

  <files_to_read>
  - .planning/ROADMAP.md (Roteiro atual para revisar)
  </files_to_read>

  Atualizar o roteiro com base no feedback. Editar arquivos no local.
  Retornar ROADMAP REVISED com alterações feitas.
  </revision>
  ", subagent_type="gsd-roteirista", model="{roadmapper_model}", description="Revisar roteiro")
  ```

- Apresentar roteiro revisado
- Loop até usuário aprovar

**Se "Revisar arquivo completo":** Exibir `cat .planning/ROADMAP.md` bruto, depois perguntar novamente.

**Gerar ou atualizar .cursor/rules/ do projeto antes do commit final:**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" generate-claude-md
```

Isso garante que novos projetos recebam a orientação padrão do fluxo de trabalho GSD e o contexto atual do projeto em `.cursor/rules/`.

**Commit do roteiro (após aprovação ou modo auto):**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: criar roteiro ([N] fases)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md .cursor/rules/
```

## 9. Concluído

Apresentar resumo de conclusão:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PROJETO INICIALIZADO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**[Nome do Projeto]**

| Artefato       | Localização                  |
|----------------|------------------------------|
| Projeto        | `.planning/PROJECT.md`       |
| Configuração   | `.planning/config.json`      |
| Pesquisa       | `.planning/research/`        |
| Requisitos     | `.planning/REQUIREMENTS.md`  |
| Roteiro        | `.planning/ROADMAP.md`       |
| Guia do projeto | `.cursor/rules/`             |

**[N] fases** | **[X] requisitos** | Pronto para construir ✓
```

**Se modo auto:**

```
╔══════════════════════════════════════════╗
║  AUTO-AVANÇANDO → DISCUTIR FASE 1        ║
╚══════════════════════════════════════════╝
```

Sair da skill e invocar SlashCommand("/gsd-discutir-fase 1 --auto")

**Se modo interativo:**

Verificar se a Fase 1 tem indicadores de UI (procurar `**UI hint**: yes` na seção de detalhes da Fase 1 do ROADMAP.md):

```bash
PHASE1_SECTION=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase 1 2>/dev/null)
PHASE1_HAS_UI=$(echo "$PHASE1_SECTION" | grep -qi "UI hint.*yes" && echo "true" || echo "false")
```

**Se Fase 1 tem UI (`PHASE1_HAS_UI` é `true`):**

```
───────────────────────────────────────────────────────────────

## ▶ Próximo

**Fase 1: [Nome da Fase]** — [Objetivo do ROADMAP.md]

/gsd-discutir-fase 1 — coletar contexto e esclarecer abordagem

<sub>/clear primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- /gsd-fase-ui 1 — gerar contrato de design UI (recomendado para fases frontend)
- /gsd-planejar-fase 1 — pular discussão, planejar diretamente

───────────────────────────────────────────────────────────────
```

**Se Fase 1 não tem UI:**

```
───────────────────────────────────────────────────────────────

## ▶ Próximo

**Fase 1: [Nome da Fase]** — [Objetivo do ROADMAP.md]

/gsd-discutir-fase 1 — coletar contexto e esclarecer abordagem

<sub>/clear primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- /gsd-planejar-fase 1 — pular discussão, planejar diretamente

───────────────────────────────────────────────────────────────
```

</process>

<output>

- `.planning/PROJECT.md`
- `.planning/config.json`
- `.planning/research/` (se pesquisa selecionada)
  - `STACK.md`
  - `FEATURES.md`
  - `ARCHITECTURE.md`
  - `PITFALLS.md`
  - `SUMMARY.md`
- `.planning/REQUIREMENTS.md`
- `.planning/ROADMAP.md`
- `.planning/STATE.md`
- `.cursor/rules/`

<success_criteria>

- [ ] Diretório .planning/ criado
- [ ] Repositório Git inicializado
- [ ] Detecção brownfield completada
- [ ] Questionamento profundo completado (fios seguidos, não apressado)
- [ ] PROJECT.md captura contexto completo → **commitado**
- [ ] config.json tem modo de fluxo de trabalho, granularidade, paralelização → **commitado**
- [ ] Pesquisa completada (se selecionada) — 4 agentes paralelos iniciados → **commitado**
- [ ] Requisitos coletados (da pesquisa ou conversa)
- [ ] Usuário definiu escopo de cada categoria (v1/v2/fora do escopo)
- [ ] REQUIREMENTS.md criado com REQ-IDs → **commitado**
- [ ] gsd-roteirista iniciado com contexto
- [ ] Arquivos do roteiro escritos imediatamente (não rascunho)
- [ ] Feedback do usuário incorporado (se houver)
- [ ] ROADMAP.md criado com fases, mapeamentos de requisitos, critérios de sucesso
- [ ] STATE.md inicializado
- [ ] Rastreabilidade do REQUIREMENTS.md atualizada
- [ ] .cursor/rules/ gerado com orientação do fluxo de trabalho GSD
- [ ] Usuário sabe que o próximo passo é `/gsd-discutir-fase 1`

**Commits atômicos:** Cada fase commita seus artefatos imediatamente. Se o contexto for perdido, artefatos persistem.

</success_criteria>
</output>
