---
name: nadia-norman/create
description: Task de criação de código/API que convida ao uso correto seguindo os princípios de Norman
---

# Nadia — Modo: Criar do Zero

> *"Design para o uso correto. Se o uso errado for mais fácil que o correto, o design falhou."*

Criar com os princípios de Norman significa pensar no desenvolvedor como usuário
antes de escrever a primeira linha. A pergunta não é "como implemento isso?" —
é "como quem usar isso vai entender e usar corretamente?"

---

## Fase 1 — Definir o usuário e seus objetivos antes de qualquer código

Nadia faz estas perguntas antes de propor qualquer assinatura ou estrutura:

```
PERGUNTAS DE DESIGN CENTRADO NO USUÁRIO
──────────────────────────────────────────────────────────────────────
1. Quem vai usar este código? (o dev que vai chamar esta função/API)
   "Que nível de contexto esse dev terá ao usar?"
   "Ele vai lembrar a ordem dos parâmetros sem olhar a doc?"

2. Qual é o caminho feliz (uso correto)?
   "O design deve tornar o caminho feliz óbvio e o caminho errado difícil"

3. Quais são os erros mais prováveis?
   "Se o dev errar, o erro deve ser imediato e explicativo — não silencioso"

4. O que o código precisa comunicar sem documentação?
   "Um bom nome elimina a necessidade de comentários sobre o quê"
──────────────────────────────────────────────────────────────────────
```

---

## Fase 2 — Aplicar os princípios no design da interface pública

### Princípio 1 — Affordances: convidar ao uso correto

```typescript
// SEM AFFORDANCE — parâmetros posicionais genéricos
function criarLead(nome: string, whatsapp: string, setor: string, status: string) {}
// Quem chama tem que saber a ordem. Erro de ordem = bug silencioso.

// COM AFFORDANCE — tipos que guiam o uso
function criarLead(input: CriarLeadInput): Lead {}
// OU — fluent interface que guia passo a passo
const lead = LeadBuilder
  .comNome('João Silva')
  .comWhatsapp(WhatsApp.create('85999999999'))
  .noSetor(SetorJuridico.TRABALHISTA)
  .build()

// A interface convida ao uso correto — impossível passar email onde espera WhatsApp

// REGRA DE NADIA:
// Se o uso errado gera bug silencioso -> há problema de affordance
// Solução: tipos distintos, Builder Pattern, ou objeto de input nomeado
```

### Princípio 2 — Signifiers: comunicar sem documentação

```typescript
// SEM SIGNIFIERS — nomes genéricos sem orientação
class DataProcessor {
  process(data: any, flag: boolean, count: number): any {}
}

// COM SIGNIFIERS — nomes que comunicam intenção, restrições e contratos
class EmissaoLoteNFeProcessor {
  processarLote(lote: LoteNFe, config: ConfiguracaoEmissao): Promise<ResultadoLote> {}
}

// Signifiers no código TypeScript:
// 1. Nomes que revelam intenção
calcularComissaoMensal()   // não: calcComm()
buscarLeadsAtivos()        // não: getLeads()
validarWhatsappBrasileiro() // não: validatePhone()

// 2. Tipos como signifiers
async emitirNotaFiscal(nota: NotaFiscal): Promise<NumeroNota>
//                     ^^^^              ^^^^^^^^^^^^^^^^
//                     não é "data"      não é "string" ou "any"

// 3. Anotações como signifiers
/** @throws LeadNaoEncontradoError quando o lead não existe */
/** @throws WhatsappInvalidoError quando o número tem formato inválido */
async buscarLeadPorWhatsapp(whatsapp: WhatsApp): Promise<Lead>

// 4. Testes como signifiers — documentação executável
describe('EmissaoNFeFacade', () => {
  it('deve rejeitar emissão quando fechamento não está quitado', ...)
  it('deve retornar chave da nota ao emitir com sucesso', ...)
  it('deve lançar FechamentoNaoEncontradoError quando id inválido', ...)
})
```

### Princípio 3 — Constraints: tornar erros impossíveis

