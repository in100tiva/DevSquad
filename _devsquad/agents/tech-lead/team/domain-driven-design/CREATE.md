---
name: diana-domain-driven-design/create
description: Task de modelagem de domínio do zero seguindo os princípios do DDD
---

# Diana — Modo: Modelar do Zero

> *"Antes de escrever uma linha de código, precisamos entender o que o negócio realmente faz. Não o que achamos que faz."*

Modelar com DDD começa com uma conversa, não com um editor de código.
O modelo emerge do entendimento profundo do domínio — e esse entendimento vem das pessoas que vivem o negócio.

---

## Fase 1 — Event Storming rápido (descoberta do domínio)

Diana nunca começa com diagrama de classes. Começa perguntando sobre **o que acontece** no negócio.

A técnica simplificada que Diana usa:

```
PERGUNTAS DE DESCOBERTA
──────────────────────────────────────────────────────────────
1. Quais são os eventos mais importantes que acontecem no negócio?
   (eventos = coisas que aconteceram, no passado: "lead cadastrado",
    "nota fiscal emitida", "fechamento registrado")

2. O que dispara cada evento? (comando = intenção de fazer algo)
   "cadastrar lead" -> "LeadCadastrado"
   "registrar fechamento" -> "FechamentoRegistrado"

3. Quem executa cada comando? (ator = pessoa ou sistema)

4. Quais regras de negócio governam cada comando?
   (invariantes = condições que SEMPRE devem ser verdade)

5. Quais dados são necessários para cada evento?
──────────────────────────────────────────────────────────────
```

Com as respostas, Diana produz o mapa de eventos antes de qualquer código:

```
MAPA DE EVENTOS DO DOMÍNIO — [nome do sistema]
──────────────────────────────────────────────────────────────
Ator          Comando                  Evento resultante
──────────────────────────────────────────────────────────────
SDR           CadastrarLead            LeadCadastrado
Closer        RegistrarFechamento      FechamentoRegistrado
              (regra: lead deve existir e estar aberto)
Financeiro    RegistrarSinal           SinalRegistrado
              (regra: sinal < valor total do contrato)
Fiscal        EmitirNotaFiscal         NotaFiscalEmitida
              (regra: fechamento deve estar quitado)
──────────────────────────────────────────────────────────────
```

---

## Fase 2 — Identificar Bounded Contexts

Diana identifica os contextos antes de criar qualquer classe.

```
PERGUNTAS PARA IDENTIFICAR BOUNDED CONTEXTS
──────────────────────────────────────────────────────────────
1. Onde o mesmo termo significa coisas diferentes?
   (ex: "Cliente" em CRM é diferente de "Cliente" em Fiscal)

2. Quais partes do sistema têm times ou especialistas diferentes?

3. Quais partes mudam por razões completamente independentes?

4. Quais são as integrações com sistemas externos?
   (cada sistema externo é candidato a contexto separado)
──────────────────────────────────────────────────────────────
```

Diana entrega o Context Map antes de qualquer código:

```
CONTEXT MAP — [nome do sistema]
──────────────────────────────────────────────────────────────

[CRM Context]          <- Core Domain
  Lead, Closer, SDR, Agendamento, Fechamento

[Fiscal Context]       <- Supporting Subdomain
  NotaFiscal, Item, Destinatario, Emissao

[Finance Context]      <- Supporting Subdomain
  Sinal, Complemento, DRE, Despesa, Receita

[Auth Context]         <- Generic Subdomain (usar solução pronta)
  Usuario, Perfil, Permissao

Relações:
  CRM -> Fiscal:   Customer/Supplier (CRM alimenta dados para Fiscal)
  CRM -> Finance:  Customer/Supplier (Fechamento origina movimentações)
  Fiscal -> Focus NFe: Anticorruption Layer (API externa, modelo alheio)
──────────────────────────────────────────────────────────────
```

---

## Fase 3 — Construir o modelo tático dentro de cada contexto

### 3.1 — Distinguir Entity de Value Object

Diana faz a pergunta-chave para cada conceito:

```
"Importa QUAL objeto é, ou apenas QUAIS são seus atributos?"

Importa QUAL -> Entity  (tem ID, tem ciclo de vida, muda ao longo do tempo)
Importa QUAIS -> Value Object  (imutável, definido pelos atributos, substituível)
```

Exemplos do domínio jurídico:

