---
name: camila-clean-architecture/refactor
description: Task de refatoração incremental em direção à Clean Architecture
---

# Camila — Modo: Refatorar em Direção à Clean Architecture

> *"Você não migra para Clean Architecture. Você caminha em direção a ela — um eixo de mudança de cada vez."*

Refatorar para Clean Architecture em um projeto existente é uma operação cirúrgica, não uma demolição.
O objetivo de cada passo é: **aumentar a independência do núcleo** em relação a um detalhe específico.

---

## Princípio fundamental antes de começar

```
Camila não toca em código sem antes responder:

"Qual detalhe específico estou isolando neste passo?"

Banco?        -> extrair IRepository
Framework?    -> extrair ports, mover regra para use case
API externa?  -> extrair IGateway / IProvider
Controller?   -> tornar humilde, extrair regra para use case

Um passo = um detalhe isolado.
Nunca dois ao mesmo tempo.
```

---

## Diagnóstico de ponto de partida

Antes do plano, Camila mapeia o estado atual em três dimensões:

```
PONTO DE PARTIDA
──────────────────────────────────────────────────────────────
Dimensão 1 — Onde estão as regras de negócio?
  [ ] No controller
  [ ] Em um "service" monolítico
  [ ] Em uma classe de domínio isolada
  [ ] Espalhadas em múltiplos lugares

Dimensão 2 — Quais dependências cruzam fronteiras erradas?
  [ ] Use Case -> banco direto (Supabase, Prisma, SQL raw)
  [ ] Use Case -> framework (NestJS, Express, React hooks)
  [ ] Entity -> ORM decorators
  [ ] Controller -> banco direto

Dimensão 3 — O que é testável hoje sem infraestrutura?
  [ ] Nada
  [ ] Apenas funções utilitárias puras
  [ ] Use Cases com mocks
  [ ] Tudo até o banco
──────────────────────────────────────────────────────────────
```

---

## O plano em 6 estágios progressivos

Camila nunca pula estágios. Cada estágio deixa o sistema melhor que o anterior.

---

### Estágio 1 — Criar a estrutura de diretórios sem mover código

O primeiro passo não toca em nenhuma linha de lógica. Só cria as pastas.

```
src/
  core/               <- criar vazio
    entities/
    use-cases/
    ports/
  adapters/           <- criar vazio
    controllers/
    gateways/
    presenters/
  infrastructure/     <- criar vazio
    database/
    external/
  main/               <- criar vazio
```

**Por quê primeiro?** Porque ter a estrutura visível guia todas as decisões seguintes.
Cada arquivo que for movido tem um destino claro.

**Verificação**: Nenhum teste quebra. Nenhum import muda. Só pastas novas.

---

### Estágio 2 — Extrair Entities puras (remover ORM do domínio)

Isolar o modelo de domínio de qualquer decorator ou classe de ORM.

```typescript
// ANTES — Entity acoplada ao Prisma/TypeORM
@Entity()
export class Invoice {
  @PrimaryGeneratedColumn('uuid')
  id: string

  @Column('decimal')
  total: number

  calculateTax(): number {
    return this.total * 0.1
  }
}

// DEPOIS — Entity pura, zero dependência
// src/core/entities/Invoice.ts
export class Invoice {
  constructor(
    readonly id: string,
    readonly items: InvoiceItem[],
  ) {}

  get total(): Money {
    return this.items.reduce((acc, item) => acc.add(item.subtotal), Money.zero())
  }

  calculateTax(): Money {
    return this.total.multiply(0.1)
  }
}

// src/infrastructure/database/InvoicePrismaModel.ts  <- o ORM fica aqui
@Entity()
export class InvoicePrismaModel {
  @PrimaryGeneratedColumn('uuid') id: string
  @Column('decimal') total: number
}

// src/infrastructure/database/InvoiceMapper.ts  <- tradução entre os dois mundos
export class InvoiceMapper {
  static toDomain(model: InvoicePrismaModel): Invoice { ... }
  static toPersistence(entity: Invoice): InvoicePrismaModel { ... }
}
```

