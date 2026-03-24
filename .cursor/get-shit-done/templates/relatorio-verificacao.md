# Template de Relatório de Verificação

Template para `.planning/phases/XX-nome/{num_fase}-VERIFICATION.md` — resultados de verificação de objetivo da fase.

---

## Template do Arquivo

```markdown
---
phase: XX-nome
verified: YYYY-MM-DDTHH:MM:SSZ
status: passed | gaps_found | human_needed
score: N/M must-haves verificados
---

# Fase {X}: {Nome} Relatório de Verificação

**Objetivo da Fase:** {objetivo do ROADMAP.md}
**Verificado:** {timestamp}
**Status:** {passed | gaps_found | human_needed}

## Alcance do Objetivo

### Verdades Observáveis

| # | Verdade | Status | Evidência |
|---|---------|--------|-----------|
| 1 | {verdade do must_haves} | ✓ VERIFICADO | {o que confirmou} |
| 2 | {verdade do must_haves} | ✗ FALHOU | {o que está errado} |
| 3 | {verdade do must_haves} | ? INCERTO | {por que não pode verificar} |

**Pontuação:** {N}/{M} verdades verificadas

### Artefatos Obrigatórios

| Artefato | Esperado | Status | Detalhes |
|----------|----------|--------|----------|
| `src/components/Chat.tsx` | Componente de lista de mensagens | ✓ EXISTE + SUBSTANTIVO | Exporta ChatList, renderiza Message[], sem stubs |
| `src/app/api/chat/route.ts` | CRUD de Mensagens | ✗ STUB | Arquivo existe mas POST retorna placeholder |
| `prisma/schema.prisma` | Modelo de Message | ✓ EXISTE + SUBSTANTIVO | Modelo definido com todos os campos |

**Artefatos:** {N}/{M} verificados

### Verificação de Links Chave

| De | Para | Via | Status | Detalhes |
|----|------|----|--------|----------|
| Chat.tsx | /api/chat | fetch no useEffect | ✓ CONECTADO | Linha 23: `fetch('/api/chat')` com tratamento de resposta |
| ChatInput | /api/chat POST | handler onSubmit | ✗ NÃO CONECTADO | onSubmit apenas chama console.log |
| /api/chat POST | banco de dados | prisma.message.create | ✗ NÃO CONECTADO | Retorna resposta hardcoded, sem chamada ao BD |

**Conexões:** {N}/{M} verificadas

## Cobertura de Requisitos

| Requisito | Status | Problema Bloqueante |
|-----------|--------|---------------------|
| {REQ-01}: {descrição} | ✓ SATISFEITO | - |
| {REQ-02}: {descrição} | ✗ BLOQUEADO | Rota de API é stub |
| {REQ-03}: {descrição} | ? PRECISA HUMANO | Não é possível verificar WebSocket programaticamente |

**Cobertura:** {N}/{M} requisitos satisfeitos

## Anti-Padrões Encontrados

| Arquivo | Linha | Padrão | Severidade | Impacto |
|---------|-------|--------|------------|---------|
| src/app/api/chat/route.ts | 12 | `// TODO: implement` | ⚠️ Aviso | Indica incompleto |
| src/components/Chat.tsx | 45 | `return <div>Placeholder</div>` | 🛑 Bloqueador | Não renderiza conteúdo |
| src/hooks/useChat.ts | - | Arquivo ausente | 🛑 Bloqueador | Hook esperado não existe |

**Anti-padrões:** {N} encontrados ({bloqueadores} bloqueadores, {avisos} avisos)

## Verificação Humana Necessária

{Se nenhuma verificação humana necessária:}
Nenhuma — todos os itens verificáveis foram verificados programaticamente.

{Se verificação humana necessária:}

### 1. {Nome do Teste}
**Teste:** {O que fazer}
**Esperado:** {O que deve acontecer}
**Por que humano:** {Por que não pode verificar programaticamente}

### 2. {Nome do Teste}
**Teste:** {O que fazer}
**Esperado:** {O que deve acontecer}
**Por que humano:** {Por que não pode verificar programaticamente}

## Resumo de Lacunas

{Se sem lacunas:}
**Nenhuma lacuna encontrada.** Objetivo da fase alcançado. Pronto para prosseguir.

{Se lacunas encontradas:}

### Lacunas Críticas (Bloqueiam Progresso)

