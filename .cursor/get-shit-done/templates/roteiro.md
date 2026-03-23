# Template do Roteiro

Template para `.planning/ROADMAP.md`.

## Roteiro Inicial (v1.0 Greenfield)

```markdown
# Roteiro: [Nome do Projeto]

## Visão Geral

[Um parágrafo descrevendo a jornada do início ao fim]

## Fases

**Numeração das Fases:**
- Fases inteiras (1, 2, 3): Trabalho planejado do marco
- Fases decimais (2.1, 2.2): Inserções urgentes (marcadas com INSERIDA)

Fases decimais aparecem entre seus inteiros adjacentes em ordem numérica.

- [ ] **Fase 1: [Nome]** - [Descrição de uma linha]
- [ ] **Fase 2: [Nome]** - [Descrição de uma linha]
- [ ] **Fase 3: [Nome]** - [Descrição de uma linha]
- [ ] **Fase 4: [Nome]** - [Descrição de uma linha]

## Detalhes das Fases

### Fase 1: [Nome]
**Objetivo**: [O que esta fase entrega]
**Depende de**: Nada (primeira fase)
**Requisitos**: [REQ-01, REQ-02, REQ-03]  <!-- colchetes opcionais, o parser aceita ambos os formatos -->
**Critérios de Sucesso** (o que deve ser VERDADEIRO):
  1. [Comportamento observável da perspectiva do usuário]
  2. [Comportamento observável da perspectiva do usuário]
  3. [Comportamento observável da perspectiva do usuário]
**Planos**: [Número de planos, ex., "3 planos" ou "A definir"]

Planos:
- [ ] 01-01: [Breve descrição do primeiro plano]
- [ ] 01-02: [Breve descrição do segundo plano]
- [ ] 01-03: [Breve descrição do terceiro plano]

### Fase 2: [Nome]
**Objetivo**: [O que esta fase entrega]
**Depende de**: Fase 1
**Requisitos**: [REQ-04, REQ-05]
**Critérios de Sucesso** (o que deve ser VERDADEIRO):
  1. [Comportamento observável da perspectiva do usuário]
  2. [Comportamento observável da perspectiva do usuário]
**Planos**: [Número de planos]

Planos:
- [ ] 02-01: [Breve descrição]
- [ ] 02-02: [Breve descrição]

### Fase 2.1: Correção Crítica (INSERIDA)
**Objetivo**: [Trabalho urgente inserido entre fases]
**Depende de**: Fase 2
**Critérios de Sucesso** (o que deve ser VERDADEIRO):
  1. [O que a correção alcança]
**Planos**: 1 plano

Planos:
- [ ] 02.1-01: [Descrição]

### Fase 3: [Nome]
**Objetivo**: [O que esta fase entrega]
**Depende de**: Fase 2
**Requisitos**: [REQ-06, REQ-07, REQ-08]
**Critérios de Sucesso** (o que deve ser VERDADEIRO):
  1. [Comportamento observável da perspectiva do usuário]
  2. [Comportamento observável da perspectiva do usuário]
  3. [Comportamento observável da perspectiva do usuário]
**Planos**: [Número de planos]

Planos:
- [ ] 03-01: [Breve descrição]
- [ ] 03-02: [Breve descrição]

### Fase 4: [Nome]
**Objetivo**: [O que esta fase entrega]
**Depende de**: Fase 3
**Requisitos**: [REQ-09, REQ-10]
**Critérios de Sucesso** (o que deve ser VERDADEIRO):
  1. [Comportamento observável da perspectiva do usuário]
  2. [Comportamento observável da perspectiva do usuário]
**Planos**: [Número de planos]

Planos:
- [ ] 04-01: [Breve descrição]

## Progresso

**Ordem de Execução:**
Fases executam em ordem numérica: 2 → 2.1 → 2.2 → 3 → 3.1 → 4

| Fase | Planos Concluídos | Status | Concluída |
|------|-------------------|--------|-----------|
| 1. [Nome] | 0/3 | Não iniciada | - |
| 2. [Nome] | 0/2 | Não iniciada | - |
| 3. [Nome] | 0/2 | Não iniciada | - |
| 4. [Nome] | 0/1 | Não iniciada | - |
```

