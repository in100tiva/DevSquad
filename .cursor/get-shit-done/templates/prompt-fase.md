# Template de Prompt de Fase

> **Nota:** A metodologia de planejamento está em `agents/gsd-planner.md`.
> Este template define o formato de saída PLAN.md que o agente produz.

Template para `.planning/phases/XX-nome/{fase}-{plano}-PLAN.md` - planos de fase executáveis otimizados para execução paralela.

**Nomenclatura:** Use o formato `{fase}-{plano}-PLAN.md` (ex., `01-02-PLAN.md` para Fase 1, Plano 2)

---

## Template do Arquivo

```markdown
---
phase: XX-nome
plan: NN
type: execute
wave: N                     # Onda de execução (1, 2, 3...). Pré-calculada no momento do plano.
depends_on: []              # IDs de plano que este plano requer (ex., ["01-01"]).
files_modified: []          # Arquivos que este plano modifica.
autonomous: true            # false se plano tem checkpoints que requerem interação do usuário
requirements: []            # OBRIGATÓRIO — IDs de requisito do ROADMAP que este plano endereça. NÃO PODE ser vazio.
user_setup: []              # Configuração humana que o Claude não pode automatizar (veja abaixo)

# Verificação goal-backward (derivada durante planejamento, verificada após execução)
must_haves:
  truths: []                # Comportamentos observáveis que devem ser verdadeiros para alcançar o objetivo
  artifacts: []             # Arquivos que devem existir com implementação real
  key_links: []             # Conexões críticas entre artefatos
---

<objective>
[O que este plano realiza]

Purpose: [Por que isto importa para o projeto]
Output: [Quais artefatos serão criados]
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/execute-plan.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/resumo.md
[Se plano contém tarefas checkpoint (type="checkpoint:*"), adicione:]
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# Apenas referencie SUMMARYs de planos anteriores se genuinamente necessário:
# - Este plano usa tipos/exports de plano anterior
# - Plano anterior tomou decisão que afeta este plano
# NÃO encadeie reflexivamente: Plano 02 ref 01, Plano 03 ref 02...

[Arquivos fonte relevantes:]
@src/caminho/para/relevante.ts
</context>

<tasks>

<task type="auto">
  <name>Tarefa 1: [Nome orientado a ação]</name>
  <files>caminho/para/arquivo.ext, outro/arquivo.ext</files>
  <read_first>caminho/para/referencia.ext, caminho/para/fonte-de-verdade.ext</read_first>
  <action>[Implementação específica - o que fazer, como fazer, o que evitar e POR QUÊ. Inclua VALORES CONCRETOS: identificadores exatos, parâmetros, saídas esperadas, caminhos de arquivo, argumentos de comando. Nunca diga "alinhe X com Y" sem especificar o estado alvo exato.]</action>
  <verify>[Comando ou verificação para provar que funcionou]</verify>
  <acceptance_criteria>
    - [Condição verificável por grep: "arquivo.ext contém 'string exata'"]
    - [Condição mensurável: "output.ext usa 'valor-esperado', NÃO 'valor-errado'"]
  </acceptance_criteria>
  <done>[Critério de aceitação mensurável]</done>
</task>

<task type="auto">
  <name>Tarefa 2: [Nome orientado a ação]</name>
  <files>caminho/para/arquivo.ext</files>
  <read_first>caminho/para/referencia.ext</read_first>
  <action>[Implementação específica com valores concretos]</action>
  <verify>[Comando ou verificação]</verify>
  <acceptance_criteria>
    - [Condição verificável por grep]
  </acceptance_criteria>
  <done>[Critério de aceitação]</done>
</task>

<!-- Para exemplos e padrões de tarefas checkpoint, veja @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/checkpoints.md -->

<task type="checkpoint:decision" gate="blocking">
  <decision>[O que precisa ser decidido]</decision>
  <context>[Por que esta decisão importa]</context>
  <options>
    <option id="opcao-a"><name>[Nome]</name><pros>[Benefícios]</pros><cons>[Trade-offs]</cons></option>
    <option id="opcao-b"><name>[Nome]</name><pros>[Benefícios]</pros><cons>[Trade-offs]</cons></option>
  </options>
  <resume-signal>Selecione: opcao-a ou opcao-b</resume-signal>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[O que o Claude construiu] - servidor rodando em [URL]</what-built>
  <how-to-verify>Visite [URL] e verifique: [verificações visuais apenas, SEM comandos CLI]</how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva os problemas</resume-signal>
</task>

</tasks>

<verification>
Antes de declarar o plano como concluído:
- [ ] [Comando de teste específico]
- [ ] [Build/verificação de tipos passa]
- [ ] [Verificação de comportamento]
</verification>

<success_criteria>

- Todas as tarefas concluídas
- Todas as verificações passam
- Nenhum erro ou aviso introduzido
- [Critério específico do plano]
  </success_criteria>

<output>
Após conclusão, crie `.planning/phases/XX-nome/{fase}-{plano}-SUMMARY.md`
</output>
```

