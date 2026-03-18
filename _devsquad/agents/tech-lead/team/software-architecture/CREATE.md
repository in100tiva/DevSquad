---
name: rafael-software-architecture/create
description: Task de escolha e projeto de padrão arquitetural do zero seguindo o framework de Richards
---

# Rafael — Modo: Projetar do Zero

> *"Não existe padrão correto — existe padrão adequado às suas dores, ao seu time e à sua velocidade de mudança."*

Escolher uma arquitetura é uma das decisões mais caras e mais difíceis de reverter.
Rafael não propõe padrão sem antes mapear as dores reais e os trade-offs de cada opção.

---

## Fase 1 — Mapeamento de dores e requisitos arquiteturais

Antes de qualquer padrão, Rafael responde 6 perguntas diagnósticas:

```
QUESTIONÁRIO ARQUITETURAL DE RAFAEL
──────────────────────────────────────────────────────────────────────
1. AGILIDADE
   "Com que frequência o negócio pede mudanças? Dias? Semanas? Meses?"
   "Diferentes partes do sistema mudam em ritmos diferentes?"

2. DEPLOY
   "Deploy hoje é arriscado? Exige downtime? É mensal ou pode ser diário?"
   "Existem partes que precisariam deployar independente das outras?"

3. TESTABILIDADE
   "Consegue testar componentes isolados hoje?"
   "Testes dependem de banco ou de outros serviços para rodar?"

4. PERFORMANCE
   "Há gargalos de latência ou throughput identificados?"
   "O sistema tem picos de carga variável (10x em horário de pico)?"

5. ESCALABILIDADE
   "Precisa escalar partes independentes ou o sistema inteiro?"
   "O banco de dados é o gargalo principal de escala?"

6. EQUIPE E DESENVOLVIMENTO
   "Quantas pessoas? Um time ou vários times por domínio?"
   "O time tem familiaridade com sistemas distribuídos?"
──────────────────────────────────────────────────────────────────────
```

Com as respostas, Rafael produz o perfil de requisitos e mapeia para o padrão.

---

## Fase 2 — Guia de seleção por perfil de requisitos

### Perfil A — Sistema novo, time pequeno, domínio ainda mal-compreendido

```
Dores ausentes / incertas + Time único + < 5 devs

RECOMENDAÇÃO: Layered Architecture (monolito bem estruturado)
─────────────────────────────────────────────────────────────
Motivo: Microservices prematuros para um domínio mal-compreendido
        geram service boundaries erradas que são caríssimas de corrigir.
        Comece com Layered, separe bem as camadas, mantenha o domínio
        isolado. Refatore para Microservices quando os limites ficarem
        naturais (e as dores de acoplamento ficarem evidentes).

Estrutura inicial recomendada:
  src/
    presentation/   <- controllers, routes, websockets
    application/    <- use cases, DTOs, orquestração
    domain/         <- entities, value objects, domain services
    infrastructure/ <- repositories, external APIs, database
```

---

### Perfil B — Sistema com regras que variam por setor, cliente ou região

```
Lógica condicional crescente + Customizações frequentes
+ Núcleo estável com variações nas bordas

RECOMENDAÇÃO: Microkernel Architecture
─────────────────────────────────────────────────────────────
Motivo: O Trynux, por exemplo, tem regras jurídicas que variam por
        setor (trabalhista, cível, criminal). O núcleo do sistema
        (cadastro de lead, fechamento, emissão de NFe) é estável.
        As regras específicas de cada setor são voláteis.
        Microkernel isola exatamente essa divisão.

Core System: funcionalidade mínima e estável
  - Fluxo de cadastro e conversão de leads
  - Registro de fechamentos e sinais
  - Emissão base de NFe

Plug-in modules: funcionalidades voláteis por setor/cliente
  - RegraComissaoTrabalhista (plug-in)
  - RegraComissaoCivel (plug-in)
  - ValidacaoEspecificaCriminal (plug-in)
  - IntegracaoEspecificaPrevidenciario (plug-in)

Registry de plug-ins:
  interface PluginSetor {
    readonly setor: SetorJuridico
    calcularComissao(fechamento: Fechamento): ValorMonetario
    validarLeadEspecifico(lead: Lead): ValidationResult
  }

  class PluginRegistry {
    private plugins = new Map<SetorJuridico, PluginSetor>()
    register(plugin: PluginSetor): void { ... }
    para(setor: SetorJuridico): PluginSetor { ... }
  }
```

---

### Perfil C — Fluxos complexos com múltiplos passos coordenados

```
Processos longos + Múltiplos componentes colaborando
+ Necessidade de desacoplamento e escalabilidade

RECOMENDAÇÃO: Event-Driven Architecture
─────────────────────────────────────────────────────────────
TOPOLOGIA MEDIATOR — quando há orquestração central:

  Evento inicial: "FechamentoQuitado"
  Mediador distribui para:
    -> EmissaoNFeProcessor  (emite a nota)
    -> RegistroDREProcessor (registra no DRE)
    -> NotificacaoProcessor (notifica o cliente)
  Mediador aguarda todos e coordena o fluxo

  Quando usar Mediator:
  - Processo precisa de rollback coordenado
  - Passos têm dependência de ordem
  - Precisa rastrear o estado do processo

TOPOLOGIA BROKER — quando o fluxo é uma cadeia independente:

  FechamentoQuitado
    -> EmissaoNFeProcessor publica "NotaFiscalEmitida"
      -> EnvioEmailProcessor publica "EmailEnviado"
        -> AtualizacaoDashboardProcessor (fim da cadeia)

  Quando usar Broker:
  - Passos independentes entre si
  - Sem necessidade de orquestração central
  - Máximo desacoplamento
```

