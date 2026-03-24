# Formato de Continuação

Formato padrão para apresentar próximos passos após completar um comando ou fluxo de trabalho.

## Estrutura Central

```
---

## ▶ Próximo

**{identificador}: {nome}** — {descrição de uma linha}

`{comando para copiar e colar}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `{opção alternativa 1}` — descrição
- `{opção alternativa 2}` — descrição

---
```

## Regras de Formato

1. **Sempre mostre o que é** — nome + descrição, nunca apenas um caminho de comando
2. **Extraia contexto da fonte** — ROADMAP.md para fases, PLAN.md `<objective>` para planos
3. **Comando em código inline** — crases, fácil de copiar e colar, renderiza como link clicável
4. **Explicação do `/clear`** — sempre inclua, mantém conciso mas explica o porquê
5. **"Também disponível" não "Outras opções"** — soa mais como aplicativo
6. **Separadores visuais** — `---` acima e abaixo para destacar

## Variantes

### Executar Próximo Plano

```
---

## ▶ Próximo

**02-03: Rotação de Refresh Token** — Adicionar /api/auth/refresh com expiração deslizante

`/gsd-execute-phase 2`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- Revisar plano antes de executar
- `/gsd-list-phase-assumptions 2` — verificar suposições

---
```

### Executar Último Plano da Fase

Adicione nota de que este é o último plano e o que vem depois:

```
---

## ▶ Próximo

**02-03: Rotação de Refresh Token** — Adicionar /api/auth/refresh com expiração deslizante
<sub>Último plano na Fase 2</sub>

`/gsd-execute-phase 2`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Após isso completar:**
- Fase 2 → transição para Fase 3
- Próximo: **Fase 3: Funcionalidades Centrais** — Dashboard e configurações do usuário

---
```

### Planejar uma Fase

```
---

## ▶ Próximo

**Fase 2: Autenticação** — Fluxo de login JWT com refresh tokens

`/gsd-plan-phase 2`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-discuss-phase 2` — coletar contexto primeiro
- `/gsd-research-phase 2` — investigar incógnitas
- Revisar roadmap

---
```

### Fase Completa, Pronto para Próxima

Mostre status de conclusão antes da próxima ação:

```
---

## ✓ Fase 2 Completa

3/3 planos executados

## ▶ Próximo

**Fase 3: Funcionalidades Centrais** — Dashboard do usuário, configurações e exportação de dados

`/gsd-plan-phase 3`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-discuss-phase 3` — coletar contexto primeiro
- `/gsd-research-phase 3` — investigar incógnitas
- Revisar o que a Fase 2 construiu

---
```

### Múltiplas Opções Equivalentes

Quando não há ação primária clara:

```
---

## ▶ Próximo

**Fase 3: Funcionalidades Centrais** — Dashboard do usuário, configurações e exportação de dados

**Para planejar diretamente:** `/gsd-plan-phase 3`

**Para discutir contexto primeiro:** `/gsd-discuss-phase 3`

**Para pesquisar incógnitas:** `/gsd-research-phase 3`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---
```

### Milestone Completo

```
---

## 🎉 Milestone v1.0 Completo

Todas as 4 fases entregues

## ▶ Próximo

**Iniciar v1.1** — questionamento → pesquisa → requisitos → roadmap

`/gsd-new-milestone`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---
```

## Extraindo Contexto

### Para fases (do ROADMAP.md):

```markdown
### Fase 2: Autenticação
**Objetivo**: Fluxo de login JWT com refresh tokens
```

Extrair: `**Fase 2: Autenticação** — Fluxo de login JWT com refresh tokens`

### Para planos (do ROADMAP.md):

```markdown
Planos:
- [ ] 02-03: Adicionar rotação de refresh token
```

Ou do PLAN.md `<objective>`:

```xml
<objective>
Adicionar rotação de refresh token com janela de expiração deslizante.

Propósito: Estender tempo de vida da sessão sem comprometer segurança.
</objective>
```

Extrair: `**02-03: Rotação de Refresh Token** — Adicionar /api/auth/refresh com expiração deslizante`

## Anti-Padrões

### Não faça: Apenas comando (sem contexto)

```
## Para Continuar

Execute `/clear`, depois cole:
/gsd-execute-phase 2
```

Usuário não tem ideia do que 02-03 é sobre.

### Não faça: Sem explicação do /clear

```
`/gsd-plan-phase 3`

Execute /clear primeiro.
```

Não explica o porquê. Usuário pode pular.

### Não faça: Linguagem "Outras opções"

```
Outras opções:
- Revisar roadmap
```

Soa como algo secundário. Use "Também disponível:" no lugar.

### Não faça: Blocos de código cercados para comandos

```
```
/gsd-plan-phase 3
```
```

Blocos cercados dentro de templates criam ambiguidade de aninhamento. Use crases inline no lugar.
