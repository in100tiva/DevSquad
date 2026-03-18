---
name: giovana-gof/create
description: Task de criação de código do zero com escolha e aplicação correta de Design Patterns
---

# Giovana — Modo: Criar do Zero

> *"Antes de escrever código, decida o que vai variar. O padrão certo nasce dessa resposta."*

Criar com Design Patterns não é decorar o catálogo e aplicar tudo que conhece.
É identificar o ponto de variação do problema e escolher o padrão que isola exatamente esse ponto.

---

## Fase 1 — Identificar o ponto de variação antes de qualquer código

Giovana faz **uma pergunta central** antes de propor qualquer padrão:

```
"O que neste problema vai mudar com o tempo?"

Resposta -> O que varia            -> Padrão candidato
────────────────────────────────────────────────────────
O algoritmo/regra                  -> Strategy
O estado do objeto                 -> State
Quem recebe uma notificação        -> Observer
Como o objeto é criado             -> Factory Method / Abstract Factory
Quantas responsabilidades extras   -> Decorator
Quais objetos colaboram            -> Mediator
A plataforma/API usada             -> Adapter / Bridge
A sequência de criação             -> Builder
Quem processa a requisição         -> Chain of Responsibility
```

Somente com a resposta clara → Giovana propõe o padrão.

---

## Fase 2 — Guia de seleção por problema

### Problema: criação de objetos espalhada ou complexa

```
Tenho new ConcreteClass() em vários lugares?
  -> Factory Method: deixar subclasse/método decidir qual concreto criar

Tenho famílias de objetos relacionados (ex: NFe + Boleto + Pix)?
  -> Abstract Factory: interface que cria famílias inteiras

Tenho objeto com 8+ parâmetros no construtor, muitos opcionais?
  -> Builder: construção passo a passo com método fluente

Tenho objetos quase idênticos com pequenas variações?
  -> Prototype: clonar e ajustar ao invés de criar do zero
```

**Factory Method — TypeScript:**

```typescript
// Problema: EmissaoService precisa criar diferentes tipos de documento fiscal
// O que varia: o tipo de documento (NFe, NFSe, CTe)
// O que não muda: o fluxo de emissão

// Interface estável (não muda)
interface DocumentoFiscal {
  validar(): void
  serializar(): Record<string, unknown>
  chave: string
}

// Factory Method — a subclasse decide qual concreto criar
abstract class EmissaoService {
  // Template Method + Factory Method combinados
  async emitir(dados: DadosEmissao): Promise<string> {
    const documento = this.criarDocumento(dados)  // <- Factory Method
    documento.validar()
    return await this.enviarParaApi(documento)
  }

  // Subclasse implementa — Factory Method
  protected abstract criarDocumento(dados: DadosEmissao): DocumentoFiscal

  private async enviarParaApi(doc: DocumentoFiscal): Promise<string> { ... }
}

class EmissaoNFe extends EmissaoService {
  protected criarDocumento(dados: DadosEmissao): DocumentoFiscal {
    return new NotaFiscalEletronica(dados)
  }
}

class EmissaoNFSe extends EmissaoService {
  protected criarDocumento(dados: DadosEmissao): DocumentoFiscal {
    return new NotaFiscalServico(dados)
  }
}
```

**Builder — TypeScript (para objetos complexos):**

```typescript
// Problema: criar RelatorioFiltrado com 10 parâmetros opcionais
// Evitar: new Relatorio(setor, status, null, null, new Date(), null, true, 50, 1, 'asc')

class RelatorioFiltradoBuilder {
  private filtros: Partial<FiltrosRelatorio> = {}
  private paginacao = { pagina: 1, limite: 20 }
  private ordem: OrdemRelatorio = { campo: 'created_at', direcao: 'desc' }

  comSetor(setor: SetorJuridico): this {
    this.filtros.setor = setor
    return this
  }

  comStatus(status: LeadStatus): this {
    this.filtros.status = status
    return this
  }

  comPeriodo(inicio: Date, fim: Date): this {
    this.filtros.periodo = { inicio, fim }
    return this
  }

  paginado(pagina: number, limite: number): this {
    this.paginacao = { pagina, limite }
    return this
  }

  ordenadoPor(campo: string, direcao: 'asc' | 'desc'): this {
    this.ordem = { campo, direcao }
    return this
  }

  build(): RelatorioFiltrado {
    return new RelatorioFiltrado(this.filtros, this.paginacao, this.ordem)
  }
}

// Uso — legível, sem parâmetros posicionais confusos
const relatorio = new RelatorioFiltradoBuilder()
  .comSetor(SetorJuridico.TRABALHISTA)
  .comPeriodo(inicioMes, fimMes)
  .paginado(1, 50)
  .ordenadoPor('valor_contrato', 'desc')
  .build()
```

---

### Problema: comportamento que varia por tipo ou estado

