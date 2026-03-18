---
name: diana-domain-driven-design/refactor
description: Task de refatoração incremental em direção a um modelo de domínio mais profundo
---

# Diana — Modo: Refatorar o Modelo

> *"Refatorar para um modelo mais profundo não é reorganizar código. É encontrar um entendimento que simplifica tudo."*

Evans distingue três níveis de refatoração:
- **Micro-refatoração** — melhorias técnicas (renomear, extrair função)
- **Refatoração para padrões** — aplicar Design Patterns
- **Refatoração para insight mais profundo** — descobrir um modelo que torna o design naturalmente mais simples

Diana foca no terceiro nível — o mais impactante e o mais negligenciado.

---

## Antes de começar: escolher o alvo certo

Evans é categórico sobre onde não começar: nem "refatore tudo" (impraticável), nem "refatore o que dói agora" (trata sintomas).

```
DIANA PERGUNTA PRIMEIRO:
──────────────────────────────────────────────────────────────
"O problema que estamos resolvendo envolve o Core Domain
 ou apenas um subdomínio de suporte?"

Se envolve o Core Domain -> resolva primeiro, mesmo que seja mais difícil
Se é Supporting/Generic  -> pode resolver em paralelo ou depois
──────────────────────────────────────────────────────────────

SEQUÊNCIA DE PRIORIDADE (Evans)
1. Se há um problema sendo resolvido agora:
   -> Cheque se a causa raiz está no Core Domain
   -> Se sim: comece por lá

2. Se há liberdade para refatorar proativamente:
   -> Melhore a fatoração do Core Domain
   -> Melhore a segregação do Core
   -> Purifique os Supporting Subdomains
```

---

## Fase 1 — Refatoração de Linguagem (custo baixo, retorno alto)

O primeiro passo de toda refatoração de modelo é alinhar vocabulário.
Nenhum código move de lugar — só nomes mudam.

```typescript
// ANTES — vocabulário desalinhado com o negócio jurídico
class Sale {                    // negócio diz "Fechamento"
  amount: number                // negócio diz "valorContrato"
  client: string                // negócio diz "Lead" (antes da conversão)
  type: string                  // negócio diz "setorJuridico"
  status: string                // negócio diz "statusFechamento"
  partialPayment: number        // negócio diz "sinal"
}

// DEPOIS — código que fala a língua do negócio
class Fechamento {
  valorContrato: ValorMonetario
  leadId: LeadId
  setorJuridico: SetorJuridico
  status: FechamentoStatus
  // sinais acessados via Aggregate
}
```

**Checklist de renomeação:**
- [ ] Classes → substantivos do glossário do negócio
- [ ] Métodos → verbos do vocabulário do negócio (`converter`, `quitar`, `emitir`)
- [ ] Parâmetros → nomes concretos do domínio (não `data`, `payload`, `info`)
- [ ] Banco de dados → mesmos nomes (tabela `fechamentos`, não `sales`)
- [ ] API → mesmos nomes (`/fechamentos`, não `/deals`)

**Verificação:** Um especialista de negócio consegue ler o diff e reconhecer os conceitos?

---

## Fase 2 — Eliminar Primitive Obsession com Value Objects

Cada primitivo que representa um conceito de negócio é um Value Object disfarçado.

```typescript
// R1 — Extrair Value Object de string
// ANTES
class Lead {
  whatsapp: string  // qualquer string, incluindo inválidas
  status: string    // qualquer string, nenhum controle
}

// DEPOIS
class Lead {
  whatsapp: WhatsApp        // só pode existir com formato válido
  status: LeadStatus        // só os valores possíveis existem
}

export class WhatsApp {
  private constructor(readonly value: string) {}

  static create(raw: string): WhatsApp {
    const digits = raw.replace(/\D/g, '')
    if (digits.length < 10 || digits.length > 11) {
      throw new InvalidWhatsAppError(raw)
    }
    return new WhatsApp(digits)
  }

  equals(other: WhatsApp): boolean { return this.value === other.value }
}

// R2 — Extrair Value Object de number monetário
// ANTES
totalContrato: number  // R$ 1500.00 ou 1500 ou 1500.001?

// DEPOIS
export class ValorMonetario {
  private constructor(readonly centavos: number) {}

  static deCentavos(centavos: number): ValorMonetario {
    if (centavos < 0) throw new ValorNegativoError()
    if (!Number.isInteger(centavos)) throw new ValorFracionadoError()
    return new ValorMonetario(centavos)
  }

  static deReais(reais: number): ValorMonetario {
    return ValorMonetario.deCentavos(Math.round(reais * 100))
  }

  somar(outro: ValorMonetario): ValorMonetario {
    return new ValorMonetario(this.centavos + outro.centavos)
  }

  get reais(): number { return this.centavos / 100 }

  static zero(): ValorMonetario { return new ValorMonetario(0) }
}
```

