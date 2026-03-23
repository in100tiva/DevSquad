# Template de Resumo

Template para `.planning/phases/XX-nome/{fase}-{plano}-SUMMARY.md` - documentação de conclusão de fase.

---

## Template do Arquivo

```markdown
---
phase: XX-nome
plan: YY
subsystem: [categoria principal: auth, payments, ui, api, database, infra, testing, etc.]
tags: [tech pesquisável: jwt, stripe, react, postgres, prisma]

# Grafo de dependências
requires:
  - phase: [fase anterior da qual depende]
    provides: [o que aquela fase construiu que esta usa]
provides:
  - [lista de pontos do que esta fase construiu/entregou]
affects: [lista de nomes de fases ou palavras-chave que precisarão deste contexto]

# Rastreamento tech
tech-stack:
  added: [bibliotecas/ferramentas adicionadas nesta fase]
  patterns: [padrões arquiteturais/código estabelecidos]

key-files:
  created: [arquivos importantes criados]
  modified: [arquivos importantes modificados]

key-decisions:
  - "Decisão 1"
  - "Decisão 2"

patterns-established:
  - "Padrão 1: descrição"
  - "Padrão 2: descrição"

requirements-completed: []  # OBRIGATÓRIO — Copie TODOS os IDs de requisitos do campo `requirements` do frontmatter deste plano.

# Métricas
duration: Xmin
completed: AAAA-MM-DD
---

# Fase [X]: [Nome] Resumo

**[Resumo substantivo de uma linha descrevendo o resultado - NÃO "fase concluída" ou "implementação finalizada"]**

## Performance

- **Duração:** [tempo] (ex., 23 min, 1h 15m)
- **Início:** [timestamp ISO]
- **Conclusão:** [timestamp ISO]
- **Tarefas:** [contagem concluída]
- **Arquivos modificados:** [contagem]

## Conquistas
- [Resultado mais importante]
- [Segunda conquista chave]
- [Terceira se aplicável]

## Commits por Tarefa

Cada tarefa foi commitada atomicamente:

1. **Tarefa 1: [nome da tarefa]** - `abc123f` (feat/fix/test/refactor)
2. **Tarefa 2: [nome da tarefa]** - `def456g` (feat/fix/test/refactor)
3. **Tarefa 3: [nome da tarefa]** - `hij789k` (feat/fix/test/refactor)

**Metadados do plano:** `lmn012o` (docs: plano concluído)

_Nota: Tarefas TDD podem ter múltiplos commits (test → feat → refactor)_

## Arquivos Criados/Modificados
- `caminho/para/arquivo.ts` - O que faz
- `caminho/para/outro.ts` - O que faz

## Decisões Tomadas
[Decisões chave com breve justificativa, ou "Nenhuma - seguiu o plano conforme especificado"]

## Desvios do Plano

[Se sem desvios: "Nenhum - plano executado exatamente como escrito"]

[Se desvios ocorreram:]

### Correções Automáticas

**1. [Regra X - Categoria] Breve descrição**
- **Encontrado durante:** Tarefa [N] ([nome da tarefa])
- **Problema:** [O que estava errado]
- **Correção:** [O que foi feito]
- **Arquivos modificados:** [caminhos dos arquivos]
- **Verificação:** [Como foi verificado]
- **Commitado em:** [hash] (parte do commit da tarefa)

[... repetir para cada correção automática ...]

---

**Total de desvios:** [N] corrigidos automaticamente ([detalhamento por regra])
**Impacto no plano:** [Breve avaliação - ex., "Todas as correções automáticas necessárias para correção/segurança. Sem expansão de escopo."]

## Problemas Encontrados
[Problemas e como foram resolvidos, ou "Nenhum"]

[Nota: "Desvios do Plano" documenta trabalho não planejado que foi tratado automaticamente via regras de desvio. "Problemas Encontrados" documenta problemas durante trabalho planejado que requereram resolução.]

## Configuração do Usuário Necessária

[Se USER-SETUP.md foi gerado:]
**Serviços externos requerem configuração manual.** Veja [{fase}-USER-SETUP.md](./{fase}-USER-SETUP.md) para:
- Variáveis de ambiente a adicionar
- Passos de configuração no dashboard
- Comandos de verificação

[Se sem USER-SETUP.md:]
Nenhuma - sem configuração de serviço externo necessária.

## Prontidão para Próxima Fase
[O que está pronto para a próxima fase]
[Quaisquer bloqueios ou preocupações]

---
*Fase: XX-nome*
*Concluída: [data]*
```

<frontmatter_guidance>
**Propósito:** Habilitar montagem automática de contexto via grafo de dependências. Frontmatter torna metadados do resumo legíveis por máquina para que plan-phase possa escanear todos os resumos rapidamente e selecionar relevantes baseado em dependências.

**Escaneamento rápido:** Frontmatter são as primeiras ~25 linhas, baratas de escanear em todos os resumos sem ler o conteúdo completo.