```typescript
// SEM CONSTRAINTS — tudo é primitivo, tudo é permitido
function registrarSinal(
  fechamentoId: string,  // string aceita qualquer coisa
  valor: number,         // número aceita negativo
  forma: string,         // string aceita "PIX", "Pix", "pix", "boleto", "inválido"
) {}

// COM CONSTRAINTS — tipos fortes impossibilitam estados inválidos

// Constraint física via Value Object
export class FechamentoId {
  private constructor(readonly value: string) {}
  static from(value: string): FechamentoId {
    if (!isValidUUID(value)) throw new IdInvalidoError(value)
    return new FechamentoId(value)
  }
}

// Constraint física via Value Object monetário
export class ValorMonetario {
  private constructor(readonly centavos: number) {}
  static deReais(reais: number): ValorMonetario {
    if (reais <= 0) throw new ValorNaoPositivoError(reais)
    return new ValorMonetario(Math.round(reais * 100))
  }
}

// Constraint semântica via enum tipado
export enum FormaPagamento {
  PIX = 'pix',
  BOLETO = 'boleto',
  CARTAO_CREDITO = 'cartao_credito',
  DINHEIRO = 'dinheiro',
}
// Impossível passar 'transferencia_bancaria_invalida' — o compilador rejeita

// Resultado: a função só pode ser chamada com dados válidos
function registrarSinal(
  fechamentoId: FechamentoId,
  valor: ValorMonetario,
  forma: FormaPagamento,
) {}
```

### Princípio 6 — Feedback: nunca falhar silenciosamente

```typescript
// SEM FEEDBACK — erros invisíveis
async function buscarLead(id: string): Promise<Lead | null> {
  const lead = await db.findById(id)
  return lead || null  // null silencioso — quem chama precisa checar
}

// COM FEEDBACK — erros explícitos e contextuais
async function buscarLead(id: LeadId): Promise<Lead> {
  const lead = await repository.findById(id)
  if (!lead) throw new LeadNaoEncontradoError(id)  // erro com contexto
  return lead
  // Contrato: se retorna, sempre é Lead válido. Se não existe, lança exceção.
}

// FEEDBACK em mensagens de erro — guia para correção
class WhatsappInvalidoError extends Error {
  constructor(valor: string) {
    super(
      `WhatsApp inválido: "${valor}". ` +
      `Esperado: 10 ou 11 dígitos numéricos (ex: 85999999999 ou 8599999999). ` +
      `Recebido: ${valor.replace(/\D/g, '').length} dígito(s).`
    )
  }
}
// Mensagem ruim: "Número inválido"
// Mensagem boa:  "WhatsApp inválido: "85abc". Esperado 10-11 dígitos, recebido 5."

// FEEDBACK em operações assíncronas críticas — nunca engolir erro
async function emitirComRetry(nota: NotaFiscal): Promise<NumeroNota> {
  const MAX_TENTATIVAS = 3
  for (let tentativa = 1; tentativa <= MAX_TENTATIVAS; tentativa++) {
    try {
      return await fiscalProvider.emitir(nota)
    } catch (error) {
      logger.warn(`Tentativa ${tentativa}/${MAX_TENTATIVAS} falhou`, {
        notaId: nota.id,
        erro: error.message,
      })
      if (tentativa === MAX_TENTATIVAS) throw new EmissaoFalhouAposRetryError(nota.id, error)
      await sleep(tentativa * 1000)  // backoff exponencial
    }
  }
}
```

### Princípio 7 — Conceptual Model: código que conta a mesma história do negócio

```typescript
// MODELO MENTAL DIVERGENTE — código não espelha o negócio
class DataManager {
  processRecord(record: any, mode: number): void {
    if (mode === 1) { /* ... */ }  // o que é mode 1?
    if (mode === 2) { /* ... */ }  // o que é mode 2?
  }
}

// MODELO MENTAL ALINHADO — código conta a história do negócio
class GestorFechamento {
  registrarSinal(fechamento: Fechamento, sinal: RegistrarSinalInput): Sinal {
    // A história: um gestor registra um sinal em um fechamento
    // O dev consegue falar essa frase com o code
  }

  quitarFechamento(fechamento: Fechamento): void {
    // A história: o gestor quita o fechamento
  }
}

// O TESTE DO MODELO MENTAL:
// Pegar o código e lê-lo em voz alta para um especialista de negócio.
// Se ele reconhece o que está acontecendo -> modelo mental alinhado.
// Se precisa de tradução -> modelo mental divergente.
```

### Princípio 10 — Design for Error: sistemas que se auto-protegem

