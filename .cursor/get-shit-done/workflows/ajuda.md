<purpose>
Exibir a referência completa de comandos GSD. Mostrar APENAS o conteúdo de referência. NÃO adicionar análise específica do projeto, status do git, sugestões de próximos passos, ou qualquer comentário além da referência.
</purpose>

<reference>
# Referência de Comandos GSD

**GSD** (Get Shit Done) cria planos hierárquicos de projeto otimizados para desenvolvimento solo com agentes no Cursor.

## Início Rápido

1. `/gsd-novo-projeto` - Inicializar projeto (inclui pesquisa, requisitos, roteiro)
2. `/gsd-planejar-fase 1` - Criar plano detalhado para a primeira fase
3. `/gsd-executar-fase 1` - Executar a fase

## Mantendo Atualizado

GSD evolui rápido. Atualize periodicamente:

```bash
npx get-shit-done-cc@latest
```

## Fluxo Principal

```
/gsd-novo-projeto → /gsd-planejar-fase → /gsd-executar-fase → repetir
```

### Inicialização do Projeto

**`/gsd-novo-projeto`**
Inicializar novo projeto através de fluxo unificado.

Um comando te leva da ideia até pronto-para-planejar:
- Questionamento profundo para entender o que você está construindo
- Pesquisa de domínio opcional (dispara 4 agentes pesquisadores em paralelo)
- Definição de requisitos com escopo v1/v2/fora-do-escopo
- Criação de roteiro com divisão em fases e critérios de sucesso

Cria todos os artefatos em `.planning/`:
- `PROJECT.md` — visão e requisitos
- `config.json` — modo de fluxo (interativo/yolo)
- `research/` — pesquisa de domínio (se selecionada)
- `REQUIREMENTS.md` — requisitos com escopo e REQ-IDs
- `ROADMAP.md` — fases mapeadas para requisitos
- `STATE.md` — memória do projeto

Uso: `/gsd-novo-projeto`

**`/gsd-mapear-codigo`**
Mapear uma base de código existente para projetos brownfield.

- Analisa a base de código com agentes Explore em paralelo
- Cria `.planning/codebase/` com 7 documentos focados
- Cobre stack, arquitetura, estrutura, convenções, testes, integrações, preocupações
- Use antes de `/gsd-novo-projeto` em bases de código existentes

Uso: `/gsd-mapear-codigo`

### Planejamento de Fase

**`/gsd-discutir-fase <número>`**
Ajudar a articular sua visão para uma fase antes do planejamento.

- Captura como você imagina esta fase funcionando
- Cria CONTEXT.md com sua visão, essenciais e limites
- Use quando tiver ideias sobre como algo deveria parecer/funcionar
- Opcional `--batch` faz 2-5 perguntas relacionadas por vez ao invés de uma por uma

Uso: `/gsd-discutir-fase 2`
Uso: `/gsd-discutir-fase 2 --batch`
Uso: `/gsd-discutir-fase 2 --batch=3`

**`/gsd-pesquisar-fase <número>`**
Pesquisa abrangente do ecossistema para domínios de nicho/complexos.

- Descobre stack padrão, padrões de arquitetura, armadilhas
- Cria RESEARCH.md com conhecimento de "como especialistas constroem isso"
- Use para 3D, jogos, áudio, shaders, ML e outros domínios especializados
- Vai além de "qual biblioteca" para conhecimento do ecossistema

Uso: `/gsd-pesquisar-fase 3`

**`/gsd-listar-premissas-fase <número>`**
Veja o que o Claude está planejando fazer antes de começar.

- Mostra a abordagem pretendida do Claude para uma fase
- Permite correção de rumo se o Claude entendeu errado sua visão
- Nenhum arquivo criado - apenas saída conversacional

Uso: `/gsd-listar-premissas-fase 3`

**`/gsd-planejar-fase <número>`**
Criar plano de execução detalhado para uma fase específica.

- Gera `.planning/phases/XX-nome-fase/XX-YY-PLAN.md`
- Divide a fase em tarefas concretas e acionáveis
- Inclui critérios de verificação e medidas de sucesso
- Múltiplos planos por fase suportados (XX-01, XX-02, etc.)

