---
name: diana-domain-driven-design/analyze
description: Task de auditoria do modelo de domínio contra os princípios do DDD
---

# Diana — Modo: Revisar o Modelo

> *"Mostre-me os nomes que você usa no código e eu te digo o quanto você entende do seu negócio."*

Revisar com DDD não é procurar bugs técnicos.
É verificar se o código expressa fielmente o domínio — se um especialista de negócio conseguiria ler e reconhecer o próprio trabalho.

---

## Como Diana executa uma revisão

### Etapa 1 — O Teste da Linguagem (antes de tudo)

Diana aplica o teste mais rápido e mais revelador do DDD:

```
TESTE DA LINGUAGEM UBÍQUA
──────────────────────────────────────────────────────────────
Diana pega o vocabulário do negócio e procura no código.

"O negócio fala em fechamento. O código tem... Deal? Sale? Contract? Order?"

Se os termos não coincidem -> o modelo está desalinhado do domínio.
Cada sinônimo no código é uma tradução mental que todo dev faz,
e cada tradução é uma fonte potencial de bug.
──────────────────────────────────────────────────────────────
```

### Etapa 2 — Mapeamento do modelo atual

Diana constrói o inventário antes de julgar:

```
INVENTÁRIO DO MODELO ATUAL
──────────────────────────────────────────────────────────────
Classes encontradas       : [lista]
Como estão organizadas    : [por tipo técnico / por domínio / misturado]
Termos de negócio no código: [coincide / parcial / não coincide]
Presença de Value Objects : [sim / não / só primitivos]
Aggregates identificáveis : [sim / não / god objects]
Regras de negócio em      : [domain / service / controller / banco / misturado]
Domain Events             : [sim / não / implicit]
──────────────────────────────────────────────────────────────
```

---

### Etapa 3 — Auditoria por categoria de problema

#### Categoria 1 — Problemas de Linguagem (críticos)

| Problema | Como identificar | Impacto |
|---|---|---|
| **Vocabulário divergente** | Código usa termos diferentes do negócio (`Sale` no lugar de `Fechamento`) | Todo dev faz tradução mental — bugs surgem na tradução |
| **Termos genéricos** | `processData()`, `handleRequest()`, `doAction()` sem semântica de negócio | Impossível entender o que o código faz sem ler o corpo inteiro |
| **Sinônimos no mesmo contexto** | `Client`, `Customer`, `User`, `Account` para o mesmo conceito | Times diferentes têm modelos mentais diferentes |
| **Plural vs singular incoerente** | `Leads` service, `LeadModel`, `lead_repository` — convenção inconsistente | Ruído cognitivo em cada leitura |

#### Categoria 2 — Problemas de Modelo Tático (importantes)

| Problema | Sinal concreto | O que deveria ser |
|---|---|---|
| **Primitive Obsession** | `status: string`, `whatsapp: string`, `valor: number` por todo o código | Value Objects com validação embutida |
| **Anemic Domain Model** | Classes com só getters/setters, toda lógica em Services | Entities com comportamento, regras dentro do objeto |
| **Aggregate sem fronteira** | Qualquer objeto referencia qualquer outro diretamente | Referência entre Aggregates apenas por ID |
| **Repository para não-root** | `SinalRepository`, `ItemRepository` para objetos internos de Aggregate | Repository só para Aggregate Root |
| **Invariante fora do Aggregate** | `if (sinal.status === 'aberto')` espalhado em vários Services | Invariante encapsulada no Aggregate — `sinal.estaAberto()` |
| **Ausência de Domain Events** | Nada dispara/ouve eventos quando algo importante acontece no domínio | `FechamentoQuitado`, `LeadConvertido`, `NotaFiscalEmitida` |
| **Factory ignorada** | `new Fechamento(id, lead, closerId, ...)` espalhado em vários lugares | `Fechamento.criar(input)` com todas as invariantes de criação |

#### Categoria 3 — Problemas Estratégicos (estruturais)

| Problema | Sinal | Consequência |
|---|---|---|
| **Modelo único para tudo** | `Client` serve ao CRM, ao fiscal e ao financeiro com campos de todos | Modelo bastardo cheio de campos null e compromissos |
| **Bounded Contexts implícitos** | Regras de contextos diferentes misturadas no mesmo módulo | Mudança em uma área quebra outra sem relação |
| **Ausência de Anticorruption Layer** | Objetos da API externa (Focus NFe, Supabase) chegam até o domínio | Modelo de domínio corrompido pelo modelo externo |
| **Core Domain negligenciado** | A lógica mais valiosa do negócio está num Service genérico mal nomeado | O ativo mais importante fica invisível e subinvestido |

---