1. **{Nome da lacuna}**
   - Ausente: {o que está faltando}
   - Impacto: {por que isto bloqueia o objetivo}
   - Correção: {o que precisa acontecer}

2. **{Nome da lacuna}**
   - Ausente: {o que está faltando}
   - Impacto: {por que isto bloqueia o objetivo}
   - Correção: {o que precisa acontecer}

### Lacunas Não-Críticas (Podem ser Adiadas)

1. **{Nome da lacuna}**
   - Problema: {o que está errado}
   - Impacto: {impacto limitado porque...}
   - Recomendação: {corrigir agora ou adiar}

## Planos de Correção Recomendados

{Se lacunas encontradas, gere recomendações de plano de correção:}

### {fase}-{próximo}-PLAN.md: {Nome da Correção}

**Objetivo:** {O que isto corrige}

**Tarefas:**
1. {Tarefa para corrigir lacuna 1}
2. {Tarefa para corrigir lacuna 2}
3. {Tarefa de verificação}

**Escopo estimado:** {Pequeno / Médio}

---

### {fase}-{próximo+1}-PLAN.md: {Nome da Correção}

**Objetivo:** {O que isto corrige}

**Tarefas:**
1. {Tarefa}
2. {Tarefa}

**Escopo estimado:** {Pequeno / Médio}

---

## Metadados de Verificação

**Abordagem de verificação:** Goal-backward (derivada do objetivo da fase)
**Fonte dos must-haves:** {frontmatter do PLAN.md | derivado do objetivo do ROADMAP.md}
**Verificações automatizadas:** {N} passaram, {M} falharam
**Verificações humanas necessárias:** {N}
**Tempo total de verificação:** {duração}

---
*Verificado: {timestamp}*
*Verificador: Claude (subagente)*
```

---

## Diretrizes

**Valores de status:**
- `passed` — Todos os must-haves verificados, sem bloqueadores
- `gaps_found` — Uma ou mais lacunas críticas encontradas
- `human_needed` — Verificações automatizadas passam mas verificação humana é necessária

**Tipos de evidência:**
- Para EXISTS: "Arquivo no caminho, exporta X"
- Para SUBSTANTIVE: "N linhas, tem padrões X, Y, Z"
- Para WIRED: "Linha N: código que conecta A a B"
- Para FAILED: "Ausente porque X" ou "Stub porque Y"

**Níveis de severidade:**
- 🛑 Bloqueador: Impede alcance do objetivo, deve corrigir
- ⚠️ Aviso: Indica incompleto mas não bloqueia
- ℹ️ Info: Notável mas não problemático

**Geração de plano de correção:**
- Apenas gere se gaps_found
- Agrupe correções relacionadas em planos únicos
- Mantenha 2-3 tarefas por plano
- Inclua tarefa de verificação em cada plano

---

## Exemplo

```markdown
---
phase: 03-chat
verified: 2025-01-15T14:30:00Z
status: gaps_found
score: 2/5 must-haves verificados
---

# Fase 3: Interface de Chat Relatório de Verificação

**Objetivo da Fase:** Interface de chat funcional onde usuários podem enviar e receber mensagens
**Verificado:** 2025-01-15T14:30:00Z
**Status:** gaps_found

## Alcance do Objetivo

### Verdades Observáveis

| # | Verdade | Status | Evidência |
|---|---------|--------|-----------|
| 1 | Usuário pode ver mensagens existentes | ✗ FALHOU | Componente renderiza placeholder, não dados de mensagem |
| 2 | Usuário pode digitar uma mensagem | ✓ VERIFICADO | Campo de input existe com handler onChange |
| 3 | Usuário pode enviar uma mensagem | ✗ FALHOU | Handler onSubmit é apenas console.log |
| 4 | Mensagem enviada aparece na lista | ✗ FALHOU | Sem atualização de estado após envio |
| 5 | Mensagens persistem após atualização | ? INCERTO | Não é possível verificar - envio não funciona |

**Pontuação:** 1/5 verdades verificadas

### Artefatos Obrigatórios

| Artefato | Esperado | Status | Detalhes |
|----------|----------|--------|----------|
| `src/components/Chat.tsx` | Componente de lista de mensagens | ✗ STUB | Retorna `<div>Chat ficará aqui</div>` |
| `src/components/ChatInput.tsx` | Input de mensagem | ✓ EXISTE + SUBSTANTIVO | Formulário com input, botão de envio, handlers |
| `src/app/api/chat/route.ts` | CRUD de Mensagens | ✗ STUB | GET retorna [], POST retorna { ok: true } |
| `prisma/schema.prisma` | Modelo de Message | ✓ EXISTE + SUBSTANTIVO | Modelo Message com id, content, userId, createdAt |