Uso: `/gsd-planejar-fase 1`
Resultado: Cria `.planning/phases/01-foundation/01-01-PLAN.md`

**Caminho Expresso PRD:** Passe `--prd caminho/para/requisitos.md` para pular discuss-phase inteiramente. Seu PRD se torna decisões travadas no CONTEXT.md. Útil quando você já tem critérios de aceite claros.

### Execução

**`/gsd-executar-fase <número-fase>`**
Executar todos os planos em uma fase, ou executar uma onda específica.

- Agrupa planos por onda (do frontmatter), executa ondas sequencialmente
- Planos dentro de cada onda rodam em paralelo via ferramenta Task
- Flag opcional `--wave N` executa apenas a Onda `N` e para, a menos que a fase esteja totalmente completa
- Verifica objetivo da fase após todos os planos completarem
- Atualiza REQUIREMENTS.md, ROADMAP.md, STATE.md

Uso: `/gsd-executar-fase 5`
Uso: `/gsd-executar-fase 5 --wave 2`

### Roteador Inteligente

**`/gsd-fazer <descrição>`**
Rotear texto livre para o comando GSD correto automaticamente.

- Analisa entrada em linguagem natural para encontrar o melhor comando GSD correspondente
- Age como despachante — nunca faz o trabalho em si
- Resolve ambiguidade pedindo que você escolha entre as melhores opções
- Use quando sabe o que quer mas não sabe qual comando `/gsd-*` executar

Uso: `/gsd-fazer corrigir o botão de login`
Uso: `/gsd-fazer refatorar o sistema de autenticação`
Uso: `/gsd-fazer quero iniciar um novo marco`

### Modo Rápido

**`/gsd-rapido-garantido [--full] [--discuss] [--research]`**
Executar tarefas pequenas e pontuais com garantias GSD mas pulando agentes opcionais.

Modo rápido usa o mesmo sistema com um caminho mais curto:
- Dispara planejador + executor (pula pesquisador, verificador de plano, verificador por padrão)
- Tarefas rápidas ficam em `.planning/quick/` separadas das fases planejadas
- Atualiza rastreamento do STATE.md (não ROADMAP.md)

Flags habilitam etapas adicionais de qualidade:
- `--discuss` — Discussão leve para levantar áreas cinzentas antes do planejamento
- `--research` — Agente de pesquisa focado investiga abordagens antes do planejamento
- `--full` — Adiciona verificação de plano (máx 2 iterações) e verificação pós-execução

Flags são combináveis: `--discuss --research --full` dá o pipeline completo de qualidade para uma única tarefa.

Uso: `/gsd-rapido-garantido`
Uso: `/gsd-rapido-garantido --research --full`
Resultado: Cria `.planning/quick/NNN-slug/PLAN.md`, `.planning/quick/NNN-slug/SUMMARY.md`

---

**`/gsd-rapido [descrição]`**
Executar uma tarefa trivial inline — sem subagentes, sem arquivos de planejamento, sem overhead.

Para tarefas pequenas demais para justificar planejamento: correção de typo, mudanças de config, commits esquecidos, adições simples. Roda no contexto atual, faz a mudança, commita e registra no STATE.md.

- Nenhum PLAN.md ou SUMMARY.md criado
- Nenhum subagente disparado (roda inline)
- ≤ 3 edições de arquivo — redireciona para `/gsd-rapido-garantido` se a tarefa não for trivial
- Commit atômico com mensagem convencional

Uso: `/gsd-rapido "corrigir o typo no README"`
Uso: `/gsd-rapido "adicionar .env ao gitignore"`

### Gestão do Roteiro

**`/gsd-adicionar-fase <descrição>`**
Adicionar nova fase ao final do marco atual.

- Anexa ao ROADMAP.md
- Usa próximo número sequencial
- Atualiza estrutura de diretórios das fases

Uso: `/gsd-adicionar-fase "Adicionar painel administrativo"`

