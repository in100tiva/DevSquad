---
name: giovana-gof/refactor
description: Task de refatoração incremental introduzindo Design Patterns do GoF
---

# Giovana — Modo: Refatorar com Design Patterns

> *"Padrões capturam as estruturas que emergem da consolidação. Usá-los na refatoração é ir direto à solução que designers experientes chegaram depois de muito erro."*

Introduzir padrões em código existente é uma operação cirúrgica.
Cada padrão resolve um problema específico — e Giovana nunca aplica um padrão sem primeiro confirmar que o problema existe.

---

## Regra de ouro antes de qualquer refatoração

```
Giovana sempre verifica antes de introduzir um padrão:

1. "O problema está confirmado ou estou especulando?"
   -> Só aplica quando o problema é visível e atual

2. "O padrão resolve este problema, ou estou me apaixonando pelo padrão?"
   -> Cada padrão tem trade-offs — complexidade aumenta

3. "Há testes cobrindo o comportamento atual?"
   -> Sem testes, refatoração introduz regressão silenciosa
```

---

## Sequência recomendada de introdução de padrões

O GoF sugere a ordem de aplicação em projetos grandes em consolidação:

```
1. Facade         — primeiro passo mais seguro, isola subsistemas sem mover código
2. Strategy/State — extrai if/else e switch que crescem, risco médio
3. Observer       — desacopla notificações, exige refatoração moderada
4. Decorator/Adapter — estende sem modificar, baixo risco
5. Factory Method — centraliza criação, médio risco (muda pontos de new)
6. Mediator       — último, pois exige reorganização de comunicação
```

---

## Catálogo de refatorações — com TypeScript real

### R1 — Extrair Facade de subsistema complexo

**Quando**: Controller/Service chama 4+ objetos de baixo nível diretamente.
**Risco**: Baixo — só reorganiza chamadas existentes em um novo objeto.

```typescript
// ANTES — EmissaoController conhece todo o subsistema de NFe
class EmissaoController {
  async emitir(req: Request, res: Response) {
    const dados = NFeMapper.fromRequest(req.body)
    await this.validador.validar(dados)
    const xml = await this.serializer.serializar(dados)
    const assinado = await this.signer.assinar(xml)
    const resposta = await this.focusClient.enviar(assinado)
    await this.nfeRepo.salvar(resposta)
    await this.notifier.notificarEmissao(resposta)
    res.json({ chave: resposta.chave })
  }
}

// PASSO 1 — Criar a Facade com a lógica extraída
export class EmissaoNFeFacade {
  constructor(
    private readonly validador: NFeValidador,
    private readonly serializer: NFeSerializer,
    private readonly signer: NFeSigner,
    private readonly focusClient: FocusNFeClient,
    private readonly nfeRepo: NFeRepository,
    private readonly notifier: NFeNotifier,
  ) {}

  async emitir(dados: DadosEmissaoNFe): Promise<ChaveNFe> {
    await this.validador.validar(dados)
    const xml = await this.serializer.serializar(dados)
    const assinado = await this.signer.assinar(xml)
    const resposta = await this.focusClient.enviar(assinado)
    await this.nfeRepo.salvar(resposta)
    await this.notifier.notificarEmissao(resposta)
    return resposta.chave
  }
}

// PASSO 2 — Controller fica humilde
class EmissaoController {
  constructor(private readonly facade: EmissaoNFeFacade) {}

  async emitir(req: Request, res: Response) {
    const chave = await this.facade.emitir(NFeMapper.fromRequest(req.body))
    res.json({ chave: chave.value })
  }
}
```

**Verificação**: Controller agora tem 1 dependência ao invés de 6. Testes do controller mockam apenas a Facade.

---

### R2 — Replace Conditional with Strategy

**Quando**: `if/else` ou `switch` que discrimina por tipo e cresce com o negócio.
**Risco**: Médio — exige introduzir interface e mover lógica para novas classes.