---

## Campos do Frontmatter

| Campo | Obrigatório | Propósito |
|-------|-------------|-----------|
| `phase` | Sim | Identificador da fase (ex., `01-fundacao`) |
| `plan` | Sim | Número do plano dentro da fase (ex., `01`, `02`) |
| `type` | Sim | Sempre `execute` para planos padrão, `tdd` para planos TDD |
| `wave` | Sim | Número da onda de execução (1, 2, 3...). Pré-calculado no momento do plano. |
| `depends_on` | Sim | Array de IDs de plano que este plano requer. |
| `files_modified` | Sim | Arquivos que este plano modifica. |
| `autonomous` | Sim | `true` se sem checkpoints, `false` se tem checkpoints |
| `requirements` | Sim | **DEVE** listar IDs de requisitos do ROADMAP. Todo requisito do roteiro DEVE aparecer em pelo menos um plano. |
| `user_setup` | Não | Array de itens de configuração que requerem ação humana (serviços externos) |
| `must_haves` | Sim | Critérios de verificação goal-backward (veja abaixo) |

**Wave é pré-calculado:** Números de onda são atribuídos durante `/gsd-plan-phase`. Execute-phase lê `wave` diretamente do frontmatter e agrupa planos por número de onda. Nenhuma análise de dependência em runtime é necessária.

**Must-haves habilitam verificação:** O campo `must_haves` carrega requisitos goal-backward do planejamento para a execução. Após todos os planos serem concluídos, execute-phase dispara um subagente de verificação que verifica estes critérios contra o codebase real.

---

## Paralelo vs Sequencial

<parallel_examples>

**Candidatos a Onda 1 (paralelo):**

```yaml
# Plano 01 - Funcionalidade de usuário
wave: 1
depends_on: []
files_modified: [src/models/user.ts, src/api/users.ts]
autonomous: true

# Plano 02 - Funcionalidade de produto (sem sobreposição com Plano 01)
wave: 1
depends_on: []
files_modified: [src/models/product.ts, src/api/products.ts]
autonomous: true

# Plano 03 - Funcionalidade de pedido (sem sobreposição)
wave: 1
depends_on: []
files_modified: [src/models/order.ts, src/api/orders.ts]
autonomous: true
```

Todos os três rodam em paralelo (Onda 1) - sem dependências, sem conflitos de arquivo.

**Sequencial (dependência genuína):**

```yaml
# Plano 01 - Fundação de auth
wave: 1
depends_on: []
files_modified: [src/lib/auth.ts, src/middleware/auth.ts]
autonomous: true

# Plano 02 - Funcionalidades protegidas (precisa de auth)
wave: 2
depends_on: ["01"]
files_modified: [src/features/dashboard.ts]
autonomous: true
```

Plano 02 na Onda 2 espera pelo Plano 01 na Onda 1 - dependência genuína dos tipos/middleware de auth.

**Plano com checkpoint:**

```yaml
# Plano 03 - UI com verificação
wave: 3
depends_on: ["01", "02"]
files_modified: [src/components/Dashboard.tsx]
autonomous: false  # Tem checkpoint:human-verify
```

Onda 3 roda após Ondas 1 e 2. Pausa no checkpoint, orquestrador apresenta ao usuário, retoma sob aprovação.

</parallel_examples>

---

## Seção de Contexto

**Contexto ciente de paralelismo:**