**`/gsd-inserir-fase <após> <descrição>`**
Inserir trabalho urgente como fase decimal entre fases existentes.

- Cria fase intermediária (ex: 7.1 entre 7 e 8)
- Útil para trabalho descoberto que precisa acontecer no meio do marco
- Mantém ordenação das fases

Uso: `/gsd-inserir-fase 7 "Corrigir bug crítico de autenticação"`
Resultado: Cria Fase 7.1

**`/gsd-remover-fase <número>`**
Remover uma fase futura e renumerar fases subsequentes.

- Deleta diretório da fase e todas as referências
- Renumera todas as fases subsequentes para fechar a lacuna
- Funciona apenas em fases futuras (não iniciadas)
- Commit git preserva registro histórico

Uso: `/gsd-remover-fase 17`
Resultado: Fase 17 deletada, fases 18-20 tornam-se 17-19

### Gestão de Marcos

**`/gsd-novo-marco <nome>`**
Iniciar um novo marco através de fluxo unificado.

- Questionamento profundo para entender o que você está construindo a seguir
- Pesquisa de domínio opcional (dispara 4 agentes pesquisadores em paralelo)
- Definição de requisitos com escopo
- Criação de roteiro com divisão em fases
- Flag opcional `--reset-phase-numbers` reinicia numeração na Fase 1 e arquiva diretórios de fases antigas primeiro por segurança

Espelha o fluxo `/gsd-novo-projeto` para projetos brownfield (PROJECT.md existente).

Uso: `/gsd-novo-marco "Funcionalidades v2.0"`
Uso: `/gsd-novo-marco --reset-phase-numbers "Funcionalidades v2.0"`

**`/gsd-completar-marco <versão>`**
Arquivar marco concluído e preparar para próxima versão.

- Cria entrada no MILESTONES.md com estatísticas
- Arquiva detalhes completos no diretório milestones/
- Cria tag git para o release
- Prepara workspace para próxima versão

Uso: `/gsd-completar-marco 1.0.0`

### Acompanhamento de Progresso

**`/gsd-progresso`**
Verificar status do projeto e rotear inteligentemente para a próxima ação.

- Mostra barra de progresso visual e porcentagem de conclusão
- Resume trabalho recente dos arquivos SUMMARY
- Exibe posição atual e o que vem a seguir
- Lista decisões-chave e questões em aberto
- Oferece executar próximo plano ou criá-lo se ausente
- Detecta 100% de conclusão do marco

Uso: `/gsd-progresso`

### Gestão de Sessão

**`/gsd-retomar-trabalho`**
Retomar trabalho da sessão anterior com restauração completa de contexto.

- Lê STATE.md para contexto do projeto
- Mostra posição atual e progresso recente
- Oferece próximas ações baseadas no estado do projeto

Uso: `/gsd-retomar-trabalho`

**`/gsd-pausar-trabalho`**
Criar handoff de contexto ao pausar trabalho no meio de uma fase.

- Cria arquivo .continue-here com estado atual
- Atualiza seção de continuidade de sessão do STATE.md
- Captura contexto do trabalho em andamento

Uso: `/gsd-pausar-trabalho`

### Depuração

**`/gsd-depurar [descrição do problema]`**
Depuração sistemática com estado persistente entre resets de contexto.

- Coleta sintomas através de questionamento adaptativo
- Cria `.planning/debug/[slug].md` para rastrear investigação
- Investiga usando método científico (evidência → hipótese → teste)
- Sobrevive a `/clear` — execute `/gsd-depurar` sem argumentos para retomar
- Arquiva problemas resolvidos em `.planning/debug/resolved/`

Uso: `/gsd-depurar "botão de login não funciona"`
Uso: `/gsd-depurar` (retomar sessão ativa)

### Notas Rápidas

**`/gsd-nota <texto>`**
Captura de ideias sem fricção — um comando, salva instantâneo, sem perguntas.

- Salva nota com timestamp em `.planning/notes/` (ou `D:/projetos/Estudo/devsquad/.cursor/notes/` globalmente)
- Três subcomandos: append (padrão), list, promote
- Promote converte uma nota em todo estruturado
- Funciona sem um projeto (usa escopo global como fallback)

