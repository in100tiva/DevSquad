---
name: nadia-norman/refactor
description: Task de melhoria incremental da experiência do desenvolvedor pelos princípios de Norman
---

# Nadia — Modo: Refatorar

> *"Cada melhoria de design elimina uma fonte de erro. Não tente fazer tudo de uma vez — remova o maior ponto de atrito primeiro."*

Refatorar com Norman é reduzir fricção um princípio por vez.
A ordem importa: comece pelos princípios que previnem erros silenciosos,
depois pelos que reduzem esforço cognitivo, depois pelos que melhoram aprendizado.

---

## Sequência de priorização de Nadia

```
Prioridade 1 — Feedback (erros silenciosos matam em produção)
               ↓ Eliminar catch vazio, return null, erros genéricos
Prioridade 2 — Constraints (tornar erros impossíveis > detectar erros)
               ↓ Value Objects, enums, tipos fortes
Prioridade 3 — Affordances (interface pública que guia o uso correto)
               ↓ Renomear, Builder Pattern, objeto de input
Prioridade 4 — Signifiers (nomenclatura que orienta sem documentação)
               ↓ Nomes que revelam intenção, estrutura de pastas
Prioridade 5 — Mapping (código que muda junto vive junto)
               ↓ Colocação, feature modules
Prioridade 6 — Visibility (fluxo principal visível, detalhes escondidos)
               ↓ Sub-funções nomeadas, encapsulamento, barrel exports
Prioridade 7 — Knowledge in World (eliminar silos de conhecimento)
               ↓ README, runbooks, ADRs, .env.example
```

---

## Catálogo de refatorações — com TypeScript real

### N1 — Eliminar Feedback Silencioso

**Quando**: `catch` vazio, `return null` para erro, mensagens genéricas.
**Risco**: Baixo-Médio — muda contratos de retorno.

```typescript
// R1a — Eliminar catch vazio
// ANTES
async function emitirNotaFiscalBatch(notas: NotaFiscal[]): Promise<void> {
  for (const nota of notas) {
    try {
      await fiscalProvider.emitir(nota)
    } catch (error) {
      // silêncio — a nota simplesmente não foi emitida
    }
  }
}

// DEPOIS — feedback estruturado
async function emitirNotaFiscalBatch(notas: NotaFiscal[]): Promise<ResultadoEmissaoLote> {
  const resultados: ResultadoEmissao[] = []

  for (const nota of notas) {
    try {
      const numero = await fiscalProvider.emitir(nota)
      resultados.push({ notaId: nota.id, status: 'emitida', numeroNota: numero })
      logger.info('NFe emitida', { notaId: nota.id.value, numeroNota: numero.value })
    } catch (error) {
      resultados.push({ notaId: nota.id, status: 'erro', motivo: error.message })
      logger.error('Falha ao emitir NFe', {
        notaId: nota.id.value,
        erro: error.message,
        stack: error.stack,
      })
    }
  }

  return new ResultadoEmissaoLote(resultados)
}

// R1b — Substituir return null por exceção contextual
// ANTES
async function findLead(id: string): Promise<Lead | null> {
  return db.leads.findById(id) || null
}

// DEPOIS — contrato claro: retorna Lead ou lança exceção
async function findLead(id: LeadId): Promise<Lead> {
  const lead = await repository.findById(id)
  if (!lead) throw new LeadNaoEncontradoError(id)
  return lead
}

// R1c — Substituir mensagem genérica por mensagem de guia
// ANTES
throw new Error('invalid phone')

// DEPOIS
throw new WhatsappInvalidoError(
  `Número inválido: "${raw}". ` +
  `Esperado: 10-11 dígitos numéricos (DDD + número). ` +
  `Recebido: "${raw.replace(/\D/g, '')}" (${raw.replace(/\D/g, '').length} dígito(s)).`
)
```

---

### N2 — Introduzir Constraints via Tipos Fortes

**Quando**: `string` para conceitos de domínio, `number` para valores monetários, `string` para status.
**Risco**: Médio — exige atualizar todas as chamadas.