**Candidatos mais comuns a Value Object no domínio jurídico:**

| Primitivo | Value Object | Validação embutida |
|---|---|---|
| `string` whatsapp | `WhatsApp` | formato, tamanho |
| `number` valor | `ValorMonetario` | não negativo, centavos inteiros |
| `string` status | `LeadStatus`, `FechamentoStatus` | enum tipado |
| `string` setor | `SetorJuridico` | valores válidos do negócio |
| `string` id | `LeadId`, `FechamentoId`, `CloserId` | UUID válido, tipo seguro |

---

## Fase 3 — Recuperar Invariantes para dentro do Aggregate

Invariantes espalhadas em Services são a causa raiz de bugs de consistência.

```typescript
// R3 — Mover invariante do Service para o Aggregate

// ANTES — invariante no Service (frágil, duplicada)
class FechamentoService {
  async registrarComplemento(fechamentoId: string, sinalId: string, valor: number) {
    const fechamento = await this.repo.findById(fechamentoId)
    const sinal = await this.sinalRepo.findById(sinalId)  // Repository para não-root!

    // Invariante espalhada:
    if (sinal.status !== 'aberto') throw new Error('Sinal já quitado')
    if (sinal.fechamentoId !== fechamentoId) throw new Error('Sinal não pertence ao fechamento')
    if (valor > sinal.valorRestante) throw new Error('Valor excede o restante')

    const complemento = new Complemento({ sinalId, valor })
    await this.complementoRepo.save(complemento)  // Repository para não-root!
  }
}

// DEPOIS — invariante encapsulada no Aggregate
class Fechamento {
  registrarComplemento(sinalOrigemId: SinalId, valor: ValorMonetario): void {
    const sinal = this._sinais.find(s => s.id.equals(sinalOrigemId))
    if (!sinal) throw new SinalNaoEncontradoError(sinalOrigemId)

    // Invariante vive onde deve viver — dentro do Aggregate
    sinal.garantirAberto()
    sinal.garantirValorDisponivel(valor)

    this._complementos.push(
      Complemento.criar({ sinalOrigemId, fechamentoId: this.id, valor })
    )
  }
}

// Service fica fino — só orquestra
class RegistrarComplementoUseCase {
  async execute(input: RegistrarComplementoInput): Promise<void> {
    const fechamento = await this.fechamentoRepo.findById(
      FechamentoId.from(input.fechamentoId)
    )
    if (!fechamento) throw new FechamentoNaoEncontradoError(input.fechamentoId)

    fechamento.registrarComplemento(
      SinalId.from(input.sinalId),
      ValorMonetario.deReais(input.valor)
    )

    await this.fechamentoRepo.save(fechamento)
  }
}
```

---

## Fase 4 — Introduzir Domain Events para comunicação entre contextos

```typescript
// R4 — Substituir chamada direta entre módulos por Domain Event

// ANTES — acoplamento direto entre CRM e Fiscal
class FechamentoService {
  async quitar(id: string) {
    // ...
    await this.nfeService.iniciarEmissao(fechamento)  // CRM chama Fiscal diretamente!
    await this.dre.registrarReceita(fechamento)       // CRM chama Finance diretamente!
  }
}

// DEPOIS — Aggregate publica evento, contextos reagem de forma desacoplada
class Fechamento {
  quitar(): void {
    this.garantirNaoQuitado()
    this._status = FechamentoStatus.QUITADO

    // Publica o fato — não chama ninguém diretamente
    this.addEvent(new FechamentoQuitado(
      this.id,
      this.leadId,
      this.valorContrato,
      this.setorJuridico,
      new Date(),
    ))
  }
}

// Fiscal Context reage ao evento
class QuandoFechamentoQuitado {
  async handle(event: FechamentoQuitado): Promise<void> {
    const dados = FechamentoDomainMapper.toEmissaoInput(event)
    await this.emitirNotaFiscal.execute(dados)
  }
}

// Finance Context reage ao mesmo evento — independentemente
class RegistrarReceitaAoQuitarFechamento {
  async handle(event: FechamentoQuitado): Promise<void> {
    await this.dreService.registrarReceita({
      valor: event.valorTotal,
      setor: event.setorJuridico,
      data: event.occurredAt,
    })
  }
}
```

---

## Fase 5 — Anticorruption Layer para sistemas externos

