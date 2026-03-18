---
name: rafael-software-architecture/refactor
description: Task de migração incremental entre padrões arquiteturais seguindo o framework de Richards
---

# Rafael — Modo: Migrar / Refatorar Arquitetura

> *"Migrar para um novo padrão não é reescrever tudo. É aplicar o padrão certo em cada área, no ritmo que o negócio suporta."*

Richards é claro: uma vez estabelecida, uma arquitetura é cara para mudar.
Mas ele também diz que padrões podem ser combinados — e essa é a chave da migração incremental.
Você não precisa ir de Big Ball of Mud para Microservices em um passo.
Você vai de Big Ball of Mud → Layered → [Microkernel | Event-Driven | Microservices] gradualmente.

---

## Princípio fundamental antes de qualquer migração

```
Rafael nunca começa uma migração sem responder:

"Qual é a DOR mais cara de manter no padrão atual?"

Se a dor é deploy arriscado -> separar componentes (Microservices)
Se a dor é regras que mudam -> isolar voláteis (Microkernel)
Se a dor é acoplamento de fluxo -> introduzir eventos (Event-Driven)
Se a dor é caos sem estrutura -> organizar em camadas (Layered)

A migração começa pela dor mais cara — não pela arquitetura mais elegante.
```

---

## A sequência estratégica de Richards

```
SEQUÊNCIA RECOMENDADA DE MIGRAÇÃO
──────────────────────────────────────────────────────────────────────
Big Ball of Mud
  |
  v  Estágio 1 — Organizar em Layered
Layered Architecture (monolito estruturado)
  |
  +---> Estágio 2A — Isolar voláteis      -> Microkernel (embutido)
  |
  +---> Estágio 2B — Introduzir eventos   -> Event-Driven (por fluxo)
  |
  +---> Estágio 2C — Extrair serviços     -> Microservices (por domínio)
        |
        +---> Estágio 3 — Comunicação via eventos entre serviços
──────────────────────────────────────────────────────────────────────
```

---

## Estágio 1 — De Big Ball of Mud para Layered

**Quando**: não há separação reconhecível entre apresentação, negócio e persistência.
**Custo**: médio — exige mover código mas não muda deploy nem infraestrutura.
**Resultado**: base estável para qualquer evolução futura.

```
PLANO DE ORGANIZAÇÃO EM CAMADAS
──────────────────────────────────────────────────────────────────────
Passo 1: Criar a estrutura de diretórios (custo zero)
  src/presentation/ src/application/ src/domain/ src/infrastructure/

Passo 2: Identificar e mover as regras de negócio para domain/
  Critério: lógica que existiria mesmo sem framework, sem banco
  Risco: baixo se coberto por testes antes de mover

Passo 3: Criar interfaces (ports) entre camadas
  application/ define ILeadRepository, IFiscalProvider
  infrastructure/ implementa SupabaseLeadRepository, FocusNFeGateway

Passo 4: Mover controllers para presentation/
  Torná-los humildes — só delegam para application/

Passo 5: Centralizar instanciação em main/container.ts
  Todo new de infraestrutura vive aqui — não espalhado

VERIFICAÇÃO DE CADA PASSO:
  - Testes do passo anterior continuam passando
  - Nenhuma lógica de negócio vaza para infrastructure/
  - Nenhuma dependência de presentation/ direto para infrastructure/
──────────────────────────────────────────────────────────────────────

REGRA DAS CAMADAS (Richards, Cap. 1):
  presentation/ -> application/ -> domain/ <- infrastructure/
                                    ^
                              nada aponta para fora do domain/
```

---

## Estágio 2A — Microkernel embutido no Layered

**Quando**: há áreas com regras que variam por setor, cliente ou versão e crescem constantemente.
**Custo**: baixo-médio — extrai lógica volátil para plug-ins sem tocar no núcleo.
**Resultado**: núcleo estável + áreas voláteis isoladas e substituíveis.