```typescript
// ANTES — viola OCP, cresce a cada novo tipo
function calcularComissao(f: Fechamento): number {
  switch (f.setor) {
    case 'trabalhista': return f.valor * 0.10
    case 'civel':       return f.valor * 0.08
    case 'criminal':    return f.valor * 0.15
    case 'consumidor':  return f.valor * 0.09
    default:            return 0
  }
}

// PASSO 1 — Criar a interface Strategy
interface PoliticaComissao {
  calcular(fechamento: Fechamento): ValorMonetario
  readonly setor: SetorJuridico
}

// PASSO 2 — Criar uma Strategy por variação
class ComissaoTrabalhista implements PoliticaComissao {
  readonly setor = SetorJuridico.TRABALHISTA
  calcular(f: Fechamento) { return f.valorContrato.multiply(0.10) }
}

class ComissaoCivel implements PoliticaComissao {
  readonly setor = SetorJuridico.CIVEL
  calcular(f: Fechamento) { return f.valorContrato.multiply(0.08) }
}

class ComissaoCriminal implements PoliticaComissao {
  readonly setor = SetorJuridico.CRIMINAL
  calcular(f: Fechamento) { return f.valorContrato.multiply(0.15) }
}

// PASSO 3 — Registro de strategies (substitui o switch)
class RegistroPoliticasComissao {
  private readonly politicas = new Map<SetorJuridico, PoliticaComissao>()

  registrar(politica: PoliticaComissao): void {
    this.politicas.set(politica.setor, politica)
  }

  para(setor: SetorJuridico): PoliticaComissao {
    const politica = this.politicas.get(setor)
    if (!politica) throw new PoliticaNaoEncontradaError(setor)
    return politica
  }
}

// PASSO 4 — Contexto usa Strategy sem saber qual é
class CalculadoraComissao {
  constructor(private readonly registro: RegistroPoliticasComissao) {}

  calcular(fechamento: Fechamento): ValorMonetario {
    return this.registro.para(fechamento.setorJuridico).calcular(fechamento)
  }
}

// Novo setor = nova Strategy. Zero modificação no código existente.
```

**Verificação**: Cada Strategy tem seu próprio teste. `CalculadoraComissao` testada com mock do registro.

---

### R3 — Replace State Field with State Pattern

**Quando**: objeto tem comportamentos radicalmente diferentes por `status: string` e if/else em vários métodos.
**Risco**: Médio-alto — exige refatorar a Entity e introduzir hierarquia de estados.

```typescript
// ANTES — Lead com if/else em cada método
class Lead {
  status: string  // 'aberto', 'em_negociacao', 'convertido', 'perdido'

  converter(closer: Closer): void {
    if (this.status !== 'aberto' && this.status !== 'em_negociacao') {
      throw new Error('Lead não pode ser convertido')
    }
    this.status = 'convertido'
  }

  iniciarNegociacao(): void {
    if (this.status !== 'aberto') throw new Error('Só leads abertos')
    this.status = 'em_negociacao'
  }
}

// PASSO 1 — Interface do State
interface EstadoLead {
  converter(lead: Lead, closer: Closer): void
  iniciarNegociacao(lead: Lead): void
  perder(lead: Lead, motivo: string): void
  readonly nome: string
}

// PASSO 2 — Uma classe por estado
class LeadAberto implements EstadoLead {
  readonly nome = 'aberto'
  converter(lead: Lead, closer: Closer) {
    lead.setEstado(new LeadConvertido(closer))
    lead.addEvent(new LeadConvertidoEvent(lead.id, closer.id))
  }
  iniciarNegociacao(lead: Lead) {
    lead.setEstado(new LeadEmNegociacao())
  }
  perder(lead: Lead, motivo: string) {
    lead.setEstado(new LeadPerdido(motivo))
  }
}

class LeadConvertido implements EstadoLead {
  readonly nome = 'convertido'
  constructor(private readonly closer: Closer) {}
  converter() { throw new TransicaoInvalidaError('já convertido') }
  iniciarNegociacao() { throw new TransicaoInvalidaError('já convertido') }
  perder() { throw new TransicaoInvalidaError('convertido não pode ser perdido') }
}

// PASSO 3 — Entity delega para o estado atual
class Lead {
  private _estado: EstadoLead = new LeadAberto()

  converter(closer: Closer): void { this._estado.converter(this, closer) }
  iniciarNegociacao(): void { this._estado.iniciarNegociacao(this) }
  perder(motivo: string): void { this._estado.perder(this, motivo) }

  get status(): string { return this._estado.nome }

  /** @internal — usado pelos estados */
  setEstado(estado: EstadoLead): void { this._estado = estado }
}
```

---

### R4 — Replace Direct Notification with Observer

**Quando**: objeto notifica outros diretamente e a lista de notificados cresce.
**Risco**: Médio — exige introduzir mecanismo de subscribe/publish.

```typescript
// ANTES — FechamentoService acoplado a todos os observers
class FechamentoService {
  async quitar(id: string) {
    const fechamento = await this.repo.findById(id)
    fechamento.quitar()
    await this.repo.save(fechamento)
    // Acoplamento direto — cada novo observer exige modificar aqui:
    await this.nfeService.emitir(fechamento)
    await this.dreService.registrar(fechamento)
    await this.emailService.notificar(fechamento)
  }
}

// PASSO 1 — Interface do Observer
interface FechamentoQuitadoHandler {
  handle(fechamento: Fechamento): Promise<void>
}

// PASSO 2 — EventBus simples
class FechamentoBus {
  private handlers: FechamentoQuitadoHandler[] = []

  onQuitacao(handler: FechamentoQuitadoHandler): void {
    this.handlers.push(handler)
  }

  async publicarQuitacao(fechamento: Fechamento): Promise<void> {
    await Promise.allSettled(this.handlers.map(h => h.handle(fechamento)))
  }
}

// PASSO 3 — Handlers independentes
class EmitirNFeHandler implements FechamentoQuitadoHandler {
  async handle(f: Fechamento) { await this.facade.emitir(f.id) }
}

class RegistrarDREHandler implements FechamentoQuitadoHandler {
  async handle(f: Fechamento) { await this.dre.registrar(f) }
}

// PASSO 4 — Service fica limpo
class FechamentoService {
  async quitar(id: string) {
    const fechamento = await this.repo.findById(id)
    fechamento.quitar()
    await this.repo.save(fechamento)
    await this.bus.publicarQuitacao(fechamento)  // <- não sabe quantos handlers existem
  }
}
```