**Strategy — TypeScript (algoritmo que varia):**

```typescript
// Problema: cálculo de comissão varia por tipo de contrato e setor
// ERRADO: if/else gigante no CalculoComissaoService
// CERTO: cada regra é um Strategy isolado

interface PoliticaComissao {
  calcular(fechamento: Fechamento): ValorMonetario
}

class ComissaoTrabalhistaSimples implements PoliticaComissao {
  calcular(f: Fechamento): ValorMonetario {
    return f.valorContrato.multiply(0.10)
  }
}

class ComissaoTrabalhistaCompleta implements PoliticaComissao {
  calcular(f: Fechamento): ValorMonetario {
    const base = f.valorContrato.multiply(0.10)
    const bonus = f.valorContrato.greaterThan(ValorMonetario.deReais(5000))
      ? ValorMonetario.deReais(500)
      : ValorMonetario.zero()
    return base.somar(bonus)
  }
}

class ComissaoCivelRecorrente implements PoliticaComissao {
  calcular(f: Fechamento): ValorMonetario {
    return f.valorContrato.multiply(0.08).multiply(f.numeroParcelas)
  }
}

// O contexto usa a política sem saber qual é
class CalculadoraComissao {
  constructor(private readonly politica: PoliticaComissao) {}

  calcular(fechamento: Fechamento): ValorMonetario {
    return this.politica.calcular(fechamento)
  }
}

// Novo tipo de comissão = nova classe. Zero modificação nas existentes.
```

**State — TypeScript (comportamento varia por estado):**

```typescript
// Problema: Lead tem comportamentos radicalmente diferentes por status
// ERRADO: if (lead.status === 'aberto') ... else if (lead.status === 'convertido')...
// CERTO: cada estado encapsula seu próprio comportamento

interface EstadoLead {
  converter(lead: Lead, closer: Closer): void
  perder(lead: Lead, motivo: string): void
  reabrir(lead: Lead): void
  descricao: string
}

class LeadAberto implements EstadoLead {
  descricao = 'Aberto'

  converter(lead: Lead, closer: Closer): void {
    lead.transicionarPara(new LeadConvertido(closer))
    lead.addEvent(new LeadConvertido(lead.id, closer.id))
  }

  perder(lead: Lead, motivo: string): void {
    lead.transicionarPara(new LeadPerdido(motivo))
  }

  reabrir(lead: Lead): void {
    throw new OperacaoInvalidaError('Lead já está aberto')
  }
}

class LeadConvertido implements EstadoLead {
  descricao = 'Convertido'
  constructor(private readonly closer: Closer) {}

  converter(): void { throw new OperacaoInvalidaError('Lead já convertido') }
  perder(): void { throw new OperacaoInvalidaError('Lead convertido não pode ser perdido') }
  reabrir(lead: Lead): void { lead.transicionarPara(new LeadAberto()) }
}
```

---

### Problema: estrutura ou acesso que precisa ser mediado

**Facade — TypeScript (subsistema complexo):**

```typescript
// Problema: emitir uma NFe envolve 5 serviços distintos
// O controller não deve conhecer todos esses detalhes

// Sem Facade — controller conhece tudo:
// await nfeValidador.validar(dados)
// const xml = await nfeSerializer.serializar(dados)
// const assinatura = await nfeSigner.assinar(xml)
// const resposta = await focusNFeClient.enviar(assinatura)
// await nfeRepository.salvar(resposta)

// Com Facade — controller conhece apenas um ponto:
export class EmissaoNFeFacade {
  constructor(
    private readonly validador: NFeValidador,
    private readonly serializer: NFeSerializer,
    private readonly signer: NFeSigner,
    private readonly client: FocusNFeClient,
    private readonly repository: NFeRepository,
  ) {}

  async emitir(dados: DadosEmissaoNFe): Promise<NumeroNota> {
    this.validador.validar(dados)
    const xml = await this.serializer.serializar(dados)
    const assinado = await this.signer.assinar(xml)
    const resposta = await this.client.enviar(assinado)
    await this.repository.salvar(resposta)
    return resposta.numeroNota
  }
}

// Controller fica limpo:
async handleEmitir(req: Request, res: Response) {
  const numero = await this.emissaoFacade.emitir(req.body)
  res.json({ numeroNota: numero.value })
}
```

**Decorator — TypeScript (responsabilidades opcionais):**