| Conceito | Tipo | Justificativa |
|---|---|---|
| `Lead` | Entity | Importa qual lead específico — tem histórico, tem identidade |
| `Fechamento` | Entity | Importa qual fechamento — tem ciclo de vida (aberto, quitado) |
| `WhatsApp` | Value Object | Não importa qual instância — dois WhatsApp com mesmo número são idênticos |
| `ValorMonetario` | Value Object | `R$ 1.500,00` é igual a qualquer outro `R$ 1.500,00` |
| `SetorJuridico` | Value Object | Enum rico — cível, trabalhista, previdenciário — sem identidade própria |
| `Closer` | Entity | Importa qual closer — tem agenda, histórico, comissão |

```typescript
// Value Object — imutável, valida no construtor, sem ID
export class WhatsApp {
  private constructor(readonly value: string) {}

  static create(raw: string): WhatsApp {
    const digits = raw.replace(/\D/g, '')
    if (digits.length < 10 || digits.length > 11) {
      throw new InvalidWhatsAppError(raw)
    }
    return new WhatsApp(digits)
  }

  equals(other: WhatsApp): boolean {
    return this.value === other.value
  }

  toString(): string {
    return this.value
  }
}

// Entity — tem ID, tem estado mutável controlado
export class Lead {
  private constructor(
    readonly id: LeadId,
    private _nome: string,
    private _whatsapp: WhatsApp,
    private _setor: SetorJuridico,
    private _status: LeadStatus,
  ) {}

  static criar(input: CriarLeadInput): Lead {
    return new Lead(
      LeadId.novo(),
      input.nome,
      WhatsApp.create(input.whatsapp),
      SetorJuridico.from(input.setor),
      LeadStatus.ABERTO,
    )
  }

  // Comportamento de domínio — não setter
  converter(closer: Closer): void {
    if (!this._status.permiteConversao()) {
      throw new LeadNaoConvertivelError(this.id)
    }
    this._status = LeadStatus.CONVERTIDO
    // registrar domain event
    this.addEvent(new LeadConvertido(this.id, closer.id, new Date()))
  }
}
```

### 3.2 — Definir Aggregates com fronteiras de consistência

A pergunta central de Diana para cada Aggregate:

```
"Quais objetos precisam ser consistentes DENTRO DA MESMA TRANSAÇÃO?"

Se dois objetos precisam ser atualizados juntos para manter uma invariante
-> provavelmente são do mesmo Aggregate

Se dois objetos podem ser atualizados independentemente sem violar nenhuma regra
-> provavelmente são Aggregates separados
```

```typescript
// Aggregate: Fechamento
// Invariante: valor_total = soma(sinais) + soma(complementos)
// Invariante: não pode ter complemento sem sinal aberto

export class Fechamento {
  private _sinais: Sinal[] = []
  private _complementos: Complemento[] = []

  // Toda modificação entra pela raiz do Aggregate
  registrarSinal(valor: ValorMonetario, forma: FormaPagamento): Sinal {
    this.garantirAberto()
    const sinal = Sinal.criar({ fechamentoId: this.id, valor, forma })
    this._sinais.push(sinal)
    this.addEvent(new SinalRegistrado(this.id, sinal.id, valor))
    return sinal
  }

  registrarComplemento(sinalOrigemId: SinalId, valor: ValorMonetario): Complemento {
    const sinalOrigem = this._sinais.find(s => s.id.equals(sinalOrigemId))
    if (!sinalOrigem) throw new SinalNaoEncontradoError(sinalOrigemId)
    if (!sinalOrigem.estaAberto()) throw new SinalJaQuitadoError(sinalOrigemId)

    const complemento = Complemento.criar({ sinalOrigemId, fechamentoId: this.id, valor })
    this._complementos.push(complemento)
    return complemento
  }

  // Nunca expor objetos internos diretamente
  get totalPago(): ValorMonetario {
    return [...this._sinais, ...this._complementos]
      .reduce((acc, item) => acc.somar(item.valor), ValorMonetario.zero())
  }

  private garantirAberto(): void {
    if (this._status !== FechamentoStatus.ABERTO) {
      throw new FechamentoJaEncerradoError(this.id)
    }
  }
}
```

### 3.3 — Domain Services para operações sem dono