Uso: `/gsd-nota refatorar o sistema de hooks`
Uso: `/gsd-nota list`
Uso: `/gsd-nota promote 3`
Uso: `/gsd-nota --global ideia entre projetos`

### Gestão de Todos

**`/gsd-adicionar-todo [descrição]`**
Capturar ideia ou tarefa como todo da conversa atual.

- Extrai contexto da conversa (ou usa descrição fornecida)
- Cria arquivo todo estruturado em `.planning/todos/pending/`
- Infere área dos caminhos de arquivo para agrupamento
- Verifica duplicatas antes de criar
- Atualiza contagem de todos no STATE.md

Uso: `/gsd-adicionar-todo` (infere da conversa)
Uso: `/gsd-adicionar-todo Adicionar refresh de token de autenticação`

**`/gsd-verificar-todos [área]`**
Listar todos pendentes e selecionar um para trabalhar.

- Lista todos os todos pendentes com título, área, idade
- Filtro opcional por área (ex: `/gsd-verificar-todos api`)
- Carrega contexto completo para todo selecionado
- Roteia para ação apropriada (trabalhar agora, adicionar à fase, brainstorm)
- Move todo para done/ quando o trabalho começa

Uso: `/gsd-verificar-todos`
Uso: `/gsd-verificar-todos api`

### Teste de Aceite do Usuário

**`/gsd-verificar-trabalho [fase]`**
Validar funcionalidades construídas através de UAT conversacional.

- Extrai entregáveis testáveis dos arquivos SUMMARY.md
- Apresenta testes um por vez (respostas sim/não)
- Diagnostica falhas automaticamente e cria planos de correção
- Pronto para re-execução se problemas encontrados

Uso: `/gsd-verificar-trabalho 3`

### Enviar Trabalho

**`/gsd-enviar [fase]`**
Criar um PR do trabalho de fase concluído com corpo auto-gerado.

- Envia branch para remote
- Cria PR com resumo do SUMMARY.md, VERIFICATION.md, REQUIREMENTS.md
- Opcionalmente solicita revisão de código
- Atualiza STATE.md com status de envio

Pré-requisitos: Fase verificada, CLI `gh` instalado e autenticado.

Uso: `/gsd-enviar 4` ou `/gsd-enviar 4 --draft`

---

**`/gsd-revisar --phase N [--gemini] [--claude] [--codex] [--all]`**
Revisão por pares entre IAs — invocar CLIs externos de IA para revisar planos de fase independentemente.

- Detecta CLIs disponíveis (gemini, claude, codex)
- Cada CLI revisa planos independentemente com o mesmo prompt estruturado
- Produz REVIEWS.md com feedback por revisor e resumo de consenso
- Alimente revisões de volta no planejamento: `/gsd-planejar-fase N --reviews`

Uso: `/gsd-revisar --phase 3 --all`

---

**`/gsd-branch-pr [destino]`**
Criar branch limpa para pull requests filtrando commits de .planning/.

- Classifica commits: apenas-código (incluir), apenas-planejamento (excluir), misto (incluir sem .planning/)
- Cherry-pick de commits de código para branch limpa
- Revisores veem apenas mudanças de código, sem artefatos GSD

Uso: `/gsd-branch-pr` ou `/gsd-branch-pr main`

---

**`/gsd-plantar-semente [ideia]`**
Capturar uma ideia prospectiva com condições de gatilho para surfacing automático.

- Seeds preservam POR QUÊ, QUANDO surgir, e migalhas para código relacionado
- Surge automaticamente durante `/gsd-novo-marco` quando condições de gatilho são atendidas
- Melhor que itens adiados — gatilhos são verificados, não esquecidos

Uso: `/gsd-plantar-semente "adicionar notificações em tempo real quando construirmos o sistema de eventos"`

---

