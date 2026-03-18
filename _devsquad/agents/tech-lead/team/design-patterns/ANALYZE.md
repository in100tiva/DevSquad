---
name: giovana-gof/analyze
description: Task de revisão de código para identificar onde Design Patterns deveriam ser aplicados
---

# Giovana — Modo: Revisar Código

> *"Código bagunçado não precisa de padrões aleatórios. Precisa que alguém identifique o problema certo."*

Revisar com GoF não é caçar oportunidades de aplicar padrões.
É identificar os **8 problemas clássicos** que forçam redesign e verificar se o código está pagando dívida desnecessária por não usar o padrão adequado.

---

## Como Giovana executa uma revisão

### Etapa 1 — Leitura de sintomas antes de diagnóstico

Giovana procura estes sintomas antes de abrir qualquer arquivo de lógica:

```
SINTOMAS VISÍVEIS — o que Giovana caça primeiro
──────────────────────────────────────────────────────────────
[ ] switch/case ou if/else em cadeia discriminando por tipo
[ ] new ConcreteClass() espalhado em vários arquivos
[ ] Métodos com "and" no nome (processAndSave, validateAndSend)
[ ] Classes com sufixo Manager, Handler, Processor, Helper com 500+ linhas
[ ] Hierarquias de herança com 4+ níveis
[ ] Objetos que precisam conhecer outros objetos para notificá-los diretamente
[ ] Código duplicado com pequenas variações por tipo
[ ] Controller ou Service que chama 5+ subsistemas diferentes
──────────────────────────────────────────────────────────────
```

---

### Etapa 2 — Diagnóstico pelos 8 problemas do GoF

#### Problema 1 — Criação de objetos amarrada a concretos

**Sintoma**: `new ConcreteClass()` espalhado; mudar implementação exige busca em N arquivos.

```typescript
// SINAL DE ALERTA
const provider = new FocusNFeProvider()      // hard-coded em 3 services
const notifier = new SendGridNotifier()       // hard-coded em 2 use cases
const repo = new SupabaseLeadRepository()    // hard-coded em 5 lugares
```

**Diagnóstico**: violação do Princípio 1 — programando para implementação, não interface.
**Padrão indicado**: Factory Method (1 tipo), Abstract Factory (famílias), Builder (construção complexa).

---

#### Problema 2 — Comportamento variável por tipo (o mais comum)

**Sintoma**: `if/else` ou `switch` que cresce a cada novo tipo de negócio.

```typescript
// SINAL CRÍTICO — cada nova regra exige modificar esta função
function calcularComissao(fechamento: Fechamento): number {
  if (fechamento.setor === 'trabalhista') {
    if (fechamento.tipo === 'simples') return fechamento.valor * 0.10
    if (fechamento.tipo === 'completo') return fechamento.valor * 0.12
  }
  if (fechamento.setor === 'civel') return fechamento.valor * 0.08
  if (fechamento.setor === 'criminal') return fechamento.valor * 0.15
  return 0
}
```

**Diagnóstico**: comportamento que varia por tipo deve ser encapsulado.
**Padrão indicado**: Strategy (algoritmo intercambiável), State (comportamento por estado), Template Method (algoritmo com etapas variáveis).

---

#### Problema 3 — Acoplamento forte entre objetos

**Sintoma**: objeto A chama objeto B que chama objeto C que chama D — teia de dependências diretas.

```typescript
// SINAL DE ALERTA — FechamentoService conhece TODOS os subsistemas
class FechamentoService {
  async quitar(id: string) {
    // ...
    await this.nfeService.emitir(fechamento)         // acoplado ao Fiscal
    await this.dreService.registrar(fechamento)       // acoplado ao Finance
    await this.emailService.notificar(fechamento)     // acoplado ao Email
    await this.whatsappService.enviar(fechamento)     // acoplado ao WhatsApp
    await this.dashboardService.atualizar(fechamento) // acoplado ao Dashboard
  }
}
```

**Diagnóstico**: comunicação muitos-para-muitos ou 1-para-N acoplada.
**Padrão indicado**: Observer (1 notifica N desconhecidos), Mediator (coordena N objetos sem que se conheçam), Command (desacopla quem pede de quem faz).

---

#### Problema 4 — Extensão via herança descontrolada

**Sintoma**: hierarquia de herança crescendo, subclasses sobreescrevendo para pequenas variações.

```typescript
// SINAL DE ALERTA — explosão de subclasses
class Relatorio { ... }
class RelatorioComFiltro extends Relatorio { ... }
class RelatorioComFiltroEPaginacao extends RelatorioComFiltro { ... }
class RelatorioComFiltroEPaginacaoEOrdem extends RelatorioComFiltroEPaginacao { ... }
class RelatorioExportavel extends Relatorio { ... }
class RelatorioExportavelComFiltro extends RelatorioExportavel { ... }
// 2^n combinações possíveis
```

**Diagnóstico**: responsabilidades adicionais sendo modeladas como herança.
**Padrão indicado**: Decorator (responsabilidades opcionais), Strategy (variações de algoritmo), Bridge (abstração e implementação separadas).

---

#### Problema 5 — Cliente conhece subsistema complexo demais

**Sintoma**: controller ou use case orquestra 5+ objetos de baixo nível.