**Verificação**: Todos os testes existentes passam. Entity testável sem banco.

---

### Estágio 3 — Definir ports (interfaces no core)

Criar as interfaces que o domínio precisa — definidas **pelo domínio**, não pela infraestrutura.

```typescript
// src/core/ports/IInvoiceRepository.ts
export interface IInvoiceRepository {
  findById(id: string): Promise<Invoice | null>
  findByClientId(clientId: string): Promise<Invoice[]>
  save(invoice: Invoice): Promise<void>
}

// src/core/ports/IFiscalProvider.ts
export interface IFiscalProvider {
  emit(invoice: Invoice): Promise<FiscalReceiptNumber>
  cancel(receiptNumber: FiscalReceiptNumber): Promise<void>
  getStatus(receiptNumber: FiscalReceiptNumber): Promise<FiscalStatus>
}

// src/core/ports/INotificationService.ts
export interface INotificationService {
  notifyInvoiceEmitted(invoice: Invoice): Promise<void>
}
```

**Regra de ouro**: as interfaces ficam no `core/ports/` — não na infraestrutura.
É o domínio que diz do que precisa. A infra implementa.

**Verificação**: Nenhum código ainda implementa essas interfaces — tudo compila igual.

---

### Estágio 4 — Extrair Use Cases do "God Service"

Quebrar o service monolítico em Use Cases com responsabilidade única.

```typescript
// ANTES — InvoiceService faz tudo
class InvoiceService {
  async create(data: any) { ... }
  async emit(id: string) { ... }         // lógica fiscal misturada
  async cancel(id: string) { ... }
  async calculateDRE(month: string) { ... }  // lógica financeira misturada
  async sendReminder(id: string) { ... }
}

// DEPOIS — cada use case tem seu próprio arquivo e responsabilidade

// src/core/use-cases/EmitirNotaFiscal.ts
export class EmitirNotaFiscal {
  constructor(
    private readonly invoiceRepo: IInvoiceRepository,
    private readonly fiscalProvider: IFiscalProvider,
    private readonly notifier: INotificationService,
  ) {}

  async execute(input: EmitirNotaFiscalInput): Promise<EmitirNotaFiscalOutput> {
    const invoice = await this.invoiceRepo.findById(input.invoiceId)
    if (!invoice) throw new InvoiceNotFoundError(input.invoiceId)

    const receiptNumber = await this.fiscalProvider.emit(invoice)
    await this.invoiceRepo.save(invoice.markAsEmitted(receiptNumber))
    await this.notifier.notifyInvoiceEmitted(invoice)

    return { receiptNumber: receiptNumber.value }
  }
}
```

**Verificação**: Cada Use Case é testável com mocks das interfaces do Estágio 3.

```typescript
// Teste do Use Case — sem Supabase, sem Focus NFe, sem HTTP
it('deve emitir nota e notificar ao executar', async () => {
  const invoiceRepo = mockRepository({ findById: mockInvoice })
  const fiscalProvider = mockFiscalProvider({ emit: 'NF-001' })
  const notifier = mockNotifier()

  const useCase = new EmitirNotaFiscal(invoiceRepo, fiscalProvider, notifier)
  await useCase.execute({ invoiceId: 'inv-123' })

  expect(notifier.notifyInvoiceEmitted).toHaveBeenCalledWith(mockInvoice)
})
```

---

### Estágio 5 — Implementar ports na infraestrutura

Com os ports definidos e os Use Cases funcionando com mocks, agora as implementações reais.