```typescript
// R5 — Proteger o modelo de domínio do modelo externo (Focus NFe, Supabase, etc.)

// ANTES — modelo da Focus NFe vazando para o domínio
class EmissaoService {
  async emitir(fechamento: Fechamento) {
    // Objeto da API externa chegando diretamente no domínio
    const payload = {
      natureza_operacao: 'PRESTACAO DE SERVICOS',
      data_emissao: new Date().toISOString(),
      destinatario: {
        cpf_cnpj: fechamento.lead.cpf,  // acoplado ao modelo externo
        // ...
      }
    }
  }
}

// DEPOIS — Anticorruption Layer isola o domínio da API externa
// src/adapters/gateways/FocusNFeGateway.ts  <- vive fora do domínio

export class FocusNFeGateway implements IFiscalProvider {
  async emitir(notaFiscal: NotaFiscal): Promise<NumeroNota> {
    // Translator: converte do modelo de domínio para o modelo da API
    const payload = FocusNFeTranslator.toPayload(notaFiscal)

    const response = await this.httpClient.post('/nfe', payload)

    // Translator: converte da resposta da API para o modelo de domínio
    return FocusNFeTranslator.toNumeroNota(response)
  }
}

// Translator — encapsula todo o conhecimento do modelo externo
class FocusNFeTranslator {
  static toPayload(nota: NotaFiscal): FocusNFePayload {
    return {
      natureza_operacao: nota.naturezaOperacao.value,
      data_emissao: nota.dataEmissao.toISOString(),
      destinatario: {
        cpf_cnpj: nota.destinatario.documento.value,
        nome: nota.destinatario.nome,
        // ...
      }
    }
  }

  static toNumeroNota(response: FocusNFeResponse): NumeroNota {
    return NumeroNota.from(response.chave_nfe)
  }
}
```

---

## Fase 6 — Refatoração para insight mais profundo

Evans chama isso de "Breakthrough" — o momento em que um novo entendimento do domínio simplifica o modelo.

Diana facilita esse processo com **Exploration Teams**:

```
EXPLORATION TEAM — sessão de 30-90 min

Participantes: 2-3 devs + 1 especialista de domínio (ideal)

Agenda:
  1. Problema: "Esta regra está espalhada em 5 lugares — por quê?"
  2. Brainstorm: "Como o negócio realmente funciona aqui?"
  3. Esboço: modelos alternativos no quadro/papel
  4. Teste: "Este modelo torna a regra óbvia ou ainda confusa?"
  5. Código: implementar o modelo candidato

Sinais de Breakthrough:
  - A regra que era complexa fica trivial
  - O código fica mais curto, não maior
  - Todos no time dizem "ah, faz sentido agora"

Sinal de que ainda não chegou:
  - O modelo novo é tão complexo quanto o anterior
  - Só devs entendem, especialistas de negócio não reconhecem
```

---

## Sequência de priorização de Diana

```
Prioridade 1 — Identificar o Core Domain
               ↓ (saber onde o esforço tem maior retorno)
Prioridade 2 — Refatoração de linguagem (renomear)
               ↓ (custo zero, clareza imediata, base para tudo)
Prioridade 3 — Eliminar Primitive Obsession com Value Objects
               ↓ (validação centralizada, bugs impossíveis)
Prioridade 4 — Recuperar invariantes para os Aggregates
               ↓ (consistência garantida pelo modelo, não por disciplina)
Prioridade 5 — Introduzir Domain Events
               ↓ (desacopla contextos sem perder comunicação)
Prioridade 6 — Implementar Anticorruption Layers
               ↓ (protege o modelo de sistemas externos)
Prioridade 7 — Buscar insights mais profundos (Exploration Teams)
               ↓ (breakthroughs que simplificam o design inteiro)
```

---

## O que Diana nunca faz em uma refatoração

- Nunca renomeia e move invariante no mesmo passo — um passo, uma mudança
- Nunca cria Value Object sem validação real — VO sem validação é só uma classe wrapper inútil
- Nunca move regra para o Aggregate sem antes ter teste cobrindo a invariante
- Nunca cria Domain Event sem nome no passado e sem significado de negócio real
- Nunca aceita "modelo parecido com o anterior mas com nomes diferentes" como refatoração

---

## Ao final do plano

Diana entrega:
1. **Identificação do Core Domain** e onde está o maior retorno
2. **Plano em fases** com antes/depois em TypeScript real
3. **Glossário atualizado** com os novos termos introduzidos
4. **Testes das invariantes** para cada Aggregate refatorado
5. **Pergunta final**: *"Quer que eu facilite uma sessão de Event Storming para o próximo módulo?"*