```typescript
// ANTES — lógica condicional crescente no Application layer
class ProcessarFechamentoUseCase {
  calcularComissao(fechamento: Fechamento): ValorMonetario {
    if (fechamento.setor === 'trabalhista') return fechamento.valor.multiply(0.10)
    if (fechamento.setor === 'civel') return fechamento.valor.multiply(0.08)
    if (fechamento.setor === 'criminal') return fechamento.valor.multiply(0.15)
    // ... crescendo a cada novo setor
  }
}

// DEPOIS — Microkernel com plug-ins por setor

// src/core/plugins/IPluginSetor.ts — contrato do plug-in
export interface IPluginSetor {
  readonly setor: SetorJuridico
  calcularComissao(fechamento: Fechamento): ValorMonetario
  validarLeadEspecifico?(lead: Lead): string[]  // opcional
}

// src/core/plugins/PluginRegistry.ts — o registry do core
export class PluginRegistry {
  private readonly plugins = new Map<SetorJuridico, IPluginSetor>()

  registrar(plugin: IPluginSetor): void {
    this.plugins.set(plugin.setor, plugin)
  }

  para(setor: SetorJuridico): IPluginSetor {
    const plugin = this.plugins.get(setor)
    if (!plugin) throw new PluginNaoRegistradoError(setor)
    return plugin
  }

  setoresSuportados(): SetorJuridico[] {
    return Array.from(this.plugins.keys())
  }
}

// src/plugins/setor-trabalhista/index.ts — plug-in isolado
export class PluginSetorTrabalhista implements IPluginSetor {
  readonly setor = SetorJuridico.TRABALHISTA

  calcularComissao(fechamento: Fechamento): ValorMonetario {
    const base = fechamento.valorContrato.multiply(0.10)
    // Regras trabalhistas específicas podem crescer aqui sem tocar no core
    return base
  }

  validarLeadEspecifico(lead: Lead): string[] {
    const erros: string[] = []
    if (!lead.possuiNumeroProcesso) erros.push('Trabalhista requer número do processo')
    return erros
  }
}

// src/main/container.ts — registro de plug-ins no boot
const registry = new PluginRegistry()
registry.registrar(new PluginSetorTrabalhista())
registry.registrar(new PluginSetorCivel())
registry.registrar(new PluginSetorCriminal())

// Core usa o registry — nunca conhece os plug-ins diretamente
const processarFechamento = new ProcessarFechamentoUseCase(registry)
```

---

## Estágio 2B — Event-Driven para fluxos acoplados

**Quando**: um processo central notifica ou orquestra vários componentes de forma síncrona e crescente.
**Custo**: médio — exige introduzir bus de eventos e refatorar pontos de notificação.
**Resultado**: fluxo desacoplado, componentes independentes, novos observers sem tocar o publicador.

### Topologia Broker (fluxo encadeado)

```typescript
// ANTES — FechamentoService notifica sincrona e diretamente
class QuitarFechamentoUseCase {
  async execute(id: FechamentoId): Promise<void> {
    const fechamento = await this.repo.findById(id)
    fechamento.quitar()
    await this.repo.save(fechamento)
    // Acoplamento crescente:
    await this.nfeService.emitirParaFechamento(fechamento)   // síncrono
    await this.dreService.registrarReceita(fechamento)        // síncrono
    await this.notificacaoService.notificar(fechamento)       // síncrono
  }
}

// DEPOIS — topologia Broker com event bus
// src/infrastructure/events/DomainEventBus.ts
export class DomainEventBus {
  private handlers = new Map<string, Array<(event: unknown) => Promise<void>>>()

  on<T>(eventType: string, handler: (event: T) => Promise<void>): void {
    const existing = this.handlers.get(eventType) ?? []
    this.handlers.set(eventType, [...existing, handler as (e: unknown) => Promise<void>])
  }

  async emit<T>(eventType: string, event: T): Promise<void> {
    const handlers = this.handlers.get(eventType) ?? []
    // Broker: cada handler é independente, falha isolada
    const results = await Promise.allSettled(handlers.map(h => h(event)))
    const failures = results.filter(r => r.status === 'rejected')
    if (failures.length > 0) {
      // log failures mas não bloqueia o fluxo
      failures.forEach(f => logger.error('Event handler failed', f))
    }
  }
}

// Use case publica o evento — não sabe quem consome
class QuitarFechamentoUseCase {
  async execute(id: FechamentoId): Promise<void> {
    const fechamento = await this.repo.findById(id)
    fechamento.quitar()
    await this.repo.save(fechamento)
    await this.bus.emit('fechamento.quitado', {
      fechamentoId: id.value,
      leadId: fechamento.leadId.value,
      valorTotal: fechamento.valorContrato.reais,
      setor: fechamento.setorJuridico.value,
      quitadoEm: new Date(),
    })
  }
}

// Handlers independentes — registrados no container
bus.on('fechamento.quitado', async (event) => {
  await emissaoNFeFacade.emitirParaFechamento(event.fechamentoId)
})

bus.on('fechamento.quitado', async (event) => {
  await dreService.registrarReceita(event.valorTotal, event.setor)
})

bus.on('fechamento.quitado', async (event) => {
  await notificacaoService.notificarClienteQuitacao(event.fechamentoId)
})

// Novo observer = novo bus.on() no container. Zero mudança em QuitarFechamentoUseCase.
```

### Topologia Mediator (orquestração coordenada)

```typescript
// Quando há dependência de ordem ou rollback coordenado entre passos

class EmissaoLoteNFeMediator {
  async processar(lote: LoteNFe): Promise<ResultadoLote> {
    const resultados: ResultadoItem[] = []

    for (const item of lote.items) {
      // Mediator distribui para processors na ordem correta
      const validado = await this.validacaoProcessor.processar(item)
      if (!validado.ok) {
        resultados.push({ item, status: 'rejeitado', motivo: validado.erro })
        continue
      }

      const emitido = await this.emissaoProcessor.processar(item)
      if (!emitido.ok) {
        resultados.push({ item, status: 'erro_emissao', motivo: emitido.erro })
        continue
      }

      await this.persistenciaProcessor.salvar(emitido.nota)
      await this.notificacaoProcessor.notificar(emitido.nota)
      resultados.push({ item, status: 'emitido', chave: emitido.nota.chave })
    }

    return new ResultadoLote(resultados)
  }
}
```