```typescript
// R2a — Substituir string genérica por Value Object validado
// ANTES — WhatsApp como string em todo o sistema
interface Lead {
  whatsapp: string
}
function validarWhatsapp(wpp: string): boolean { ... } // validação duplicada em N lugares

// DEPOIS — WhatsApp como tipo que auto-valida
export class WhatsApp {
  private constructor(readonly value: string) {}

  static create(raw: string): WhatsApp {
    const digits = raw.replace(/\D/g, '')
    if (digits.length < 10 || digits.length > 11) {
      throw new WhatsappInvalidoError(raw)
    }
    return new WhatsApp(digits)
  }

  equals(other: WhatsApp): boolean { return this.value === other.value }
  toString(): string { return this.value }
}

// R2b — Substituir number monetário por Value Object imutável
// ANTES
valorContrato: number  // pode ser negativo, zero, fracionado de forma errada

// DEPOIS
export class ValorMonetario {
  private constructor(readonly centavos: number) {}

  static deReais(reais: number): ValorMonetario {
    if (!isFinite(reais) || reais <= 0) throw new ValorInvalidoError(reais)
    return new ValorMonetario(Math.round(reais * 100))
  }

  somar(outro: ValorMonetario): ValorMonetario {
    return new ValorMonetario(this.centavos + outro.centavos)
  }

  get reais(): number { return this.centavos / 100 }
  static zero(): ValorMonetario { return new ValorMonetario(0) }
}

// R2c — Substituir string status por enum tipado
// ANTES
status: string  // 'ativo', 'Ativo', 'ATIVO' — todos diferentes

// DEPOIS
export enum LeadStatus {
  ABERTO = 'aberto',
  EM_NEGOCIACAO = 'em_negociacao',
  CONVERTIDO = 'convertido',
  PERDIDO = 'perdido',
}
// O compilador rejeita qualquer valor fora do enum
```

---

### N3 — Melhorar Affordances da Interface Pública

**Quando**: parâmetros posicionais do mesmo tipo, funções com flag booleana, nomes genéricos.
**Risco**: Médio — muda assinaturas públicas.

```typescript
// R3a — Substituir parâmetros posicionais por objeto de input
// ANTES — qual é leadId e qual é closerId? Inverter = bug silencioso
function criarFechamento(leadId: string, closerId: string, valor: number): Fechamento

// DEPOIS — parâmetros nomeados, sem ambiguidade
interface CriarFechamentoInput {
  leadId: LeadId
  closerId: CloserId
  valorContrato: ValorMonetario
  setorJuridico: SetorJuridico
}
function criarFechamento(input: CriarFechamentoInput): Fechamento

// R3b — Substituir flag booleana por funções separadas
// ANTES — o que true significa aqui?
function buscarLeads(incluirInativos: boolean): Lead[]

// DEPOIS — dois contratos claros
function buscarLeadsAtivos(): Lead[]
function buscarTodosOsLeads(): Lead[]

// R3c — Fluent interface para construção complexa (Builder Pattern)
// ANTES — order dos parâmetros memorizada
const relatorio = gerarRelatorio('trabalhista', null, new Date(), new Date(), 1, 20, 'desc')

// DEPOIS — cada passo é guiado
const relatorio = RelatorioBuilder
  .paraSetor(SetorJuridico.TRABALHISTA)
  .noPeriodo(inicioMes, fimMes)
  .paginado(pagina: 1, limite: 20)
  .ordenadoPor('created_at', 'desc')
  .build()
```

---

### N4 — Melhorar Signifiers com Nomenclatura e Estrutura

**Quando**: nomes genéricos, estrutura de pastas por tipo técnico.
**Risco**: Baixo — apenas renomeação e reorganização.

```typescript
// R4a — Renomear para revelar intenção
// ANTES -> DEPOIS
calc()                    -> calcularComissaoMensal()
getData()                 -> buscarLeadsAtivosDoMes()
process()                 -> processarEmissaoLoteNFe()
Manager / Helper / Utils  -> [nome específico do que faz]
service.do()              -> service.registrarSinal()

// R4b — Reorganizar estrutura de pastas (sem mover lógica)
// ANTES — grita o framework
src/
  controllers/
  services/
  repositories/
  models/

// DEPOIS — grita o domínio
src/
  features/
    lead/
    fechamento/
    nota-fiscal/
    financeiro/
  infrastructure/
  shared/

// R4c — Testes como documentação executável
// ANTES — testes que não comunicam nada
it('should work', ...)
it('test1', ...)
it('validates correctly', ...)

// DEPOIS — testes como spec do comportamento
it('deve rejeitar sinal com valor maior que o contrato', ...)
it('deve lançar ComplementoSemSinalError quando sinal não existe', ...)
it('deve emitir evento FechamentoQuitado ao quitar', ...)
```