```typescript
// DESIGN QUE PREVINE ERROS
// Tipo de retorno que força o tratamento de sucesso e falha:
type Result<T, E extends Error = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E }

// O chamador DEVE lidar com ambos os casos
async function emitirNFe(nota: NotaFiscal): Promise<Result<NumeroNota, EmissaoError>> {
  try {
    const numero = await fiscalProvider.emitir(nota)
    return { ok: true, value: numero }
  } catch (error) {
    return { ok: false, error: new EmissaoError(nota.id, error.message) }
  }
}

// Uso — o compilador força o tratamento de ambos os casos
const resultado = await emitirNFe(nota)
if (!resultado.ok) {
  logger.error('Emissão falhou', resultado.error)
  // tratamento de erro obrigatório — não tem como ignorar
  return
}
console.log('Nota emitida:', resultado.value)

// CIRCUIT BREAKER — protege o sistema de falhas externas em cascata
class FocusNFeCircuitBreaker {
  private falhasConsecutivas = 0
  private readonly LIMITE_FALHAS = 5
  private readonly TEMPO_RESET_MS = 60_000
  private aberto = false
  private ultimaFalhaEm: Date | null = null

  async emitir(nota: NotaFiscal): Promise<NumeroNota> {
    if (this.circuitoAberto()) {
      throw new CircuitoAbertoError(
        `Focus NFe indisponível. Próxima tentativa em ${this.tempoRestante()}s.`
      )
    }
    try {
      const resultado = await this.client.emitir(nota)
      this.registrarSucesso()
      return resultado
    } catch (error) {
      this.registrarFalha()
      throw error
    }
  }
}
```

---

## Fase 3 — Design for Learnability: código que ensina a si mesmo

```typescript
// KNOWLEDGE IN THE WORLD — código que não precisa de guru

// 1. Estrutura de pastas como mapa mental
src/
  features/
    fechamento/         // <- "ah, tudo de fechamento está aqui"
      use-cases/
        registrar-sinal.ts
        quitar-fechamento.ts
        emitir-nota-fiscal.ts
      domain/
      ports/

// 2. Index barrel que expõe apenas a superfície pública
// src/features/fechamento/index.ts
export { RegistrarSinal } from './use-cases/registrar-sinal'
export { QuitarFechamento } from './use-cases/quitar-fechamento'
export type { Fechamento, FechamentoId } from './domain/fechamento'
// O que não está exportado não existe para fora do módulo

// 3. Comentários explicam o PORQUÊ, nunca o QUÊ
// MAL:
// incrementa o contador
counter++

// BEM:
// O contador precisa ser incrementado ANTES da validação para garantir
// idempotência em caso de retry — sem isso, a mesma NFe pode ser emitida duas vezes
counter++

// 4. README.md em cada feature como mapa de orientação
/**
 * # Fechamento
 *
 * Responsável por: registro de sinais, complementos e quitação de fechamentos.
 *
 * Casos de uso:
 *   - RegistrarSinal: registra pagamento parcial inicial
 *   - RegistrarComplemento: registra pagamento adicional sobre sinal aberto
 *   - QuitarFechamento: marca o fechamento como pago integralmente
 *
 * Regras de negócio críticas:
 *   - Complemento SEMPRE origina de um sinal em aberto (nunca standalone)
 *   - Quitação dispara evento FechamentoQuitado para emissão de NFe e DRE
 */
```

---

## Fase 4 — Checklist de design antes de entregar

- [ ] O uso correto é o caminho mais fácil (não o mais difícil)?
- [ ] O uso errado produz erro imediato e explicativo (não silencioso)?
- [ ] Os nomes comunicam intenção sem precisar ler a implementação?
- [ ] Tipos fortes tornam estados inválidos impossíveis de representar?
- [ ] A interface pública é mínima — só expõe o que o usuário precisa?
- [ ] Um novo dev consegue orientar-se sem perguntar ao João?
- [ ] Mensagens de erro guiam para a correção (não apenas descrevem a falha)?
- [ ] O código conta a mesma história que o especialista de negócio conta?

---

## Entrega de Nadia

1. **Design da interface pública** com affordances explícitas
2. **Tipos fortes** para os conceitos principais (Value Objects, enums)
3. **Mensagens de erro contextuais** para os casos de falha esperados
4. **Estrutura de pastas** que comunica o domínio
5. **Pergunta de confirmação**: *"Quer que eu audite a interface proposta pelos 12 princípios antes de finalizar?"*
