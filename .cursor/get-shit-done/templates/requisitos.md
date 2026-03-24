# Template de Requisitos

Template para `.planning/REQUIREMENTS.md` — requisitos verificáveis que definem "pronto."

<template>

```markdown
# Requisitos: [Nome do Projeto]

**Definidos:** [data]
**Valor Central:** [do PROJECT.md]

## Requisitos v1

Requisitos para o lançamento inicial. Cada um mapeia para fases do roteiro.

### Autenticação

- [ ] **AUTH-01**: Usuário pode se cadastrar com email e senha
- [ ] **AUTH-02**: Usuário recebe verificação por email após cadastro
- [ ] **AUTH-03**: Usuário pode redefinir senha via link por email
- [ ] **AUTH-04**: Sessão do usuário persiste após atualização do navegador

### [Categoria 2]

- [ ] **[CAT]-01**: [Descrição do requisito]
- [ ] **[CAT]-02**: [Descrição do requisito]
- [ ] **[CAT]-03**: [Descrição do requisito]

### [Categoria 3]

- [ ] **[CAT]-01**: [Descrição do requisito]
- [ ] **[CAT]-02**: [Descrição do requisito]

## Requisitos v2

Adiados para lançamento futuro. Rastreados mas não no roteiro atual.

### [Categoria]

- **[CAT]-01**: [Descrição do requisito]
- **[CAT]-02**: [Descrição do requisito]

## Fora do Escopo

Explicitamente excluídos. Documentados para evitar expansão de escopo.

| Funcionalidade | Motivo |
|----------------|--------|
| [Funcionalidade] | [Motivo da exclusão] |
| [Funcionalidade] | [Motivo da exclusão] |

## Rastreabilidade

Quais fases cobrem quais requisitos. Atualizado durante criação do roteiro.

| Requisito | Fase | Status |
|-----------|------|--------|
| AUTH-01 | Fase 1 | Pendente |
| AUTH-02 | Fase 1 | Pendente |
| AUTH-03 | Fase 1 | Pendente |
| AUTH-04 | Fase 1 | Pendente |
| [REQ-ID] | Fase [N] | Pendente |

**Cobertura:**
- Requisitos v1: [X] total
- Mapeados para fases: [Y]
- Não mapeados: [Z] ⚠️

---
*Requisitos definidos: [data]*
*Última atualização: [data] após [gatilho]*
```

</template>

<guidelines>

**Formato do Requisito:**
- ID: `[CATEGORIA]-[NÚMERO]` (AUTH-01, CONTENT-02, SOCIAL-03)
- Descrição: Centrada no usuário, testável, atômica
- Checkbox: Apenas para requisitos v1 (v2 ainda não são acionáveis)

**Categorias:**
- Derivar das categorias do FEATURES.md da pesquisa
- Manter consistência com convenções do domínio
- Típicas: Autenticação, Conteúdo, Social, Notificações, Moderação, Pagamentos, Admin

**v1 vs v2:**
- v1: Escopo comprometido, estará nas fases do roteiro
- v2: Reconhecidos mas adiados, não no roteiro atual
- Mover v2 → v1 requer atualização do roteiro

**Fora do Escopo:**
- Exclusões explícitas com justificativa
- Evita "por que vocês não incluíram X?" depois
- Anti-funcionalidades da pesquisa ficam aqui com avisos

**Rastreabilidade:**
- Vazio inicialmente, preenchido durante criação do roteiro
- Cada requisito mapeia para exatamente uma fase
- Requisitos não mapeados = lacuna no roteiro

**Valores de Status:**
- Pendente: Não iniciado
- Em Progresso: Fase está ativa
- Concluído: Requisito verificado
- Bloqueado: Aguardando fator externo

</guidelines>

<evolution>

**Após cada fase ser concluída:**
1. Marque requisitos cobertos como Concluídos
2. Atualize status da rastreabilidade
3. Anote qualquer requisito que mudou de escopo

**Após atualizações do roteiro:**
1. Verifique que todos os requisitos v1 ainda estão mapeados
2. Adicione novos requisitos se o escopo expandiu
3. Mova requisitos para v2/fora do escopo se removidos do escopo