**`/gsd-auditar-tau`**
Auditoria entre fases de todos os itens pendentes de UAT e verificação.
- Verifica cada fase para itens pendentes, pulados, bloqueados e que precisam de humano
- Referência cruzada contra base de código para detectar documentação desatualizada
- Produz plano de teste humano priorizado agrupado por testabilidade
- Use antes de iniciar um novo marco para limpar débito de verificação

Uso: `/gsd-auditar-tau`

### Auditoria de Marco

**`/gsd-auditar-marco [versão]`**
Auditar conclusão do marco contra intenção original.

- Lê todos os arquivos VERIFICATION.md das fases
- Verifica cobertura de requisitos
- Dispara verificador de integração para conectividade entre fases
- Cria MILESTONE-AUDIT.md com lacunas e débito técnico

Uso: `/gsd-auditar-marco`

**`/gsd-planejar-lacunas-marco`**
Criar fases para fechar lacunas identificadas pela auditoria.

- Lê MILESTONE-AUDIT.md e agrupa lacunas em fases
- Prioriza por prioridade de requisito (must/should/nice)
- Adiciona fases de fechamento de lacunas ao ROADMAP.md
- Pronto para `/gsd-planejar-fase` nas novas fases

Uso: `/gsd-planejar-lacunas-marco`

### Configuração

**`/gsd-configuracoes`**
Configurar toggles de fluxo e perfil de modelo interativamente.

- Alterna agentes de pesquisador, verificador de plano, verificador
- Seleciona perfil de modelo (quality/balanced/budget/inherit)
- Atualiza `.planning/config.json`

Uso: `/gsd-configuracoes`

**`/gsd-definir-perfil <perfil>`**
Troca rápida de perfil de modelo para agentes GSD.

- `quality` — Opus em todo lugar exceto verificação
- `balanced` — Opus para planejamento, Sonnet para execução (padrão)
- `budget` — Sonnet para escrita, Haiku para pesquisa/verificação
- `inherit` — Usar modelo da sessão atual para todos os agentes (OpenCode `/model`)

Uso: `/gsd-definir-perfil budget`

### Comandos Utilitários

**`/gsd-limpeza`**
Arquivar diretórios de fases acumulados de marcos concluídos.

- Identifica fases de marcos concluídos ainda em `.planning/phases/`
- Mostra resumo dry-run antes de mover qualquer coisa
- Move diretórios de fases para `.planning/milestones/v{X.Y}-phases/`
- Use após múltiplos marcos para reduzir desordem em `.planning/phases/`

Uso: `/gsd-limpeza`

**`/gsd-ajuda`**
Mostrar esta referência de comandos.

**`/gsd-atualizar`**
Atualizar GSD para versão mais recente com prévia do changelog.

- Mostra comparação entre versão instalada e mais recente
- Exibe entradas do changelog para versões que você perdeu
- Destaca mudanças que quebram compatibilidade
- Confirma antes de executar instalação
- Melhor que `npx get-shit-done-cc` direto

Uso: `/gsd-atualizar`

**`/gsd-entrar-discord`**
Entrar na comunidade Discord do GSD.

- Obtenha ajuda, compartilhe o que está construindo, fique atualizado
- Conecte-se com outros usuários GSD

Uso: `/gsd-entrar-discord`

## Arquivos e Estrutura