<guidelines>
**Planejamento inicial (v1.0):**
- Quantidade de fases depende da configuração de granularidade (grossa: 3-5, padrão: 5-8, fina: 8-12)
- Cada fase entrega algo coerente
- Fases podem ter 1+ planos (divida se >3 tarefas ou múltiplos subsistemas)
- Planos usam nomenclatura: {fase}-{plano}-PLAN.md (ex., 01-02-PLAN.md)
- Sem estimativas de tempo (isto não é PM corporativo)
- Tabela de progresso atualizada pelo workflow de execução
- Contagem de planos pode ser "A definir" inicialmente, refinada durante o planejamento

**Critérios de sucesso:**
- 2-5 comportamentos observáveis por fase (da perspectiva do usuário)
- Verificados cruzadamente com requisitos durante criação do roteiro
- Fluem para `must_haves` no plan-phase
- Verificados pelo verify-phase após execução
- Formato: "Usuário pode [ação]" ou "[Coisa] funciona/existe"

**Após marcos serem entregues:**
- Recolha marcos concluídos em tags `<details>`
- Adicione novas seções de marco para trabalho futuro
- Mantenha numeração contínua de fases (nunca reinicie em 01)
</guidelines>

<status_values>
- `Não iniciada` - Ainda não começou
- `Em progresso` - Trabalhando atualmente
- `Concluída` - Feita (adicione data de conclusão)
- `Adiada` - Empurrada para depois (com motivo)
</status_values>

## Roteiro Agrupado por Marco (Após v1.0 Ser Entregue)

Após completar o primeiro marco, reorganize com agrupamentos por marco:

```markdown
# Roteiro: [Nome do Projeto]

## Marcos

- ✅ **v1.0 MVP** - Fases 1-4 (entregue AAAA-MM-DD)
- 🚧 **v1.1 [Nome]** - Fases 5-6 (em progresso)
- 📋 **v2.0 [Nome]** - Fases 7-10 (planejado)

## Fases

<details>
<summary>✅ v1.0 MVP (Fases 1-4) - ENTREGUE AAAA-MM-DD</summary>

### Fase 1: [Nome]
**Objetivo**: [O que esta fase entrega]
**Planos**: 3 planos

Planos:
- [x] 01-01: [Breve descrição]
- [x] 01-02: [Breve descrição]
- [x] 01-03: [Breve descrição]

[... fases restantes da v1.0 ...]

</details>

### 🚧 v1.1 [Nome] (Em Progresso)

**Objetivo do Marco:** [O que v1.1 entrega]

#### Fase 5: [Nome]
**Objetivo**: [O que esta fase entrega]
**Depende de**: Fase 4
**Planos**: 2 planos

Planos:
- [ ] 05-01: [Breve descrição]
- [ ] 05-02: [Breve descrição]

[... fases restantes da v1.1 ...]

### 📋 v2.0 [Nome] (Planejado)

**Objetivo do Marco:** [O que v2.0 entrega]

[... fases da v2.0 ...]

## Progresso

| Fase | Marco | Planos Concluídos | Status | Concluída |
|------|-------|-------------------|--------|-----------|
| 1. Fundação | v1.0 | 3/3 | Concluída | AAAA-MM-DD |
| 2. Funcionalidades | v1.0 | 2/2 | Concluída | AAAA-MM-DD |
| 5. Segurança | v1.1 | 0/2 | Não iniciada | - |
```

**Notas:**
- Emoji do marco: ✅ entregue, 🚧 em progresso, 📋 planejado
- Marcos concluídos recolhidos em `<details>` para legibilidade
- Marcos atuais/futuros expandidos
- Numeração contínua de fases (01-99)
- Tabela de progresso inclui coluna de marco