```markdown
<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# Inclua referências a SUMMARY apenas se genuinamente necessário:
# - Este plano importa tipos de plano anterior
# - Plano anterior tomou decisão que afeta este plano
# - Saída do plano anterior é entrada para este plano
#
# Planos independentes NÃO precisam de referências a SUMMARY anteriores.
# NÃO encadeie reflexivamente: 02 ref 01, 03 ref 02...

@src/fonte/relevante.ts
</context>
```

**Padrão ruim (cria dependências falsas):**
```markdown
<context>
@.planning/phases/03-features/03-01-SUMMARY.md  # Só porque é anterior
@.planning/phases/03-features/03-02-SUMMARY.md  # Encadeamento reflexivo
</context>
```

---

## Orientação de Escopo

**Dimensionamento de plano:**

- 2-3 tarefas por plano
- ~50% de uso de contexto máximo
- Fases complexas: Múltiplos planos focados, não um plano grande

**Quando dividir:**

- Subsistemas diferentes (auth vs API vs UI)
- >3 tarefas
- Risco de estouro de contexto
- Candidatos a TDD - planos separados

**Fatias verticais são preferidas:**

```
PREFERIR: Plano 01 = Usuário (modelo + API + UI)
          Plano 02 = Produto (modelo + API + UI)

EVITAR:   Plano 01 = Todos os modelos
          Plano 02 = Todas as APIs
          Plano 03 = Todas as UIs
```

---

## Planos TDD

Funcionalidades TDD recebem planos dedicados com `type: tdd`.

**Heurística:** Você pode escrever `expect(fn(input)).toBe(output)` antes de escrever `fn`?
→ Sim: Crie um plano TDD
→ Não: Tarefa padrão em plano padrão

Veja `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/tdd.md` para estrutura de plano TDD.

---

## Tipos de Tarefa

| Tipo | Usar Para | Autonomia |
|------|-----------|-----------|
| `auto` | Tudo que o Claude pode fazer independentemente | Totalmente autônomo |
| `checkpoint:human-verify` | Verificação visual/funcional | Pausa, retorna ao orquestrador |
| `checkpoint:decision` | Escolhas de implementação | Pausa, retorna ao orquestrador |
| `checkpoint:human-action` | Passos manuais verdadeiramente inevitáveis (raro) | Pausa, retorna ao orquestrador |

**Comportamento de checkpoint em execução paralela:**
- Plano roda até o checkpoint
- Agente retorna com detalhes do checkpoint + agent_id
- Orquestrador apresenta ao usuário
- Usuário responde
- Orquestrador retoma agente com `resume: agent_id`

---

## Exemplos

**Plano paralelo autônomo:**

```markdown
---
phase: 03-features
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [src/features/user/model.ts, src/features/user/api.ts, src/features/user/UserList.tsx]
autonomous: true
---

<objective>
Implementar funcionalidade completa de Usuário como fatia vertical.

Purpose: Gestão de usuários auto-contida que pode rodar em paralelo com outras funcionalidades.
Output: Modelo de Usuário, endpoints de API e componentes de UI.
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
</context>

<tasks>
<task type="auto">
  <name>Tarefa 1: Criar modelo de Usuário</name>
  <files>src/features/user/model.ts</files>
  <action>Definir tipo User com id, email, name, createdAt. Exportar interface TypeScript.</action>
  <verify>tsc --noEmit passa</verify>
  <done>Tipo User exportado e utilizável</done>
</task>

<task type="auto">
  <name>Tarefa 2: Criar endpoints de API de Usuário</name>
  <files>src/features/user/api.ts</files>
  <action>GET /users (lista), GET /users/:id (individual), POST /users (criar). Usar tipo User do modelo.</action>
  <verify>Testes de fetch passam para todos os endpoints</verify>
  <done>Todas as operações CRUD funcionam</done>
</task>
</tasks>

<verification>
- [ ] npm run build funciona
- [ ] Endpoints de API respondem corretamente
</verification>

<success_criteria>
- Todas as tarefas concluídas
- Funcionalidade de Usuário funciona de ponta a ponta
</success_criteria>

<output>
Após conclusão, crie `.planning/phases/03-features/03-01-SUMMARY.md`
</output>
```

**Plano com checkpoint (não-autônomo):**