**Critérios de conclusão de requisito:**
- Requisito está "Concluído" quando:
  - Funcionalidade está implementada
  - Funcionalidade está verificada (testes passam, verificação manual feita)
  - Funcionalidade está commitada

</evolution>

<example>

```markdown
# Requisitos: CommunityApp

**Definidos:** 2025-01-14
**Valor Central:** Usuários podem compartilhar e discutir conteúdo com pessoas que compartilham seus interesses

## Requisitos v1

### Autenticação

- [ ] **AUTH-01**: Usuário pode se cadastrar com email e senha
- [ ] **AUTH-02**: Usuário recebe verificação por email após cadastro
- [ ] **AUTH-03**: Usuário pode redefinir senha via link por email
- [ ] **AUTH-04**: Sessão do usuário persiste após atualização do navegador

### Perfis

- [ ] **PROF-01**: Usuário pode criar perfil com nome de exibição
- [ ] **PROF-02**: Usuário pode fazer upload de imagem de avatar
- [ ] **PROF-03**: Usuário pode escrever bio (máx 500 caracteres)
- [ ] **PROF-04**: Usuário pode ver perfis de outros usuários

### Conteúdo

- [ ] **CONT-01**: Usuário pode criar post de texto
- [ ] **CONT-02**: Usuário pode fazer upload de imagem com post
- [ ] **CONT-03**: Usuário pode editar próprios posts
- [ ] **CONT-04**: Usuário pode deletar próprios posts
- [ ] **CONT-05**: Usuário pode ver feed de posts

### Social

- [ ] **SOCL-01**: Usuário pode seguir outros usuários
- [ ] **SOCL-02**: Usuário pode deixar de seguir usuários
- [ ] **SOCL-03**: Usuário pode curtir posts
- [ ] **SOCL-04**: Usuário pode comentar em posts
- [ ] **SOCL-05**: Usuário pode ver feed de atividades (posts de usuários seguidos)

## Requisitos v2

### Notificações

- **NOTF-01**: Usuário recebe notificações no app
- **NOTF-02**: Usuário recebe email para novos seguidores
- **NOTF-03**: Usuário recebe email para comentários em próprios posts
- **NOTF-04**: Usuário pode configurar preferências de notificação

### Moderação

- **MODR-01**: Usuário pode denunciar conteúdo
- **MODR-02**: Usuário pode bloquear outros usuários
- **MODR-03**: Admin pode ver conteúdo denunciado
- **MODR-04**: Admin pode remover conteúdo
- **MODR-05**: Admin pode banir usuários

## Fora do Escopo

| Funcionalidade | Motivo |
|----------------|--------|
| Chat em tempo real | Alta complexidade, não é central ao valor de comunidade |
| Posts de vídeo | Custos de armazenamento/banda, adiar para v2+ |
| Login OAuth | Email/senha suficiente para v1 |
| App mobile | Web primeiro, mobile depois |

## Rastreabilidade

| Requisito | Fase | Status |
|-----------|------|--------|
| AUTH-01 | Fase 1 | Pendente |
| AUTH-02 | Fase 1 | Pendente |
| AUTH-03 | Fase 1 | Pendente |
| AUTH-04 | Fase 1 | Pendente |
| PROF-01 | Fase 2 | Pendente |
| PROF-02 | Fase 2 | Pendente |
| PROF-03 | Fase 2 | Pendente |
| PROF-04 | Fase 2 | Pendente |
| CONT-01 | Fase 3 | Pendente |
| CONT-02 | Fase 3 | Pendente |
| CONT-03 | Fase 3 | Pendente |
| CONT-04 | Fase 3 | Pendente |
| CONT-05 | Fase 3 | Pendente |
| SOCL-01 | Fase 4 | Pendente |
| SOCL-02 | Fase 4 | Pendente |
| SOCL-03 | Fase 4 | Pendente |
| SOCL-04 | Fase 4 | Pendente |
| SOCL-05 | Fase 4 | Pendente |

**Cobertura:**
- Requisitos v1: 18 total
- Mapeados para fases: 18
- Não mapeados: 0 ✓

---
*Requisitos definidos: 2025-01-14*
*Última atualização: 2025-01-14 após definição inicial*
```

</example>