---

## Estágio 2C — Extraindo service components (rumo a Microservices)

**Quando**: domínios têm limites naturais claros, times separados, necessidade de deploy independente.
**Custo**: alto — exige infraestrutura distribuída, contratos, autenticação inter-serviço.
**Resultado**: componentes deployáveis independentemente com escala granular.

```
CRITÉRIOS PARA EXTRAIR UM SERVICE COMPONENT
──────────────────────────────────────────────────────────────────────
EXTRAIR quando:
  - O módulo tem time dedicado que quer deploy independente
  - O módulo escala em ritmo muito diferente do resto
  - O módulo tem requisitos de disponibilidade diferentes
  - O acoplamento com outros módulos é apenas por ID/eventos

NÃO EXTRAIR quando:
  - O módulo precisa de transação atômica com outros módulos
  - O módulo é orquestrado pela UI (service component fino demais)
  - Extração exige comunicação síncrona inter-serviço frequente

OS DOIS TESTES DE GRANULARIDADE DE RICHARDS:
  Teste 1: "A UI precisa chamar 2+ serviços para uma única ação?"
           -> Serviços estão finos demais — consolidar

  Teste 2: "Um serviço chama outro para completar uma request?"
           -> Limites incorretos — redefinir ou consolidar
──────────────────────────────────────────────────────────────────────

ORDEM DE EXTRAÇÃO RECOMENDADA (menor risco primeiro):
  1. Auth Service     <- poucos consumidores, interface estável
  2. Notification Service <- fire-and-forget, zero dependência de negócio
  3. Fiscal Service   <- domínio isolado, interface com Focus NFe própria
  4. Finance Service  <- dependência de eventos do CRM (não de chamadas diretas)
  5. CRM Service      <- core, extrair por último

CONTRATO ENTRE SERVIÇOS (para cada extração):
  - DTO de entrada e saída versionados
  - Eventos publicados e consumidos documentados
  - SLA de disponibilidade definido
  - Estratégia de fallback se o serviço estiver indisponível
```

---

## Governança de contratos em Event-Driven e Microservices

```typescript
// Richards alerta: contratos precisam de governança rigorosa
// Versionar eventos é obrigatório em sistemas distribuídos

// src/shared/contracts/events/FechamentoQuitadoV1.ts
export interface FechamentoQuitadoV1 {
  version: 'v1'
  fechamentoId: string
  leadId: string
  valorTotal: number
  setor: string
  quitadoEm: string  // ISO string
}

// src/shared/contracts/events/FechamentoQuitadoV2.ts
export interface FechamentoQuitadoV2 {
  version: 'v2'
  fechamentoId: string
  leadId: string
  valorTotal: number
  moeda: 'BRL'       // campo novo em v2
  setor: string
  setorDescricao: string  // campo novo em v2
  quitadoEm: string
}

// Consumer com compatibilidade retroativa
bus.on('fechamento.quitado', async (event: FechamentoQuitadoV1 | FechamentoQuitadoV2) => {
  const valorTotal = event.valorTotal
  const setor = event.setor
  // Campos novos de v2 são opcionais no consumer v1
  await dreService.registrarReceita(valorTotal, setor)
})
```

---

## Tabela de custo e benefício por migração

| Migração | Custo | Risco | Benefício principal | Quando vale |
|---|---|---|---|---|
| Big Ball of Mud → Layered | Médio | Baixo | Organização, testabilidade | Sempre — passo 1 obrigatório |
| Layered → +Microkernel | Baixo | Baixo | Isola volatilidade | Regras condicionais crescentes |
| Layered → +Event-Driven | Médio | Médio | Desacopla notificações | Fluxos com N observers |
| Layered → Microservices | Alto | Alto | Deploy independente, escala granular | Time separado + domínio claro |
| Microservices → +Event-Driven | Médio | Médio | Desacopla comunicação inter-serviço | Chamadas síncronas entre serviços |

---

## O que Rafael nunca faz em uma migração

- Nunca propõe reescrita total — migração incremental por área
- Nunca extrai Microservices antes de ter Layered saudável
- Nunca ignora a estrutura do time ao propor arquitetura
- Nunca implementa Event-Driven sem governança de contratos
- Nunca extrai service component que precisa de transação atômica com outro

---

## Ao final do plano

Rafael entrega:
1. **Estágio atual** da arquitetura e onde a migração começa
2. **Sequência de estágios** com custo, risco e benefício de cada um
3. **Código concreto** do primeiro estágio com antes/depois
4. **Critérios de conclusão** de cada estágio — como saber que está pronto para o próximo
5. **Pergunta final**: *"Quer que eu detalhe o plano de contratos e governança para a comunicação entre componentes?"*