**Artefatos:** 2/4 verificados

### Verificação de Links Chave

| De | Para | Via | Status | Detalhes |
|----|------|----|--------|----------|
| Chat.tsx | /api/chat GET | fetch | ✗ NÃO CONECTADO | Sem chamada fetch no componente |
| ChatInput | /api/chat POST | onSubmit | ✗ NÃO CONECTADO | Handler apenas loga, não faz fetch |
| /api/chat GET | banco de dados | prisma.message.findMany | ✗ NÃO CONECTADO | Retorna [] hardcoded |
| /api/chat POST | banco de dados | prisma.message.create | ✗ NÃO CONECTADO | Retorna { ok: true }, sem chamada ao BD |

**Conexões:** 0/4 verificadas

## Cobertura de Requisitos

| Requisito | Status | Problema Bloqueante |
|-----------|--------|---------------------|
| CHAT-01: Usuário pode enviar mensagem | ✗ BLOQUEADO | API POST é stub |
| CHAT-02: Usuário pode ver mensagens | ✗ BLOQUEADO | Componente é placeholder |
| CHAT-03: Mensagens persistem | ✗ BLOQUEADO | Sem integração com banco de dados |

**Cobertura:** 0/3 requisitos satisfeitos

## Anti-Padrões Encontrados

| Arquivo | Linha | Padrão | Severidade | Impacto |
|---------|-------|--------|------------|---------|
| src/components/Chat.tsx | 8 | `<div>Chat ficará aqui</div>` | 🛑 Bloqueador | Sem conteúdo real |
| src/app/api/chat/route.ts | 5 | `return Response.json([])` | 🛑 Bloqueador | Vazio hardcoded |
| src/app/api/chat/route.ts | 12 | `// TODO: salvar no banco` | ⚠️ Aviso | Incompleto |

**Anti-padrões:** 3 encontrados (2 bloqueadores, 1 aviso)

## Verificação Humana Necessária

Nenhuma necessária até que lacunas automatizadas sejam corrigidas.

## Resumo de Lacunas

### Lacunas Críticas (Bloqueiam Progresso)

1. **Componente de chat é placeholder**
   - Ausente: Renderização real da lista de mensagens
   - Impacto: Usuários veem "Chat ficará aqui" ao invés de mensagens
   - Correção: Implementar Chat.tsx para buscar e renderizar mensagens

2. **Rotas de API são stubs**
   - Ausente: Integração com banco de dados em GET e POST
   - Impacto: Sem persistência de dados, sem funcionalidade real
   - Correção: Conectar chamadas prisma nos handlers de rota

3. **Sem conexão entre frontend e backend**
   - Ausente: Chamadas fetch nos componentes
   - Impacto: Mesmo se a API funcionasse, UI não a chamaria
   - Correção: Adicionar fetch useEffect no Chat, fetch onSubmit no ChatInput

## Planos de Correção Recomendados

### 03-04-PLAN.md: Implementar API de Chat

**Objetivo:** Conectar rotas de API ao banco de dados

**Tarefas:**
1. Implementar GET /api/chat com prisma.message.findMany
2. Implementar POST /api/chat com prisma.message.create
3. Verificar: API retorna dados reais, POST cria registros

**Escopo estimado:** Pequeno

---

### 03-05-PLAN.md: Implementar UI de Chat

**Objetivo:** Conectar componente Chat à API

**Tarefas:**
1. Implementar Chat.tsx com fetch useEffect e renderização de mensagens
2. Conectar ChatInput onSubmit ao POST /api/chat
3. Verificar: Mensagens são exibidas, novas mensagens aparecem após envio

**Escopo estimado:** Pequeno

---

## Metadados de Verificação

**Abordagem de verificação:** Goal-backward (derivada do objetivo da fase)
**Fonte dos must-haves:** frontmatter do 03-01-PLAN.md
**Verificações automatizadas:** 2 passaram, 8 falharam
**Verificações humanas necessárias:** 0 (bloqueadas por falhas automatizadas)
**Tempo total de verificação:** 2 min

---
*Verificado: 2025-01-15T14:30:00Z*
*Verificador: Claude (subagente)*
```