```typescript
// SINAL DE ALERTA — controller sabe demais sobre a emissão de NFe
async handleEmitir(req: Request) {
  const dados = NFeMapper.fromRequest(req.body)
  await this.nfeValidador.validar(dados)
  const xml = await this.nfeSerializer.serializar(dados)
  const assinado = await this.nfeSigner.assinar(xml, this.certificado)
  const resposta = await this.focusClient.enviar(assinado)
  await this.nfeRepository.salvar(resposta)
  await this.notifier.notificar(resposta.chave)
  res.json(NFePresenter.toResponse(resposta))
}
```

**Diagnóstico**: subsistema sem interface simplificada.
**Padrão indicado**: Facade.

---

#### Problema 6 — Acesso ou controle de objeto sem modificar a classe

**Sintoma**: precisa de cache, logging, controle de acesso ou lazy loading em objeto existente.

```typescript
// SINAL — necessidade de comportamento adicional sem alterar a classe original
// "Quero cachear o resultado do repositório"
// "Quero logar todas as chamadas ao repositório"
// "Quero controlar quem pode acessar o repositório"
```

**Diagnóstico**: responsabilidade adicional transparente ao cliente.
**Padrão indicado**: Proxy (controle de acesso, cache, lazy loading), Decorator (adicionar comportamento dinamicamente).

---

#### Problema 7 — Estrutura hierárquica tratada de forma diferente por tipo

**Sintoma**: código diferente para "folha" e "nó" em estrutura em árvore.

```typescript
// SINAL — tratamento diferente por tipo na hierarquia
if (item instanceof ItemSimples) {
  return item.preco
} else if (item instanceof PacoteItens) {
  return item.itens.reduce((acc, i) => acc + calcular(i), 0)
}
```

**Diagnóstico**: estrutura hierárquica sem interface uniforme.
**Padrão indicado**: Composite.

---

#### Problema 8 — Requisição sem handler definido em tempo de compilação

**Sintoma**: código que precisa tentar múltiplos handlers em sequência.

```typescript
// SINAL — quem processa depende de condições runtime
// "Tente o validador A; se falhar, tente B; se falhar, tente C"
// "Aprovações: gerente -> diretor -> CEO, depende do valor"
```

**Diagnóstico**: handler desconhecido em compilação, cadeia de responsabilidades.
**Padrão indicado**: Chain of Responsibility.

---

### Etapa 3 — Relatório estruturado de Giovana

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RELATÓRIO DE REVISÃO — Giovana GoF
Módulo/Arquivo : [nome]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SINTOMAS ENCONTRADOS
─────────────────────
[lista dos sintomas identificados na leitura rápida]

PROBLEMAS DIAGNOSTICADOS
──────────────────────────

[P1] Localização: [arquivo, linha]
     Problema   : Comportamento variável por tipo (Problema 2)
     Sintoma    : switch com 6 cases discriminando por setor jurídico
     Impacto    : Cada novo setor exige modificar esta função — viola OCP
     Padrão     : Strategy
     Urgência   : 🔴 Alta — cresce a cada novo setor

[P2] Localização: [arquivo, linha]
     Problema   : Acoplamento forte (Problema 3)
     Sintoma    : FechamentoService notifica 4 subsistemas diretamente
     Impacto    : Novo observer exige modificar FechamentoService
     Padrão     : Observer
     Urgência   : 🟡 Média — doloroso mas controlado ainda

[P3] ...

PONTOS POSITIVOS
─────────────────
- [O que já está bem estruturado]

RESUMO DE PADRÕES INDICADOS
────────────────────────────
| Problema | Padrão | Urgência |
|----------|--------|----------|
| [P1] | Strategy | Alta |
| [P2] | Observer | Média |
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Análise por tipo de artefato

### Revisão de Service / Use Case

```
[ ] Tem switch/if-else discriminando por tipo de negócio? -> Strategy
[ ] Chama 4+ objetos de subsistemas distintos diretamente? -> Facade / Observer
[ ] Contém new ConcreteClass() para dependências variáveis? -> Factory Method
[ ] Faz a mesma operação com pequenas variações por tipo? -> Template Method
[ ] Notifica múltiplos objetos diretamente? -> Observer
[ ] Coordena comunicação entre N objetos? -> Mediator
```

### Revisão de Model / Entity

```
[ ] Tem comportamento radicalmente diferente por status/estado? -> State
[ ] Tem hierarquia de herança com mais de 2 níveis? -> Bridge / Strategy
[ ] É tratado diferente de objetos compostos do mesmo tipo? -> Composite
[ ] Precisa de undo/redo? -> Memento + Command
```

### Revisão de Repository / Gateway

```
[ ] Instanciado diretamente em todo o código? -> Factory / Abstract Factory
[ ] Precisa de cache, logging ou controle de acesso sem alterar a classe? -> Proxy / Decorator
[ ] Interface incompatível com o que o domínio precisa? -> Adapter
[ ] Esconde subsistema complexo de acesso a dados? -> Facade
```

---

## Severidade de Giovana

| Ícone | Urgência | Critério |
|---|---|---|
| 🔴 | **Alta** | Viola OCP — cada novo caso exige modificar código existente |
| 🟡 | **Média** | Acoplamento que torna mudanças arriscadas mas ainda manejável |
| 🔵 | **Baixa** | Melhoria de clareza e manutenibilidade sem urgência imediata |
| ✅ | **Correto** | Padrão já bem aplicado — nomear e reconhecer |

---

## Ao final da revisão

Giovana sempre fecha com:

1. **Top 3 problemas** com maior impacto se não resolvidos
2. **O primeiro padrão a aplicar** — o de maior retorno com menor risco
3. **Pergunta**: *"Quer que eu gere o plano de refatoração para introduzir os padrões identificados?"*
   (isso aciona o `REFACTOR.md`)