```typescript
// Problema: LoggingRepository + CachingRepository + MetricsRepository
// Herança criaria 2^n combinações. Decorator compõe livremente.

interface ILeadRepository {
  findById(id: LeadId): Promise<Lead | null>
  save(lead: Lead): Promise<void>
}

// Decorador base
class LoggingLeadRepository implements ILeadRepository {
  constructor(
    private readonly inner: ILeadRepository,
    private readonly logger: Logger,
  ) {}

  async findById(id: LeadId): Promise<Lead | null> {
    this.logger.info(`findById: ${id.value}`)
    const result = await this.inner.findById(id)
    this.logger.info(`findById resultado: ${result ? 'encontrado' : 'nulo'}`)
    return result
  }

  async save(lead: Lead): Promise<void> {
    this.logger.info(`save: lead ${lead.id.value}`)
    await this.inner.save(lead)
  }
}

class CachingLeadRepository implements ILeadRepository {
  private cache = new Map<string, Lead>()

  constructor(private readonly inner: ILeadRepository) {}

  async findById(id: LeadId): Promise<Lead | null> {
    if (this.cache.has(id.value)) return this.cache.get(id.value)!
    const lead = await this.inner.findById(id)
    if (lead) this.cache.set(id.value, lead)
    return lead
  }

  async save(lead: Lead): Promise<void> {
    this.cache.delete(lead.id.value)  // invalida cache
    await this.inner.save(lead)
  }
}

// Composição livre no container:
const leadRepo = new LoggingLeadRepository(
  new CachingLeadRepository(
    new SupabaseLeadRepository(supabase)
  ),
  logger
)
```

---

### Problema: comunicação e coordenação entre objetos

**Observer — TypeScript (1 muda, N são notificados):**

```typescript
// Problema: quando Fechamento é quitado, Fiscal e Finance precisam agir
// Sem Observer: FechamentoService chama FiscalService e FinanceService diretamente

interface FechamentoObserver {
  onFechamentoQuitado(fechamento: Fechamento): Promise<void>
}

class FechamentoEventEmitter {
  private observers: FechamentoObserver[] = []

  subscribe(observer: FechamentoObserver): void {
    this.observers.push(observer)
  }

  async notificarQuitacao(fechamento: Fechamento): Promise<void> {
    await Promise.all(
      this.observers.map(o => o.onFechamentoQuitado(fechamento))
    )
  }
}

// Observers independentes — não se conhecem
class IniciarEmissaoNFeObserver implements FechamentoObserver {
  async onFechamentoQuitado(f: Fechamento): Promise<void> {
    await this.emissaoFacade.emitirParaFechamento(f.id)
  }
}

class RegistrarReceitaDREObserver implements FechamentoObserver {
  async onFechamentoQuitado(f: Fechamento): Promise<void> {
    await this.dreService.registrarReceita(f.valorContrato, f.setorJuridico)
  }
}

// Novo Observer = nova classe. FechamentoService não sabe quantos existem.
```

**Command — TypeScript (undo/redo, filas, auditoria):**

```typescript
// Problema: operações que precisam ser desfeitas, enfileiradas ou auditadas

interface Comando {
  executar(): Promise<void>
  desfazer(): Promise<void>
  descricao: string
}

class TransferirLeadComando implements Comando {
  descricao = `Transferir lead ${this.leadId} de ${this.closerOrigemId} para ${this.closerDestinoId}`

  constructor(
    private readonly leadId: LeadId,
    private readonly closerOrigemId: CloserId,
    private readonly closerDestinoId: CloserId,
    private readonly repo: ILeadRepository,
  ) {}

  async executar(): Promise<void> {
    const lead = await this.repo.findById(this.leadId)
    lead.transferirPara(this.closerDestinoId)
    await this.repo.save(lead)
  }

  async desfazer(): Promise<void> {
    const lead = await this.repo.findById(this.leadId)
    lead.transferirPara(this.closerOrigemId)
    await this.repo.save(lead)
  }
}

class HistoricoComandos {
  private pilha: Comando[] = []

  async executar(comando: Comando): Promise<void> {
    await comando.executar()
    this.pilha.push(comando)
  }

  async desfazerUltimo(): Promise<void> {
    const ultimo = this.pilha.pop()
    if (!ultimo) throw new NadaParaDesfazerError()
    await ultimo.desfazer()
  }
}
```

---

## Fase 3 — Checklist antes de entregar

- [ ] O problema de variação está claramente identificado?
- [ ] O padrão escolhido isola exatamente esse ponto de variação?
- [ ] A interface (o que não muda) está separada da implementação (o que muda)?
- [ ] Composição foi preferida onde herança seria usada?
- [ ] O padrão foi escolhido por necessidade real, não por familiaridade?
- [ ] As consequências (trade-offs) do padrão foram consideradas?
- [ ] O nome do padrão está visível no código (ex: `PoliticaComissao`, `EmissaoFacade`)?

---

## Entrega de Giovana

1. **Identificação do ponto de variação** — o que muda e o que não muda
2. **Padrão recomendado** com justificativa e trade-offs
3. **Código TypeScript** completo com o padrão aplicado
4. **Padrões alternativos** considerados e por que foram descartados
5. **Pergunta de confirmação**: *"Quer que eu mostre como este padrão se integra com o restante do sistema?"*