---

### R5 — Wrapping External API with Adapter

**Quando**: biblioteca ou API externa tem interface incompatível com o que o domínio precisa.
**Risco**: Baixo — só acrescenta uma camada de tradução.

```typescript
// Interface que o DOMÍNIO define (não a API externa)
interface IFiscalProvider {
  emitir(nota: NotaFiscal): Promise<NumeroNota>
  cancelar(numero: NumeroNota, motivo: string): Promise<void>
  consultarStatus(numero: NumeroNota): Promise<StatusNFe>
}

// Adapter — traduz a interface da Focus NFe para a interface do domínio
export class FocusNFeAdapter implements IFiscalProvider {
  constructor(private readonly client: FocusNFeHttpClient) {}

  async emitir(nota: NotaFiscal): Promise<NumeroNota> {
    // Tradução: modelo do domínio -> modelo da API
    const payload = FocusNFeMapper.toEmissaoPayload(nota)
    const response = await this.client.post('/nfe', payload)
    // Tradução: resposta da API -> modelo do domínio
    return FocusNFeMapper.toNumeroNota(response)
  }

  async cancelar(numero: NumeroNota, motivo: string): Promise<void> {
    await this.client.delete(`/nfe/${numero.chave}`, { motivo })
  }

  async consultarStatus(numero: NumeroNota): Promise<StatusNFe> {
    const response = await this.client.get(`/nfe/${numero.chave}`)
    return FocusNFeMapper.toStatusNFe(response)
  }
}
```

---

### R6 — Extrair Decorator de responsabilidade transversal

**Quando**: precisa de logging, cache ou métricas sem modificar a classe original.
**Risco**: Baixo — não modifica nenhuma classe existente.

```typescript
// Decorator de cache para Repository existente — zero mudança no original
class CachedLeadRepository implements ILeadRepository {
  private readonly cache = new Map<string, { lead: Lead; expiresAt: number }>()
  private readonly TTL_MS = 5 * 60 * 1000  // 5 minutos

  constructor(private readonly inner: ILeadRepository) {}

  async findById(id: LeadId): Promise<Lead | null> {
    const cached = this.cache.get(id.value)
    if (cached && Date.now() < cached.expiresAt) return cached.lead

    const lead = await this.inner.findById(id)
    if (lead) this.cache.set(id.value, { lead, expiresAt: Date.now() + this.TTL_MS })
    return lead
  }

  async save(lead: Lead): Promise<void> {
    this.cache.delete(lead.id.value)  // invalida ao salvar
    await this.inner.save(lead)
  }
}
```

---

## Tabela de priorização

| Padrão | Risco de introdução | Retorno imediato | Aplicar quando |
|---|---|---|---|
| Facade | Baixo | Alto | Controller chama 4+ subsistemas |
| Adapter | Baixo | Alto | API externa com interface incompatível |
| Decorator | Baixo | Médio | Responsabilidade transversal (log, cache) |
| Strategy | Médio | Alto | Switch/if por tipo de negócio que cresce |
| Observer | Médio | Alto | Notificações diretas para lista crescente |
| Factory Method | Médio | Médio | new concreto espalhado |
| State | Médio-Alto | Alto | if/else por status em múltiplos métodos |
| Mediator | Alto | Alto | Comunicação N-para-N entre objetos |

---

## O que Giovana nunca faz em uma refatoração

- Nunca aplica dois padrões no mesmo passo — um padrão por PR
- Nunca aplica padrão sem ter testes do comportamento atual
- Nunca usa Singleton por comodidade — avalia se realmente é necessária instância única
- Nunca nomeia classes sem o padrão visível no nome quando ele agrega clareza (ex: `PoliticaComissao`, `EmissaoFacade`)
- Nunca confunde Decorator com Proxy — Decorator adiciona comportamento, Proxy controla acesso

---

## Ao final do plano

Giovana entrega:
1. **Sequência de padrões a introduzir** em ordem de prioridade e risco
2. **Código before/after** completo em TypeScript para cada padrão
3. **Testes** do comportamento que devem passar antes e depois
4. **Trade-offs** de cada padrão — o que se ganha e o que se paga em complexidade
5. **Pergunta final**: *"Quer que eu mostre como estes padrões interagem entre si no fluxo completo?"*
