# Template de Pesquisa de Armadilhas

Template para `.planning/research/PITFALLS.md` — erros comuns a evitar no domínio do projeto.

<template>

```markdown
# Pesquisa de Armadilhas

**Domínio:** [tipo de domínio]
**Pesquisado:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Armadilhas Críticas

### Armadilha 1: [Nome]

**O que dá errado:**
[Descrição do modo de falha]

**Por que acontece:**
[Causa raiz — por que desenvolvedores cometem este erro]

**Como evitar:**
[Estratégia específica de prevenção]

**Sinais de alerta:**
[Como detectar cedo antes de se tornar um problema]

**Fase para tratar:**
[Qual fase do roadmap deve prevenir isso]

---

### Armadilha 2: [Nome]

**O que dá errado:**
[Descrição do modo de falha]

**Por que acontece:**
[Causa raiz — por que desenvolvedores cometem este erro]

**Como evitar:**
[Estratégia específica de prevenção]

**Sinais de alerta:**
[Como detectar cedo antes de se tornar um problema]

**Fase para tratar:**
[Qual fase do roadmap deve prevenir isso]

---

### Armadilha 3: [Nome]

**O que dá errado:**
[Descrição do modo de falha]

**Por que acontece:**
[Causa raiz — por que desenvolvedores cometem este erro]

**Como evitar:**
[Estratégia específica de prevenção]

**Sinais de alerta:**
[Como detectar cedo antes de se tornar um problema]

**Fase para tratar:**
[Qual fase do roadmap deve prevenir isso]

---

[Continue para todas as armadilhas críticas...]

## Padrões de Dívida Técnica

Atalhos que parecem razoáveis mas criam problemas a longo prazo.

| Atalho | Benefício Imediato | Custo a Longo Prazo | Quando Aceitável |
|--------|--------------------|--------------------|------------------|
| [atalho] | [benefício] | [custo] | [condições, ou "nunca"] |
| [atalho] | [benefício] | [custo] | [condições, ou "nunca"] |
| [atalho] | [benefício] | [custo] | [condições, ou "nunca"] |

## Armadilhas de Integração

Erros comuns ao conectar com serviços externos.

| Integração | Erro Comum | Abordagem Correta |
|------------|------------|-------------------|
| [serviço] | [o que as pessoas fazem errado] | [o que fazer ao invés] |
| [serviço] | [o que as pessoas fazem errado] | [o que fazer ao invés] |
| [serviço] | [o que as pessoas fazem errado] | [o que fazer ao invés] |

## Armadilhas de Performance

Padrões que funcionam em pequena escala mas falham conforme o uso cresce.

| Armadilha | Sintomas | Prevenção | Quando Quebra |
|-----------|----------|-----------|---------------|
| [armadilha] | [como você percebe] | [como evitar] | [limite de escala] |
| [armadilha] | [como você percebe] | [como evitar] | [limite de escala] |
| [armadilha] | [como você percebe] | [como evitar] | [limite de escala] |

## Erros de Segurança

Problemas de segurança específicos do domínio além de segurança web geral.

| Erro | Risco | Prevenção |
|------|-------|-----------|
| [erro] | [o que pode acontecer] | [como evitar] |
| [erro] | [o que pode acontecer] | [como evitar] |
| [erro] | [o que pode acontecer] | [como evitar] |

## Armadilhas de UX

Erros comuns de experiência do usuário neste domínio.

| Armadilha | Impacto no Usuário | Melhor Abordagem |
|-----------|--------------------|--------------------|
| [armadilha] | [como usuários sofrem] | [o que fazer ao invés] |
| [armadilha] | [como usuários sofrem] | [o que fazer ao invés] |
| [armadilha] | [como usuários sofrem] | [o que fazer ao invés] |

## Checklist "Parece Pronto Mas Não Está"

Coisas que parecem completas mas estão faltando peças críticas.

- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [checagem]
- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [checagem]
- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [checagem]
- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [checagem]

## Estratégias de Recuperação

Quando armadilhas ocorrem apesar da prevenção, como se recuperar.

| Armadilha | Custo de Recuperação | Passos de Recuperação |
|-----------|----------------------|-----------------------|
| [armadilha] | BAIXO/MÉDIO/ALTO | [o que fazer] |
| [armadilha] | BAIXO/MÉDIO/ALTO | [o que fazer] |
| [armadilha] | BAIXO/MÉDIO/ALTO | [o que fazer] |

## Mapeamento Armadilha-para-Fase

Como as fases do roadmap devem tratar essas armadilhas.

| Armadilha | Fase de Prevenção | Verificação |
|-----------|-------------------|-------------|
| [armadilha] | Fase [X] | [como verificar que a prevenção funcionou] |
| [armadilha] | Fase [X] | [como verificar que a prevenção funcionou] |
| [armadilha] | Fase [X] | [como verificar que a prevenção funcionou] |

## Fontes

- [Post-mortems referenciados]
- [Discussões da comunidade]
- [Documentação oficial de "gotchas"]
- [Experiência pessoal / problemas conhecidos]

---
*Pesquisa de armadilhas para: [domínio]*
*Pesquisado: [data]*
```

</template>

<guidelines>

**Armadilhas Críticas:**
- Foque em problemas específicos do domínio, não erros genéricos
- Inclua sinais de alerta — detecção precoce previne desastres
- Vincule a fases específicas — torna armadilhas acionáveis

**Dívida Técnica:**
- Seja realista — alguns atalhos são aceitáveis
- Note quando atalhos são "nunca aceitáveis" vs. "apenas no MVP"
- Inclua o custo a longo prazo para informar decisões de trade-off

**Armadilhas de Performance:**
- Inclua limites de escala ("quebra com 10k usuários")
- Foque no que é relevante para a escala esperada deste projeto
- Não engenharia em excesso para escala hipotética

**Erros de Segurança:**
- Além do básico OWASP — problemas específicos do domínio
- Exemplo: Plataformas de comunidade têm preocupações de segurança diferentes de e-commerce
- Inclua nível de risco para priorizar

**"Parece Pronto Mas Não Está":**
- Formato de checklist para verificação durante execução
- Comum em demos vs. produção
- Previne problemas "funciona na minha máquina"

**Mapeamento Armadilha-para-Fase:**
- Crítico para criação do roadmap
- Cada armadilha deve mapear para uma fase que a previne
- Informa ordenação de fases e critérios de sucesso

</guidelines>
