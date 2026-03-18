---
name: devsquad
description: DevSquad é um framework de code review baseado em agentes especialistas coordenados por Lucas Tech Lead. Acionar com /devsquad ou ao mencionar "analise", "revise", "refatore", "crie do zero", "diagnóstico", "clean code", "clean architecture", "DDD", "design patterns", "software architecture", "affordance", "feedback", "modelo mental", "microservices", "event-driven", "bounded context", "aggregate", "dependency rule", "use case", "tech lead", "lucas", "devsquad", etc. Lucas Tech Lead é sempre o primeiro passo — ele recebe o pedido e decide quem da equipe convocar.
---

# DevSquad — Code Review com Agentes

## Antes de começar

- O usuário deve ler **[_devsquad/PREREQUISITES.md](_devsquad/PREREQUISITES.md)** (pré-requisitos de MCPs: Context7, Playwright, Docker MCP Toolkit — para que servem, limites free tier, como criar conta/chave, configuração do mcp.json, troubleshooting).
- O runner executa o check definido em **[_devsquad/core/mcp-check.md](_devsquad/core/mcp-check.md)** (Passo 0-PRE) **antes** de carregar Lucas. Se MCPs obrigatórios estiverem ausentes, exibe as mensagens de bloqueio e aponta para PREREQUISITES.md.

---

## Como o DevSquad funciona

1. **Lucas Tech Lead é o único ponto de entrada.** Todos os pedidos passam por Lucas primeiro.
2. Ao receber qualquer pedido relacionado a código ou arquitetura:
   - Carregar **[_devsquad/agents/tech-lead/SKILL.md](_devsquad/agents/tech-lead/SKILL.md)**.
   - Lucas identifica o **modo** (criar / analisar / refatorar) e o **escopo**.
   - Lucas carrega a **task** correspondente (CREATE / ANALYZE / REFACTOR).
   - A task define **quais membros da equipe** convocar e **em qual ordem**.
   - Lucas entrega o **plano consolidado** (não relatórios separados por membro).

---

## Comandos

| Comando | Ação |
|--------|------|
| `/devsquad` | Menu principal — Lucas recebe e pergunta o modo se não estiver claro |
| `/devsquad analyze` | Lucas em modo ANALYZE |
| `/devsquad create` | Lucas em modo CREATE |
| `/devsquad refactor` | Lucas em modo REFACTOR |
| `/devsquad team` | Listar membros da equipe (usa [_devsquad/core/agents.engine.md](_devsquad/core/agents.engine.md) em tech-lead/team/) |
| `/devsquad project` | Análise do projeto ([_devsquad/core/project-analyzer.md](_devsquad/core/project-analyzer.md)) |
| `/devsquad prereqs` | Mostrar status dos MCPs (context7, playwright, docker) sem executar a cadeia — leitura do resultado do Passo 0-PRE |
| `/devsquad help` | Instruções e comandos disponíveis |

---

## Roteamento

- **Qualquer pedido** de análise, revisão, criação ou refatoração → **[_devsquad/agents/tech-lead/SKILL.md](_devsquad/agents/tech-lead/SKILL.md)**.
- **Listagem de agentes** → **[_devsquad/core/agents.engine.md](_devsquad/core/agents.engine.md)** (ler membros em `_devsquad/agents/tech-lead/team/`).
- **Análise do projeto** → **[_devsquad/core/project-analyzer.md](_devsquad/core/project-analyzer.md)**.

---

## Execução (runner)

O fluxo completo está em **[_devsquad/core/runner.review.md](_devsquad/core/runner.review.md)**:

- Passo 0-PRE: MCP check (mcp-check.md).
- Passos 1–4: Carregar Lucas → task → membros → síntese.
- Passo 5: Deepening (quando o usuário escolhe aprofundar uma área).

Caminho base dos membros da equipe: **`_devsquad/agents/tech-lead/team/`**.