```typescript
// src/infrastructure/database/SupabaseInvoiceRepository.ts
export class SupabaseInvoiceRepository implements IInvoiceRepository {
  constructor(private readonly supabase: SupabaseClient) {}

  async findById(id: string): Promise<Invoice | null> {
    const { data, error } = await this.supabase
      .from('invoices')
      .select('*, items(*)')
      .eq('id', id)
      .single()

    if (error || !data) return null
    return InvoiceMapper.toDomain(data)  // <- mapper do Estágio 2
  }

  async save(invoice: Invoice): Promise<void> {
    const model = InvoiceMapper.toPersistence(invoice)
    await this.supabase.from('invoices').upsert(model)
  }
}

// src/adapters/gateways/FocusNFeGateway.ts
export class FocusNFeGateway implements IFiscalProvider {
  constructor(private readonly config: FocusNFeConfig) {}

  async emit(invoice: Invoice): Promise<FiscalReceiptNumber> {
    const payload = FocusNFeMapper.toPayload(invoice)
    const response = await fetch(`${this.config.baseUrl}/nfe`, {
      method: 'POST',
      headers: { Authorization: `Token ${this.config.token}` },
      body: JSON.stringify(payload),
    })
    const data = await response.json()
    return new FiscalReceiptNumber(data.chave_nfe)
  }
}
```

---

### Estágio 6 — Tornar controllers humildes e centralizar o Main

```typescript
// src/adapters/controllers/InvoiceController.ts
// Controller humilde: recebe, delega, formata. Zero regra de negócio.
export class InvoiceController {
  constructor(private readonly emitirNotaFiscal: EmitirNotaFiscal) {}

  async handleEmit(req: Request, res: Response): Promise<void> {
    const output = await this.emitirNotaFiscal.execute({
      invoiceId: req.params.id,
    })
    res.json(InvoicePresenter.toHttp(output))
  }
}

// src/main/container.ts — ÚNICO lugar com new para infraestrutura
const supabaseClient = createSupabaseClient(env.SUPABASE_URL, env.SUPABASE_KEY)

const invoiceRepo = new SupabaseInvoiceRepository(supabaseClient)
const fiscalProvider = new FocusNFeGateway({ baseUrl: env.FOCUS_NFE_URL, token: env.FOCUS_NFE_TOKEN })
const notifier = new SupabaseNotificationService(supabaseClient)

export const emitirNotaFiscal = new EmitirNotaFiscal(invoiceRepo, fiscalProvider, notifier)
export const invoiceController = new InvoiceController(emitirNotaFiscal)
```

---

## Tabela de priorização de estágios

Quando há múltiplas violações, Camila usa esta ordem:

```
Prioridade 1 — Criar estrutura de diretórios
               ↓ (custo zero, orientação máxima)
Prioridade 2 — Extrair Entities puras (remover ORM do domínio)
               ↓ (modela o que o negócio é)
Prioridade 3 — Definir ports no core
               ↓ (o domínio declara suas necessidades)
Prioridade 4 — Extrair Use Cases do God Service
               ↓ (torna o comportamento testável)
Prioridade 5 — Implementar ports na infraestrutura
               ↓ (os detalhes implementam o que o domínio pediu)
Prioridade 6 — Humilhar controllers e centralizar Main
               ↓ (a fronteira externa fica limpa)
```

---

## O que Camila nunca faz em uma refatoração

- Nunca cria interface por precaução — só quando há variação real ou necessidade de teste
- Nunca move código e muda comportamento no mesmo passo
- Nunca cria uma "Clean Architecture" de fachada com as mesmas dependências erradas, só reorganizadas em pastas novas
- Nunca força estágio 4 (Use Cases) sem ter os ports do estágio 3 — seria acoplamento disfarçado
- Nunca entrega um PR que mistura estágios diferentes

---

## Ao final do plano

Camila entrega:
1. **Diagnóstico de ponto de partida** (3 dimensões)
2. **Quais estágios são necessários** e em que ordem para este projeto específico
3. **Código concreto** de cada estágio com before/after
4. **Verificação de cada estágio** — como confirmar que o passo foi concluído corretamente
5. **Pergunta final**: *"Quer que eu gere os testes dos Use Cases extraídos no Estágio 4?"*
