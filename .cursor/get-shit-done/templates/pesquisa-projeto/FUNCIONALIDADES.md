# Template de Pesquisa de Funcionalidades

Template para `.planning/research/FEATURES.md` — panorama de funcionalidades para o domínio do projeto.

<template>

```markdown
# Pesquisa de Funcionalidades

**Domínio:** [tipo de domínio]
**Pesquisado:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Panorama de Funcionalidades

### Table Stakes (Usuários Esperam Essas)

Funcionalidades que os usuários assumem existir. Sem elas = produto parece incompleto.

| Funcionalidade | Por Que Esperada | Complexidade | Notas |
|----------------|------------------|--------------|-------|
| [funcionalidade] | [expectativa do usuário] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [expectativa do usuário] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [expectativa do usuário] | BAIXA/MÉDIA/ALTA | [notas de implementação] |

### Diferenciadores (Vantagem Competitiva)

Funcionalidades que diferenciam o produto. Não obrigatórias, mas valiosas.

| Funcionalidade | Proposta de Valor | Complexidade | Notas |
|----------------|-------------------|--------------|-------|
| [funcionalidade] | [por que importa] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [por que importa] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [por que importa] | BAIXA/MÉDIA/ALTA | [notas de implementação] |

### Anti-Funcionalidades (Comumente Pedidas, Frequentemente Problemáticas)

Funcionalidades que parecem boas mas criam problemas.

| Funcionalidade | Por Que Pedida | Por Que Problemática | Alternativa |
|----------------|----------------|----------------------|-------------|
| [funcionalidade] | [apelo superficial] | [problemas reais] | [melhor abordagem] |
| [funcionalidade] | [apelo superficial] | [problemas reais] | [melhor abordagem] |

## Dependências de Funcionalidades

```
[Funcionalidade A]
    └──requer──> [Funcionalidade B]
                       └──requer──> [Funcionalidade C]

[Funcionalidade D] ──aprimora──> [Funcionalidade A]

[Funcionalidade E] ──conflita──> [Funcionalidade F]
```

### Notas de Dependência

- **[Funcionalidade A] requer [Funcionalidade B]:** [por que a dependência existe]
- **[Funcionalidade D] aprimora [Funcionalidade A]:** [como funcionam juntas]
- **[Funcionalidade E] conflita com [Funcionalidade F]:** [por que são incompatíveis]

## Definição de MVP

### Lançar Com (v1)

Produto mínimo viável — o que é necessário para validar o conceito.

- [ ] [Funcionalidade] — [por que essencial]
- [ ] [Funcionalidade] — [por que essencial]
- [ ] [Funcionalidade] — [por que essencial]

### Adicionar Após Validação (v1.x)

Funcionalidades para adicionar quando o core estiver funcionando.

- [ ] [Funcionalidade] — [gatilho para adicionar]
- [ ] [Funcionalidade] — [gatilho para adicionar]

### Consideração Futura (v2+)

Funcionalidades para adiar até product-market fit ser estabelecido.

- [ ] [Funcionalidade] — [por que adiar]
- [ ] [Funcionalidade] — [por que adiar]

## Matriz de Priorização de Funcionalidades

| Funcionalidade | Valor pro Usuário | Custo de Implementação | Prioridade |
|----------------|-------------------|------------------------|------------|
| [funcionalidade] | ALTO/MÉDIO/BAIXO | ALTO/MÉDIO/BAIXO | P1/P2/P3 |
| [funcionalidade] | ALTO/MÉDIO/BAIXO | ALTO/MÉDIO/BAIXO | P1/P2/P3 |
| [funcionalidade] | ALTO/MÉDIO/BAIXO | ALTO/MÉDIO/BAIXO | P1/P2/P3 |

**Legenda de prioridade:**
- P1: Obrigatório para lançamento
- P2: Deveria ter, adicionar quando possível
- P3: Bom ter, consideração futura

## Análise de Funcionalidades dos Concorrentes

| Funcionalidade | Concorrente A | Concorrente B | Nossa Abordagem |
|----------------|---------------|---------------|-----------------|
| [funcionalidade] | [como fazem] | [como fazem] | [nosso plano] |
| [funcionalidade] | [como fazem] | [como fazem] | [nosso plano] |

## Fontes

- [Produtos concorrentes analisados]
- [Pesquisa de usuário ou fontes de feedback]
- [Padrões do setor referenciados]

---
*Pesquisa de funcionalidades para: [domínio]*
*Pesquisado: [data]*
```

</template>

<guidelines>

**Table Stakes:**
- Essas são inegociáveis para o lançamento
- Usuários não dão crédito por tê-las, mas penalizam por não ter
- Exemplo: Uma plataforma de comunidade sem perfis de usuário está quebrada

**Diferenciadores:**
- É onde você compete
- Devem se alinhar com o Valor Core do PROJECT.md
- Não tente se diferenciar em tudo

**Anti-Funcionalidades:**
- Previna scope creep documentando o que parece bom mas não é
- Inclua a abordagem alternativa
- Exemplo: "Tempo real em tudo" frequentemente cria complexidade sem valor

**Dependências de Funcionalidades:**
- Crítico para ordenação de fases do roadmap
- Se A requer B, B deve estar em uma fase anterior
- Conflitos informam o que NÃO combinar na mesma fase

**Definição de MVP:**
- Seja implacável sobre o que é verdadeiramente mínimo
- "Bom ter" não é MVP
- Lance com menos, valide, depois expanda

</guidelines>