```markdown
---
phase: 03-features
plan: 03
type: execute
wave: 2
depends_on: ["03-01", "03-02"]
files_modified: [src/components/Dashboard.tsx]
autonomous: false
---

<objective>
Construir dashboard com verificação visual.

Purpose: Integrar funcionalidades de usuário e produto em visão unificada.
Output: Componente de dashboard funcionando.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/execute-plan.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/resumo.md
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/phases/03-features/03-01-SUMMARY.md
@.planning/phases/03-features/03-02-SUMMARY.md
</context>

<tasks>
<task type="auto">
  <name>Tarefa 1: Construir layout do Dashboard</name>
  <files>src/components/Dashboard.tsx</files>
  <action>Criar grid responsivo com componentes UserList e ProductList. Usar Tailwind para estilização.</action>
  <verify>npm run build funciona</verify>
  <done>Dashboard renderiza sem erros</done>
</task>

<!-- Padrão de checkpoint: Claude inicia servidor, usuário visita URL. Veja checkpoints.md para padrões completos. -->
<task type="auto">
  <name>Iniciar servidor de desenvolvimento</name>
  <action>Executar `npm run dev` em background, aguardar prontidão</action>
  <verify>fetch http://localhost:3000 retorna 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard - servidor em http://localhost:3000</what-built>
  <how-to-verify>Visite localhost:3000/dashboard. Verifique: grid desktop, empilhamento mobile, sem problemas de scroll.</how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva os problemas</resume-signal>
</task>
</tasks>

<verification>
- [ ] npm run build funciona
- [ ] Verificação visual aprovada
</verification>

<success_criteria>
- Todas as tarefas concluídas
- Usuário aprovou layout visual
</success_criteria>

<output>
Após conclusão, crie `.planning/phases/03-features/03-03-SUMMARY.md`
</output>
```

---

## Anti-Padrões

**Ruim: Encadeamento reflexivo de dependências**
```yaml
depends_on: ["03-01"]  # Só porque 01 vem antes de 02
```

**Ruim: Agrupamento por camada horizontal**
```
Plano 01: Todos os modelos
Plano 02: Todas as APIs (depende de 01)
Plano 03: Todas as UIs (depende de 02)
```

**Ruim: Flag de autonomia ausente**
```yaml
# Tem checkpoint mas sem autonomous: false
depends_on: []
files_modified: [...]
# autonomous: ???  <- Ausente!
```

**Ruim: Tarefas vagas**
```xml
<task type="auto">
  <name>Configurar autenticação</name>
  <action>Adicionar auth ao app</action>
</task>
```

**Ruim: read_first ausente (executor modifica arquivos que não leu)**
```xml
<task type="auto">
  <name>Atualizar config do banco de dados</name>
  <files>src/config/database.ts</files>
  <!-- Sem read_first! Executor não sabe o estado atual ou convenções -->
  <action>Atualizar a config do banco para corresponder às configurações de produção</action>
</task>
```

**Ruim: Critérios de aceitação vagos (não verificáveis)**
```xml
<acceptance_criteria>
  - Config está configurada corretamente
  - Conexão com banco funciona corretamente
</acceptance_criteria>
```

**Bom: Concreto com read_first + critérios verificáveis**
```xml
<task type="auto">
  <name>Atualizar config do banco para connection pooling</name>
  <files>src/config/database.ts</files>
  <read_first>src/config/database.ts, .env.example, docker-compose.yml</read_first>
  <action>Adicionar configuração de pool: min=2, max=20, idleTimeoutMs=30000. Adicionar config SSL: rejectUnauthorized=true quando NODE_ENV=production. Adicionar entrada .env.example: DATABASE_POOL_MAX=20.</action>
  <acceptance_criteria>
    - database.ts contém "max: 20" e "idleTimeoutMillis: 30000"
    - database.ts contém condicional SSL em NODE_ENV
    - .env.example contém DATABASE_POOL_MAX
  </acceptance_criteria>
</task>
```

---

## Diretrizes

- Sempre use estrutura XML para parsing do Claude
- Inclua `wave`, `depends_on`, `files_modified`, `autonomous` em todo plano
- Prefira fatias verticais a camadas horizontais
- Apenas referencie SUMMARYs anteriores quando genuinamente necessário
- Agrupe checkpoints com tarefas auto relacionadas no mesmo plano
- 2-3 tarefas por plano, ~50% de contexto máx

