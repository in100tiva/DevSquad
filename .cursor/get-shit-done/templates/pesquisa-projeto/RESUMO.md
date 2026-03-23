# Template de Resumo de Pesquisa

Template para `.planning/research/SUMMARY.md` — resumo executivo da pesquisa do projeto com implicações para o roadmap.

<template>

```markdown
# Resumo da Pesquisa do Projeto

**Projeto:** [nome do PROJECT.md]
**Domínio:** [tipo de domínio inferido]
**Pesquisado:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Resumo Executivo

[Visão geral de 2-3 parágrafos dos achados da pesquisa]

- Que tipo de produto é este e como especialistas o constroem
- A abordagem recomendada baseada na pesquisa
- Riscos principais e como mitigá-los

## Principais Achados

### Stack Recomendado

[Resumo do STACK.md — 1-2 parágrafos]

**Tecnologias core:**
- [Tecnologia]: [propósito] — [por que recomendada]
- [Tecnologia]: [propósito] — [por que recomendada]
- [Tecnologia]: [propósito] — [por que recomendada]

### Funcionalidades Esperadas

[Resumo do FEATURES.md]

**Obrigatórias (table stakes):**
- [Funcionalidade] — usuários esperam isso
- [Funcionalidade] — usuários esperam isso

**Deveria ter (competitivas):**
- [Funcionalidade] — diferenciador
- [Funcionalidade] — diferenciador

**Adiar (v2+):**
- [Funcionalidade] — não essencial para lançamento

### Abordagem Arquitetural

[Resumo do ARCHITECTURE.md — 1 parágrafo]

**Componentes principais:**
1. [Componente] — [responsabilidade]
2. [Componente] — [responsabilidade]
3. [Componente] — [responsabilidade]

### Armadilhas Críticas

[Top 3-5 do PITFALLS.md]

1. **[Armadilha]** — [como evitar]
2. **[Armadilha]** — [como evitar]
3. **[Armadilha]** — [como evitar]

## Implicações para o Roadmap

Baseado na pesquisa, estrutura de fases sugerida:

### Fase 1: [Nome]
**Justificativa:** [por que isso vem primeiro baseado na pesquisa]
**Entrega:** [o que esta fase produz]
**Endereça:** [funcionalidades do FEATURES.md]
**Evita:** [armadilha do PITFALLS.md]

### Fase 2: [Nome]
**Justificativa:** [por que esta ordem]
**Entrega:** [o que esta fase produz]
**Usa:** [elementos do stack do STACK.md]
**Implementa:** [componente da arquitetura]

### Fase 3: [Nome]
**Justificativa:** [por que esta ordem]
**Entrega:** [o que esta fase produz]

[Continue para fases sugeridas...]

### Justificativa da Ordenação de Fases

- [Por que esta ordem baseada nas dependências descobertas]
- [Por que este agrupamento baseado nos padrões de arquitetura]
- [Como isso evita armadilhas da pesquisa]

### Flags de Pesquisa

Fases que provavelmente precisam de pesquisa mais profunda durante planejamento:
- **Fase [X]:** [razão — ex., "integração complexa, precisa de pesquisa de API"]
- **Fase [Y]:** [razão — ex., "domínio nicho, documentação escassa"]

Fases com padrões estabelecidos (pular research-phase):
- **Fase [X]:** [razão — ex., "bem documentado, padrões estabelecidos"]

## Avaliação de Confiança

| Área | Confiança | Notas |
|------|-----------|-------|
| Stack | [ALTA/MÉDIA/BAIXA] | [razão] |
| Funcionalidades | [ALTA/MÉDIA/BAIXA] | [razão] |
| Arquitetura | [ALTA/MÉDIA/BAIXA] | [razão] |
| Armadilhas | [ALTA/MÉDIA/BAIXA] | [razão] |

**Confiança geral:** [ALTA/MÉDIA/BAIXA]

### Lacunas a Endereçar

[Quaisquer áreas onde a pesquisa foi inconclusiva ou precisa validação durante implementação]

- [Lacuna]: [como tratar durante planejamento/execução]
- [Lacuna]: [como tratar durante planejamento/execução]

## Fontes

### Primárias (ALTA confiança)
- [ID da biblioteca Context7] — [tópicos]
- [URL da documentação oficial] — [o que foi verificado]

### Secundárias (MÉDIA confiança)
- [Fonte] — [achado]

### Terciárias (BAIXA confiança)
- [Fonte] — [achado, precisa validação]

---
*Pesquisa concluída: [data]*
*Pronto para roadmap: sim*
```

</template>

<guidelines>

**Resumo Executivo:**
- Escreva para quem só lerá esta seção
- Inclua a recomendação principal e o risco principal
- Máximo 2-3 parágrafos

**Principais Achados:**
- Resuma, não duplique documentos completos
- Vincule aos docs detalhados (STACK.md, FEATURES.md, etc.)
- Foque no que importa para decisões do roadmap

**Implicações para o Roadmap:**
- Esta é a seção mais importante
- Informa diretamente a criação do roadmap
- Seja explícito sobre sugestões de fases e justificativas
- Inclua flags de pesquisa para cada fase sugerida

**Avaliação de Confiança:**
- Seja honesto sobre incertezas
- Note lacunas que precisam de resolução durante planejamento
- ALTA = verificado com fontes oficiais
- MÉDIA = consenso da comunidade, múltiplas fontes concordam
- BAIXA = fonte única ou inferência

**Integração com criação de roadmap:**
- Este arquivo é carregado como contexto durante criação do roadmap
- Sugestões de fase aqui se tornam ponto de partida para o roadmap
- Flags de pesquisa informam planejamento de fases

</guidelines>
