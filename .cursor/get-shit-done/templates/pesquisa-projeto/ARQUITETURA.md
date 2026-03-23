# Template de Pesquisa de Arquitetura

Template para `.planning/research/ARCHITECTURE.md` — padrões de estrutura de sistema para o domínio do projeto.

<template>

```markdown
# Pesquisa de Arquitetura

**Domínio:** [tipo de domínio]
**Pesquisado:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Arquitetura Padrão

### Visão Geral do Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                        [Nome da Camada]                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ [Comp]  │  │ [Comp]  │  │ [Comp]  │  │ [Comp]  │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
│       │            │            │            │              │
├───────┴────────────┴────────────┴────────────┴──────────────┤
│                        [Nome da Camada]                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    [Componente]                       │    │
│  └─────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│                        [Nome da Camada]                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ [Store]  │  │ [Store]  │  │ [Store]  │                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

### Responsabilidades dos Componentes

| Componente | Responsabilidade | Implementação Típica |
|------------|------------------|----------------------|
| [nome] | [o que ele possui] | [como é normalmente construído] |
| [nome] | [o que ele possui] | [como é normalmente construído] |
| [nome] | [o que ele possui] | [como é normalmente construído] |

## Estrutura de Projeto Recomendada

```
src/
├── [pasta]/           # [propósito]
│   ├── [subpasta]/    # [propósito]
│   └── [arquivo].ts   # [propósito]
├── [pasta]/           # [propósito]
│   ├── [subpasta]/    # [propósito]
│   └── [arquivo].ts   # [propósito]
├── [pasta]/           # [propósito]
└── [pasta]/           # [propósito]
```

### Justificativa da Estrutura

- **[pasta]/:** [por que organizado dessa forma]
- **[pasta]/:** [por que organizado dessa forma]

## Padrões Arquiteturais

### Padrão 1: [Nome do Padrão]

**O quê:** [descrição]
**Quando usar:** [condições]
**Trade-offs:** [prós e contras]

**Exemplo:**
```typescript
// [Breve exemplo de código mostrando o padrão]
```

### Padrão 2: [Nome do Padrão]

**O quê:** [descrição]
**Quando usar:** [condições]
**Trade-offs:** [prós e contras]

**Exemplo:**
```typescript
// [Breve exemplo de código mostrando o padrão]
```

### Padrão 3: [Nome do Padrão]

**O quê:** [descrição]
**Quando usar:** [condições]
**Trade-offs:** [prós e contras]

## Fluxo de Dados

### Fluxo de Requisição

```
[Ação do Usuário]
    ↓
[Componente] → [Handler] → [Serviço] → [Armazenamento]
    ↓              ↓           ↓            ↓
[Resposta] ← [Transformação] ← [Query] ← [Banco de Dados]
```

### Gerenciamento de Estado

```
[Store de Estado]
    ↓ (subscribe)
[Componentes] ←→ [Ações] → [Reducers/Mutations] → [Store de Estado]
```

### Fluxos de Dados Principais

1. **[Nome do fluxo]:** [descrição de como os dados se movem]
2. **[Nome do fluxo]:** [descrição de como os dados se movem]

## Considerações de Escala

| Escala | Ajustes de Arquitetura |
|--------|------------------------|
| 0-1k usuários | [abordagem — geralmente monolito é suficiente] |
| 1k-100k usuários | [abordagem — o que otimizar primeiro] |
| 100k+ usuários | [abordagem — quando considerar separação] |

### Prioridades de Escala

1. **Primeiro gargalo:** [o que quebra primeiro, como corrigir]
2. **Segundo gargalo:** [o que quebra depois, como corrigir]

## Anti-Padrões

### Anti-Padrão 1: [Nome]

**O que as pessoas fazem:** [o erro]
**Por que está errado:** [o problema que causa]
**Faça isso ao invés:** [a abordagem correta]

### Anti-Padrão 2: [Nome]

**O que as pessoas fazem:** [o erro]
**Por que está errado:** [o problema que causa]
**Faça isso ao invés:** [a abordagem correta]

## Pontos de Integração

### Serviços Externos

| Serviço | Padrão de Integração | Notas |
|---------|----------------------|-------|
| [serviço] | [como conectar] | [armadilhas] |
| [serviço] | [como conectar] | [armadilhas] |

### Fronteiras Internas

| Fronteira | Comunicação | Notas |
|-----------|-------------|-------|
| [módulo A ↔ módulo B] | [API/eventos/direto] | [considerações] |

## Fontes

- [Referências de arquitetura]
- [Documentação oficial]
- [Estudos de caso]

---
*Pesquisa de arquitetura para: [domínio]*
*Pesquisado: [data]*
```

</template>

<guidelines>

**Visão Geral do Sistema:**
- Use diagramas ASCII para clareza (├── └── │ ─ apenas para visualização de estrutura)
- Mostre componentes principais e seus relacionamentos
- Não detalhe demais — isso é conceitual, não implementação

**Estrutura do Projeto:**
- Seja específico sobre organização de pastas
- Explique a justificativa para o agrupamento
- Siga as convenções do stack escolhido

**Padrões:**
- Inclua exemplos de código quando úteis
- Explique trade-offs honestamente
- Note quando padrões são exagero para projetos pequenos

**Considerações de Escala:**
- Seja realista — a maioria dos projetos não precisa escalar para milhões
- Foque em "o que quebra primeiro" não limites teóricos
- Evite recomendações de otimização prematura

**Anti-Padrões:**
- Específicos para este domínio
- Inclua o que fazer ao invés
- Ajuda a prevenir erros comuns durante implementação

</guidelines>