---

## Configuração do Usuário (Serviços Externos)

Quando um plano introduz serviços externos que requerem configuração humana, declare no frontmatter:

```yaml
user_setup:
  - service: stripe
    why: "Processamento de pagamento requer chaves de API"
    env_vars:
      - name: STRIPE_SECRET_KEY
        source: "Stripe Dashboard → Developers → API keys → Secret key"
      - name: STRIPE_WEBHOOK_SECRET
        source: "Stripe Dashboard → Developers → Webhooks → Signing secret"
    dashboard_config:
      - task: "Criar endpoint de webhook"
        location: "Stripe Dashboard → Developers → Webhooks → Add endpoint"
        details: "URL: https://[seu-dominio]/api/webhooks/stripe"
    local_dev:
      - "stripe listen --forward-to localhost:3000/api/webhooks/stripe"
```

**A regra automação-primeiro:** `user_setup` contém APENAS o que o Claude literalmente não pode fazer:
- Criação de conta (requer cadastro humano)
- Recuperação de secrets (requer acesso ao dashboard)
- Configuração de dashboard (requer humano no navegador)

**NÃO incluído:** Instalação de pacotes, alterações de código, criação de arquivos, comandos CLI que o Claude pode executar.

**Resultado:** Execute-plan gera `{fase}-USER-SETUP.md` com checklist para o usuário.

Veja `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/configuracao-usuario.md` para schema completo e exemplos

---

## Must-Haves (Verificação Goal-Backward)

O campo `must_haves` define o que deve ser VERDADEIRO para o objetivo da fase ser alcançado. Derivado durante planejamento, verificado após execução.

**Estrutura:**

```yaml
must_haves:
  truths:
    - "Usuário pode ver mensagens existentes"
    - "Usuário pode enviar uma mensagem"
    - "Mensagens persistem após atualização"
  artifacts:
    - path: "src/components/Chat.tsx"
      provides: "Renderização da lista de mensagens"
      min_lines: 30
    - path: "src/app/api/chat/route.ts"
      provides: "Operações CRUD de mensagens"
      exports: ["GET", "POST"]
    - path: "prisma/schema.prisma"
      provides: "Modelo de Message"
      contains: "model Message"
  key_links:
    - from: "src/components/Chat.tsx"
      to: "/api/chat"
      via: "fetch no useEffect"
      pattern: "fetch.*api/chat"
    - from: "src/app/api/chat/route.ts"
      to: "prisma.message"
      via: "query de banco de dados"
      pattern: "prisma\\.message\\.(find|create)"
```

**Descrição dos campos:**

| Campo | Propósito |
|-------|-----------|
| `truths` | Comportamentos observáveis da perspectiva do usuário. Cada um deve ser testável. |
| `artifacts` | Arquivos que devem existir com implementação real. |
| `artifacts[].path` | Caminho do arquivo relativo à raiz do projeto. |
| `artifacts[].provides` | O que este artefato entrega. |
| `artifacts[].min_lines` | Opcional. Mínimo de linhas para ser considerado substantivo. |
| `artifacts[].exports` | Opcional. Exports esperados para verificar. |
| `artifacts[].contains` | Opcional. Padrão que deve existir no arquivo. |
| `key_links` | Conexões críticas entre artefatos. |
| `key_links[].from` | Artefato de origem. |
| `key_links[].to` | Artefato ou endpoint de destino. |
| `key_links[].via` | Como se conectam (descrição). |
| `key_links[].pattern` | Opcional. Regex para verificar que a conexão existe. |

**Por que isto importa:**

Conclusão de tarefa ≠ Alcance de objetivo. Uma tarefa "criar componente de chat" pode ser concluída criando um placeholder. O campo `must_haves` captura o que deve realmente funcionar, permitindo que a verificação detecte lacunas antes que se acumulem.

**Fluxo de verificação:**

1. Plan-phase deriva must_haves do objetivo da fase (goal-backward)
2. Must_haves escritos no frontmatter do PLAN.md
3. Execute-phase roda todos os planos
4. Subagente de verificação verifica must_haves contra o codebase
5. Lacunas encontradas → planos de correção criados → executar → re-verificar
6. Todos must_haves passam → fase concluída

Veja `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/verify-phase.md` para lógica de verificação.