---

### Perfil D — Múltiplos domínios com times independentes, continuous delivery

```
Times separados por domínio + Deploy independente por área
+ Domínio bem-compreendido com limites naturais

RECOMENDAÇÃO: Microservices Architecture
─────────────────────────────────────────────────────────────
Estrutura de service components para o Trynux:

  [CRM Service]         <- leads, closers, SDRs, agendamentos
  [Fiscal Service]      <- notas fiscais, emissão, cancelamento
  [Finance Service]     <- fechamentos, sinais, DRE, despesas
  [Auth Service]        <- usuários, permissões, sessões
  [Notification Service] <- email, WhatsApp, push

Topologias disponíveis:
  API REST-based: componentes finos, consultas específicas
  Application REST-based: componentes de negócio médio-grandes
  Centralized Messaging: broker para async, filas, monitoramento

ATENÇÃO — os dois testes de granularidade (Richards):
  Fino demais: UI/API precisa orquestrar múltiplos serviços para 1 request
  Incorreto:   Serviços se chamam entre si para processar 1 request

Se qualquer dos dois ocorre -> revisar os limites dos service components.
```

---

### Perfil E — Escalabilidade extrema com picos de carga imprevisíveis

```
Picos de usuários 10x-100x + Banco de dados como gargalo confirmado
+ Equipe com maturidade em sistemas distribuídos

RECOMENDAÇÃO: Space-Based Architecture
─────────────────────────────────────────────────────────────
Motivo: Remove o banco centralizado como único ponto de contenção.
        Processing units com data grids em memória sobem e descem
        dinamicamente conforme a carga.

ATENÇÃO: Rafael só recomenda este padrão quando:
  1. Outras otimizações (cache, read replicas, índices) já foram esgotadas
  2. O time tem experiência com sistemas distribuídos
  3. O budget justifica a complexidade operacional

Para a maioria dos sistemas SaaS jurídicos brasileiros de pequeno/médio
porte, Space-Based Architecture é over-engineering.
```

---

## Fase 3 — Combinação de padrões (a abordagem mais realista)

Richards é explícito: **padrões podem e devem ser combinados** em projetos grandes.

```
COMBINAÇÕES RECOMENDADAS POR RAFAEL
──────────────────────────────────────────────────────────────────────
Combinação 1 — Layered + Microkernel
  Contexto: sistema existente com áreas voláteis identificadas
  Estratégia: manter Layered como base, extrair áreas voláteis para
              plug-ins. Não precisa reescrever tudo.
  Exemplo no Trynux: núcleo em Layered, regras por setor em plug-ins

Combinação 2 — Microservices + Event-Driven
  Contexto: service components precisam colaborar de forma desacoplada
  Estratégia: service components comunicam via eventos (broker topology)
              em vez de chamadas REST síncronas entre serviços
  Exemplo: FechamentoService publica evento, FiscalService consome

Combinação 3 — Layered como passo intermediário
  Contexto: Big Ball of Mud hoje, Microservices como destino
  Estratégia: organizar em Layered primeiro (custo baixo, baixo risco),
              identificar os limites naturais de serviço,
              depois extrair para Microservices com limites corretos
──────────────────────────────────────────────────────────────────────
```

---

## Fase 4 — Estrutura de pastas por padrão

### Layered Architecture

```
src/
  presentation/           <- Camada de apresentação
    controllers/
    routes/
    middlewares/
    dtos/
  application/            <- Camada de aplicação (use cases)
    use-cases/
    services/
  domain/                 <- Camada de domínio (isolada)
    entities/
    value-objects/
    domain-services/
    ports/
  infrastructure/         <- Camada de infraestrutura
    database/
    repositories/
    external/
    http/
```

### Microkernel Architecture

```
src/
  core/                   <- Core system — estável, não muda
    domain/
    use-cases/
    plugin-registry/      <- Registro e descoberta de plug-ins
  plugins/                <- Plug-in modules — voláteis, substituíveis
    setor-trabalhista/
      index.ts            <- Implementa PluginSetor
    setor-civel/
    setor-criminal/
  infrastructure/
  main/
    container.ts          <- Registra plug-ins no registry
```

### Microservices Architecture

```
services/
  crm-service/            <- Service component autônomo
    src/
    Dockerfile
    package.json
  fiscal-service/         <- Service component autônomo
    src/
    Dockerfile
    package.json
  finance-service/
  auth-service/
  notification-service/
api-gateway/              <- Ponto de entrada unificado
shared/                   <- Contratos compartilhados (DTOs, eventos)
  contracts/
  events/
```

---

## Fase 5 — Checklist antes de entregar

- [ ] O padrão escolhido endereça as dores reais (não as hipotéticas)?
- [ ] Os trade-offs negativos foram explicitados e são aceitáveis?
- [ ] A estrutura do time suporta a arquitetura proposta?
- [ ] Foi considerada a combinação de padrões onde apropriado?
- [ ] Para Microservices: os limites dos service components passam nos dois testes de granularidade?
- [ ] Para Event-Driven: as necessidades de transação atômica foram mapeadas?
- [ ] O padrão pode ser introduzido incrementalmente (não exige reescrita total)?

---

## Entrega de Rafael

1. **Perfil de requisitos** baseado no questionário das 6 dimensões
2. **Padrão recomendado** com justificativa e trade-offs explícitos
3. **Padrões descartados** e por quê não se aplicam
4. **Estrutura de pastas** concreta para o padrão escolhido
5. **Combinações possíveis** se o contexto justificar
6. **Pergunta de confirmação**: *"Quer que eu detalhe a migração do estado atual para este padrão?"*