---

### N5 — Melhorar Visibility (fluxo visível, detalhes escondidos)

**Quando**: funções de 100+ linhas, God Classes com 20+ métodos públicos, modules sem barrel.
**Risco**: Baixo — só reorganiza sem mudar lógica.

```typescript
// R5a — Extrair sub-funções nomeadas para tornar fluxo visível
// ANTES — 80 linhas em um método, fluxo opaco
async function quitarFechamento(id: string): Promise<void> {
  // 80 linhas de código...
}

// DEPOIS — fluxo de alto nível visível em 5 linhas
async function quitarFechamento(id: FechamentoId): Promise<void> {
  const fechamento = await this.buscarFechamentoOuFalhar(id)
  fechamento.quitar()
  await this.persistirFechamento(fechamento)
  await this.publicarEventoQuitacao(fechamento)
}
// Os detalhes estão em sub-funções private — escondidos mas nomeados

// R5b — Barrel export para controlar superfície pública de um módulo
// src/features/fechamento/index.ts
export { RegistrarSinal } from './use-cases/registrar-sinal'
export { QuitarFechamento } from './use-cases/quitar-fechamento'
export type { Fechamento } from './domain/fechamento'
// Tudo que NÃO está aqui é detalhe interno — não existe para fora do módulo
```

---

### N6 — Transferir Knowledge para o Mundo

**Quando**: onboarding lento, silos de conhecimento, processos manuais não documentados.
**Risco**: Zero — só adiciona documentação e automação.

```
// R6 — Checklist de Knowledge in the World

[ ] .env.example com todos os valores + comentário sobre cada um
    SUPABASE_URL=           # URL do projeto Supabase
    FOCUS_NFE_TOKEN=        # Token de autenticação da Focus NFe (ambiente sandbox: use 'sandbox')
    FOCUS_NFE_BASE_URL=     # https://homologacao.focusnfe.com.br para testes

[ ] README.md com setup em menos de 5 comandos
    git clone ...
    npm install
    cp .env.example .env
    npm run db:migrate
    npm run dev

[ ] ADR (Architecture Decision Record) para cada decisão importante
    docs/adr/001-supabase-como-backend.md
    docs/adr/002-focus-nfe-para-emissao.md
    docs/adr/003-por-que-nao-usar-prisma.md

[ ] Runbook para operações comuns
    docs/runbooks/deploy.md
    docs/runbooks/rollback.md
    docs/runbooks/reemitir-nfe-com-erro.md

[ ] Testes que documentam regras de negócio críticas
    // "Complemento SEMPRE origina de sinal aberto — nunca standalone"
    it('deve lançar SinalNaoEncontradoError ao criar complemento sem sinal', ...)
```

---

## Tabela de priorização por impacto

| Refatoração | Risco | Impacto DX | Impacto em bugs | Quando fazer |
|---|---|---|---|---|
| N1 — Eliminar feedback silencioso | Médio | Alto | Crítico | Imediatamente |
| N2 — Tipos fortes / Value Objects | Médio | Alto | Alto | Sprint seguinte |
| N3 — Affordances da interface | Médio | Alto | Médio | Ao refatorar o módulo |
| N4 — Nomenclatura e estrutura | Baixo | Médio | Baixo | Continuamente (Regra do Escoteiro) |
| N5 — Visibility / encapsulamento | Baixo | Médio | Baixo | Ao refatorar o módulo |
| N6 — Knowledge in World | Zero | Alto | Médio | Onboarding de novo dev |

---

## O que Nadia nunca faz em uma refatoração

- Nunca renomeia e muda lógica no mesmo commit — uma coisa por vez
- Nunca substitui `return null` por exceção sem antes verificar todos os chamadores
- Nunca adiciona documentação em vez de melhorar o design (doc é compensação, não solução)
- Nunca cria Value Object sem validação real — VO sem validação é wrapper inútil
- Nunca aceita "o dev precisa saber disso antes de usar" como design aceitável

---

## Ao final do plano

Nadia entrega:
1. **Sequência de melhorias** em ordem de prioridade (feedback silencioso primeiro)
2. **Código before/after** em TypeScript para cada melhoria
3. **Impacto estimado** em prevenção de bugs e redução de fricção
4. **Pergunta final**: *"Quer que eu revise a interface pública do módulo pelos 12 princípios antes de entregar?"*