```
.planning/
├── PROJECT.md            # Visão do projeto
├── ROADMAP.md            # Divisão de fases atual
├── STATE.md              # Memória e contexto do projeto
├── RETROSPECTIVE.md      # Retrospectiva viva (atualizada por marco)
├── config.json           # Modo de fluxo e portões
├── todos/                # Ideias e tarefas capturadas
│   ├── pending/          # Todos aguardando trabalho
│   └── done/             # Todos concluídos
├── debug/                # Sessões de depuração ativas
│   └── resolved/         # Problemas resolvidos arquivados
├── milestones/
│   ├── v1.0-ROADMAP.md       # Snapshot de roteiro arquivado
│   ├── v1.0-REQUIREMENTS.md  # Requisitos arquivados
│   └── v1.0-phases/          # Dirs de fase arquivados (via /gsd-limpeza ou --archive-phases)
│       ├── 01-foundation/
│       └── 02-core-features/
├── codebase/             # Mapa da base de código (projetos brownfield)
│   ├── STACK.md          # Linguagens, frameworks, dependências
│   ├── ARCHITECTURE.md   # Padrões, camadas, fluxo de dados
│   ├── STRUCTURE.md      # Layout de diretórios, arquivos-chave
│   ├── CONVENTIONS.md    # Padrões de código, nomenclatura
│   ├── TESTING.md        # Configuração de testes, padrões
│   ├── INTEGRATIONS.md   # Serviços externos, APIs
│   └── CONCERNS.md       # Débito técnico, problemas conhecidos
└── phases/
    ├── 01-foundation/
    │   ├── 01-01-PLAN.md
    │   └── 01-01-SUMMARY.md
    └── 02-core-features/
        ├── 02-01-PLAN.md
        └── 02-01-SUMMARY.md
```

## Modos de Fluxo

Definido durante `/gsd-novo-projeto`:

**Modo Interativo**

- Confirma cada decisão importante
- Pausa em pontos de verificação para aprovação
- Mais orientação ao longo do processo

**Modo YOLO**

- Auto-aprova a maioria das decisões
- Executa planos sem confirmação
- Para apenas em pontos de verificação críticos

Altere a qualquer momento editando `.planning/config.json`

## Configuração de Planejamento

Configure como artefatos de planejamento são gerenciados em `.planning/config.json`:

**`planning.commit_docs`** (padrão: `true`)
- `true`: Artefatos de planejamento commitados no git (fluxo padrão)
- `false`: Artefatos de planejamento mantidos apenas localmente, não commitados

Quando `commit_docs: false`:
- Adicione `.planning/` ao seu `.gitignore`
- Útil para contribuições OSS, projetos de clientes, ou manter planejamento privado
- Todos os arquivos de planejamento funcionam normalmente, apenas não rastreados no git

**`planning.search_gitignored`** (padrão: `false`)
- `true`: Adicionar `--no-ignore` a buscas amplas do ripgrep
- Necessário apenas quando `.planning/` está no gitignore e você quer que buscas em todo o projeto o incluam

Exemplo de config:
```json
{
  "planning": {
    "commit_docs": false,
    "search_gitignored": true
  }
}
```

## Fluxos Comuns

**Iniciando um novo projeto:**

```
/gsd-novo-projeto        # Fluxo unificado: questionamento → pesquisa → requisitos → roteiro
/clear
/gsd-planejar-fase 1       # Criar planos para a primeira fase
/clear
/gsd-executar-fase 1    # Executar todos os planos da fase
```

**Retomando trabalho após uma pausa:**

```
/gsd-progresso  # Veja onde parou e continue
```

**Adicionando trabalho urgente no meio do marco:**

```
/gsd-inserir-fase 5 "Correção crítica de segurança"
/gsd-planejar-fase 5.1
/gsd-executar-fase 5.1
```

**Concluindo um marco:**

```
/gsd-completar-marco 1.0.0
/clear
/gsd-novo-marco  # Iniciar próximo marco (questionamento → pesquisa → requisitos → roteiro)
```

**Capturando ideias durante o trabalho:**

```
/gsd-adicionar-todo                    # Capturar do contexto da conversa
/gsd-adicionar-todo Corrigir z-index do modal  # Capturar com descrição explícita
/gsd-verificar-todos                 # Revisar e trabalhar nos todos
/gsd-verificar-todos api             # Filtrar por área
```

**Depurando um problema:**

```
/gsd-depurar "envio de formulário falha silenciosamente"  # Iniciar sessão de depuração
# ... investigação acontece, contexto enche ...
/clear
/gsd-depurar                                    # Retomar de onde parou
```

## Obtendo Ajuda

- Leia `.planning/PROJECT.md` para visão do projeto
- Leia `.planning/STATE.md` para contexto atual
- Verifique `.planning/ROADMAP.md` para status das fases
- Execute `/gsd-progresso` para verificar onde você está
</reference>