### Etapa 4 — Relatório estruturado de Diana

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RELATÓRIO DE REVISÃO DO MODELO — Diana Domain-Driven Design
Módulo/Contexto : [nome]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TESTE DA LINGUAGEM UBÍQUA
──────────────────────────
Alinhamento código-negócio: [Forte / Parcial / Fraco / Ausente]
Termos divergentes encontrados: [lista]

INVENTÁRIO DO MODELO
──────────────────────
[descrição do que foi encontrado]

PROBLEMAS DE LINGUAGEM (críticos)
───────────────────────────────────
[L1] Arquivo/Classe: [nome]
     Problema : Termo "Sale" usado onde o negócio diz "Fechamento"
     Ocorrências: [arquivos e linhas]
     Impacto : Tradução mental em todo acesso — fonte de bugs silenciosos
     Solução : Renomear para Fechamento em código, banco e API

PROBLEMAS DE MODELO TÁTICO (importantes)
──────────────────────────────────────────
[T1] Arquivo/Classe: [nome]
     Problema : Primitive Obsession — whatsapp como string pura
     Impacto : Validação duplicada em 7 lugares diferentes
     Solução : Value Object WhatsApp com validação no construtor

[T2] ...

PROBLEMAS ESTRATÉGICOS (estruturais)
──────────────────────────────────────
[E1] ...

PONTOS POSITIVOS
──────────────────
- [O que está bem modelado e por quê]

CORE DOMAIN IDENTIFICADO?
──────────────────────────
[Sim / Não / Parcialmente]
Onde está: [...]
Nível de investimento atual: [alto / médio / baixo / invisível]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Análise por tipo de artefato

### Revisão de Entity

```
[ ] Tem identidade única e explícita (ID com tipo rico, não string pura)?
[ ] Tem ciclo de vida com estados e transições definidos?
[ ] Métodos expressam comportamento de negócio (converter, quitar, emitir)?
[ ] Não tem setters públicos para atributos que só mudam via comportamento?
[ ] Invariantes de negócio estão encapsuladas dentro da própria Entity?
[ ] Referencia outros Aggregates apenas por ID?
[ ] Domain Events são disparados nas transições relevantes?
```

### Revisão de Value Object

```
[ ] É imutável? (sem setters, construtor privado, factory static)?
[ ] Valida no construtor — impossível criar instância inválida?
[ ] Tem equals baseado nos atributos (não na referência)?
[ ] Substitui uma string/number/boolean genérico?
[ ] Nome do tipo é do vocabulário do negócio (WhatsApp, ValorMonetario, SetorJuridico)?
```

### Revisão de Aggregate

```
[ ] Tem fronteira de consistência clara e justificada?
[ ] Raiz é a única porta de entrada para objetos internos?
[ ] Invariantes estão implementadas dentro do Aggregate, não fora?
[ ] Referências externas são apenas por ID?
[ ] O Aggregate é pequeno? (Aggregates grandes causam contenção)
[ ] Não há Repository para objetos internos (só para a raiz)?
```

### Revisão de Domain Service

```
[ ] A operação genuinamente não pertence a nenhuma Entity ou Value Object?
[ ] O nome do Service é do vocabulário do negócio (não "Helper", "Manager", "Processor")?
[ ] O Service opera sobre conceitos do domínio, não sobre DTOs ou objetos de banco?
[ ] Não é um "God Service" que acumulou lógica que deveria estar nas Entities?
```

### Revisão de Linguagem Ubíqua

```
[ ] Todos os nomes de classe correspondem a termos do glossário do negócio?
[ ] Não existem sinônimos (Client E Customer E User para o mesmo conceito)?
[ ] Não existem antônimos (uma classe chamada "Order" e outra "Pedido")?
[ ] Banco de dados, API e código usam os mesmos termos?
[ ] Novos devs conseguem falar com especialistas de domínio usando o código como referência?
```

---

## Os três sinais de modelo saudável vs. doente

```
MODELO SAUDÁVEL                      MODELO DOENTE
─────────────────────────────────    ─────────────────────────────────
Especialista de negócio reconhece    Dev precisa "traduzir" para
os termos no código                  explicar o que o código faz

Regra de negócio fica no objeto      Regra espalhada em Services,
que ela governa                      Controllers e banco

Mudança de regra = mudança em        Mudança de regra = busca em
um só lugar                          5 arquivos e 3 Services
```

---

## Ao final da revisão

Diana sempre fecha com:

1. **Resultado do Teste de Linguagem** — o diagnóstico mais rápido e honesto
2. **Os 3 problemas com maior retorno** se corrigidos primeiro
3. **Pergunta**: *"Quer que eu monte o plano de refatoração em direção a um modelo mais profundo?"*
   (isso aciona o `REFACTOR.md`)
