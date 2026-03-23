# Template PROJECT.md

Template para `.planning/PROJECT.md` — o documento vivo de contexto do projeto.

<template>

```markdown
# [Nome do Projeto]

## O Que É Isto

[Descrição atual e precisa — 2-3 frases. O que este produto faz e para quem é?
Use a linguagem e enquadramento do usuário. Atualize sempre que a realidade divergir desta descrição.]

## Valor Central

[A ÚNICA coisa que mais importa. Se tudo mais falhar, isto deve funcionar.
Uma frase que direciona a priorização quando surgem trade-offs.]

## Requisitos

### Validados

<!-- Entregues e confirmados como valiosos. -->

(Nenhum ainda — entregue para validar)

### Ativos

<!-- Escopo atual. Construindo em direção a estes. -->

- [ ] [Requisito 1]
- [ ] [Requisito 2]
- [ ] [Requisito 3]

### Fora do Escopo

<!-- Limites explícitos. Inclui justificativa para evitar re-adição. -->

- [Exclusão 1] — [porquê]
- [Exclusão 2] — [porquê]

## Contexto

[Informações de background que informam a implementação:
- Ambiente técnico ou ecossistema
- Trabalho anterior relevante ou experiência
- Pesquisa de usuário ou temas de feedback
- Problemas conhecidos a resolver]

## Restrições

- **[Tipo]**: [O quê] — [Por quê]
- **[Tipo]**: [O quê] — [Por quê]

Tipos comuns: Stack técnica, Prazo, Orçamento, Dependências, Compatibilidade, Performance, Segurança

## Decisões Chave

<!-- Decisões que restringem trabalho futuro. Adicione ao longo do ciclo de vida do projeto. -->

| Decisão | Justificativa | Resultado |
|---------|---------------|-----------|
| [Escolha] | [Por quê] | [✓ Bom / ⚠️ Revisar / — Pendente] |

---
*Última atualização: [data] após [gatilho]*
```

</template>

<guidelines>

**O Que É Isto:**
- Descrição atual e precisa do produto
- 2-3 frases capturando o que faz e para quem é
- Use as palavras e enquadramento do usuário
- Atualize quando o produto evoluir além desta descrição

**Valor Central:**
- A única coisa mais importante
- Tudo mais pode falhar; isto não pode
- Direciona a priorização quando surgem trade-offs
- Raramente muda; se mudar, é um pivô significativo

**Requisitos — Validados:**
- Requisitos que foram entregues e provaram ser valiosos
- Formato: `- ✓ [Requisito] — [versão/fase]`
- Estes estão travados — alterá-los requer discussão explícita

**Requisitos — Ativos:**
- Escopo atual sendo construído
- São hipóteses até serem entregues e validados
- Mova para Validados quando entregues, Fora do Escopo se invalidados

**Requisitos — Fora do Escopo:**
- Limites explícitos sobre o que não estamos construindo
- Sempre inclua justificativa (evita re-adição posterior)
- Inclui: considerados e rejeitados, adiados para o futuro, explicitamente excluídos

**Contexto:**
- Background que informa decisões de implementação
- Ambiente técnico, trabalho anterior, feedback de usuários
- Problemas conhecidos ou dívida técnica a resolver
- Atualize conforme novo contexto surgir

**Restrições:**
- Limites rígidos nas escolhas de implementação
- Stack técnica, prazo, orçamento, compatibilidade, dependências
- Inclua o "porquê" — restrições sem justificativa são questionadas

**Decisões Chave:**
- Escolhas significativas que afetam trabalho futuro
- Adicione decisões conforme são tomadas ao longo do projeto
- Acompanhe o resultado quando conhecido:
  - ✓ Bom — decisão se provou correta
  - ⚠️ Revisar — decisão pode precisar de reconsideração
  - — Pendente — muito cedo para avaliar

**Última Atualização:**
- Sempre registre quando e por que o documento foi atualizado
- Formato: `após Fase 2` ou `após marco v1.0`
- Aciona revisão sobre se o conteúdo ainda está preciso

</guidelines>

<evolution>

PROJECT.md evolui ao longo do ciclo de vida do projeto.
Estas regras estão embutidas no PROJECT.md gerado (seção ## Evolução)
e implementadas por workflows/transition.md e workflows/complete-milestone.md.

**Após cada transição de fase:**
1. Requisitos invalidados? → Mova para Fora do Escopo com motivo
2. Requisitos validados? → Mova para Validados com referência da fase
3. Novos requisitos surgiram? → Adicione aos Ativos
4. Decisões a registrar? → Adicione às Decisões Chave
5. "O Que É Isto" ainda é preciso? → Atualize se divergiu

**Após cada marco:**
1. Revisão completa de todas as seções
2. Verificação do Valor Central — ainda é a prioridade certa?
3. Auditoria do Fora do Escopo — razões ainda válidas?
4. Atualize Contexto com estado atual (usuários, feedback, métricas)

</evolution>

<brownfield>

Para codebases existentes:

1. **Mapeie o codebase primeiro** via `/gsd-map-codebase`

2. **Infira requisitos Validados** do código existente:
   - O que o codebase realmente faz?
   - Quais padrões estão estabelecidos?
   - O que claramente funciona e é confiável?

3. **Colete requisitos Ativos** do usuário:
   - Apresente o estado atual inferido
   - Pergunte o que querem construir em seguida

4. **Inicialize:**
   - Validados = inferidos do código existente
   - Ativos = objetivos do usuário para este trabalho
   - Fora do Escopo = limites que o usuário especificar
   - Contexto = inclui estado atual do codebase

</brownfield>

<state_reference>

STATE.md referencia PROJECT.md:

```markdown
## Referência do Projeto

Veja: .planning/PROJECT.md (atualizado [data])

**Valor central:** [Resumo do Valor Central]
**Foco atual:** [Nome da fase atual]
```

Isto garante que o Claude leia o contexto atual do PROJECT.md.

</state_reference>