**Grafo de dependências:** `requires`/`provides`/`affects` criam links explícitos entre fases, habilitando fechamento transitivo para seleção de contexto.

**Subsystem:** Categorização primária (auth, payments, ui, api, database, infra, testing) para detectar fases relacionadas.

**Tags:** Palavras-chave técnicas pesquisáveis (bibliotecas, frameworks, ferramentas) para consciência da stack técnica.

**Key-files:** Arquivos importantes para referências @context no PLAN.md.

**Patterns:** Convenções estabelecidas que fases futuras devem manter.

**Preenchimento:** Frontmatter é preenchido durante criação do resumo no execute-plan.md. Veja `<step name="create_summary">` para orientação campo a campo.
</frontmatter_guidance>

<one_liner_rules>
O resumo de uma linha DEVE ser substantivo:

**Bom:**
- "Auth JWT com rotação de refresh usando biblioteca jose"
- "Schema Prisma com modelos User, Session e Product"
- "Dashboard com métricas em tempo real via Server-Sent Events"

**Ruim:**
- "Fase concluída"
- "Autenticação implementada"
- "Fundação finalizada"
- "Todas as tarefas feitas"

O resumo de uma linha deve dizer a alguém o que realmente foi entregue.
</one_liner_rules>

<example>
```markdown
# Fase 1: Fundação Resumo

**Auth JWT com rotação de refresh usando biblioteca jose, modelo Prisma User e middleware de API protegida**

## Performance

- **Duração:** 28 min
- **Início:** 2025-01-15T14:22:10Z
- **Conclusão:** 2025-01-15T14:50:33Z
- **Tarefas:** 5
- **Arquivos modificados:** 8

## Conquistas
- Modelo de usuário com auth email/senha
- Endpoints de login/logout com cookies JWT httpOnly
- Middleware de rota protegida verificando validade do token
- Rotação de refresh token em cada request

## Arquivos Criados/Modificados
- `prisma/schema.prisma` - Modelos User e Session
- `src/app/api/auth/login/route.ts` - Endpoint de login
- `src/app/api/auth/logout/route.ts` - Endpoint de logout
- `src/middleware.ts` - Verificações de rota protegida
- `src/lib/auth.ts` - Helpers JWT usando jose

## Decisões Tomadas
- Usou jose ao invés de jsonwebtoken (ESM-nativo, compatível com Edge)
- Access tokens de 15 min com refresh tokens de 7 dias
- Armazenamento de refresh tokens no banco para capacidade de revogação

## Desvios do Plano

### Correções Automáticas

**1. [Regra 2 - Ausência Crítica] Adicionado hash de senha com bcrypt**
- **Encontrado durante:** Tarefa 2 (Implementação do endpoint de login)
- **Problema:** Plano não especificou hash de senha - armazenar texto plano seria falha crítica de segurança
- **Correção:** Adicionado hash bcrypt no registro, comparação no login com salt rounds 10
- **Arquivos modificados:** src/app/api/auth/login/route.ts, src/lib/auth.ts
- **Verificação:** Teste de hash de senha passa, texto plano nunca armazenado
- **Commitado em:** abc123f (commit da Tarefa 2)

**2. [Regra 3 - Bloqueante] Instalada dependência jose ausente**
- **Encontrado durante:** Tarefa 4 (Geração de token JWT)
- **Problema:** Pacote jose não estava no package.json, import falhando
- **Correção:** Executou `npm install jose`
- **Arquivos modificados:** package.json, package-lock.json
- **Verificação:** Import funciona, build passa
- **Commitado em:** def456g (commit da Tarefa 4)

---

**Total de desvios:** 2 corrigidos automaticamente (1 ausência crítica, 1 bloqueante)
**Impacto no plano:** Ambas correções automáticas essenciais para segurança e funcionalidade. Sem expansão de escopo.

## Problemas Encontrados
- Import CommonJS do jsonwebtoken falhou no Edge runtime - trocou para jose (mudança planejada de biblioteca, funcionou conforme esperado)

## Prontidão para Próxima Fase
- Fundação de auth completa, pronta para desenvolvimento de funcionalidades
- Endpoint de registro de usuário necessário antes do lançamento público

---
*Fase: 01-fundacao*
*Concluída: 2025-01-15*
```
</example>

<guidelines>
**Frontmatter:** OBRIGATÓRIO - completar todos os campos. Habilita montagem automática de contexto para planejamento futuro.

**Resumo de uma linha:** Deve ser substantivo. "Auth JWT com rotação de refresh usando biblioteca jose" não "Autenticação implementada".

**Seção de decisões:**
- Decisões chave tomadas durante execução com justificativa
- Extraídas para contexto acumulado do STATE.md
- Use "Nenhuma - seguiu o plano conforme especificado" se sem desvios

**Após criação:** STATE.md atualizado com posição, decisões, problemas.
</guidelines>