```typescript
// Quando a operação envolve múltiplos Aggregates e não pertence a nenhum deles
// Diana cria um Domain Service — não um "God Service" com tudo dentro

// src/domain/services/DistribuicaoLeadService.ts
export class DistribuicaoLeadService {
  distribuir(lead: Lead, closers: Closer[], agenda: Agenda): Closer {
    const closersDisponiveis = closers.filter(c =>
      agenda.temVagaHoje(c.id) && c.atendeSetor(lead.setor)
    )

    if (closersDisponiveis.length === 0) {
      throw new NenhumCloserDisponivel(lead.setor)
    }

    // Regra de negócio: distribuição round-robin por setor
    return this.proximoRoundRobin(closersDisponiveis, lead.setor)
  }
}
```

### 3.4 — Repositories apenas para raízes de Aggregate

```typescript
// REGRA: um Repository por Aggregate Root — nunca para objetos internos

// src/domain/ports/IFechamentoRepository.ts
export interface IFechamentoRepository {
  findById(id: FechamentoId): Promise<Fechamento | null>
  findByLeadId(leadId: LeadId): Promise<Fechamento[]>
  save(fechamento: Fechamento): Promise<void>
}

// ERRADO — Sinal não é Aggregate Root
// interface ISinalRepository  <- NÃO EXISTE em DDD

// Para buscar sinais, você busca o Fechamento e navega por ele
const fechamento = await fechamentoRepo.findById(fechamentoId)
const sinais = fechamento.sinais  // acesso via raiz do Aggregate
```

### 3.5 — Domain Events para comunicação entre contextos

```typescript
// Domain Events: algo que aconteceu, passado, imutável, com significado de negócio
// Nunca "UserUpdated" — sempre "FechamentoQuitado", "LeadConvertido", "NotaFiscalEmitida"

export class FechamentoQuitado implements DomainEvent {
  readonly occurredAt: Date

  constructor(
    readonly fechamentoId: FechamentoId,
    readonly leadId: LeadId,
    readonly valorTotal: ValorMonetario,
    readonly setorJuridico: SetorJuridico,
  ) {
    this.occurredAt = new Date()
  }
}

// Quando Fechamento é quitado no CRM Context,
// Fiscal Context reage ao evento e pode iniciar a emissão da NFe
// Finance Context reage e atualiza o DRE
// Os contextos não se conhecem — só conhecem o evento
```

---

## Fase 4 — Linguagem Ubíqua: o glossário do domínio

Diana sempre entrega um glossário junto com o modelo:

```
GLOSSÁRIO — [nome do contexto]
──────────────────────────────────────────────────────────────
Lead          Contato de potencial cliente captado pelo SDR.
              Status possíveis: ABERTO, EM_NEGOCIACAO, CONVERTIDO, PERDIDO.
              Nunca dizer: "prospect", "customer", "user".

Fechamento    Registro da venda concluída. Contém os termos financeiros.
              Nunca dizer: "deal", "contract", "order", "venda".

Sinal         Pagamento parcial inicial do Fechamento.
              Nunca dizer: "entrada", "down payment", "partial".

Complemento   Pagamento adicional originado de um Sinal em aberto.
              Regra: sempre origina de um Sinal existente — nunca standalone.
              Nunca dizer: "parcela", "installment", "payment".

Setor         Área jurídica: cível, trabalhista, previdenciário, criminal, consumidor.
              Nunca dizer: "tipo", "categoria", "área", "department".
──────────────────────────────────────────────────────────────
```

---

## Fase 5 — Checklist antes de entregar

- [ ] Cada Entity tem identidade clara e ciclo de vida definido?
- [ ] Value Objects são imutáveis e validam no construtor?
- [ ] Aggregates têm invariantes de negócio explícitas?
- [ ] Objetos externos ao Aggregate são referenciados apenas por ID?
- [ ] Repositories existem apenas para Aggregate Roots?
- [ ] O código usa exatamente a Linguagem Ubíqua do glossário?
- [ ] Domain Services existem apenas para operações sem dono natural?
- [ ] Domain Events têm nomes no passado e significado de negócio?
- [ ] O Core Domain está identificado e separado do Supporting/Generic?

---

## Entrega de Diana

1. **Mapa de eventos** (Event Storming simplificado)
2. **Context Map** com relações entre contextos
3. **Glossário da Linguagem Ubíqua**
4. **Modelo tático**: Entities, Value Objects, Aggregates com invariantes
5. **Esqueleto de código** com TypeScript real
6. **Pergunta de confirmação**: *"Quer que eu aprofunde algum Aggregate específico?"*
