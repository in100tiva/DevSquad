# DevSquad Review Runner — `_devsquad/core/runner.review.md`

O runner **sempre** carrega **Lucas Tech Lead** como ponto de entrada.
Caminho base dos membros da equipe: **`_devsquad/agents/tech-lead/team/`**.

---

## PASSO 0-PRE — MCP Check (antes de tudo)

1. Ler **[_devsquad/core/mcp-check.md](_devsquad/core/mcp-check.md)**.
2. Executar a verificação dos MCPs obrigatórios (context7, playwright); opcional: MCP_DOCKER.
3. **Se context7 ou playwright ausentes** → exibir a mensagem de bloqueio exata do mcp-check.md e **não prosseguir**; referenciar **[_devsquad/PREREQUISITES.md](_devsquad/PREREQUISITES.md)** para instruções.
4. **Se apenas Docker ausente** → avisar sem bloquear.

---

## PASSO 1 — Carregar Lucas

1. Ler **[_devsquad/agents/tech-lead/SKILL.md](_devsquad/agents/tech-lead/SKILL.md)**.
2. Lucas executa **Passo 0** (ler `_devsquad/_memory/preferences.md`; aplicar ou fazer onboarding e salvar).
3. Identificar **modo** e **escopo** a partir do pedido do usuário.

---

## PASSO 2 — Carregar a task do Lucas

- **CREATE** → [_devsquad/agents/tech-lead/CREATE.md](_devsquad/agents/tech-lead/CREATE.md)
- **ANALYZE** → [_devsquad/agents/tech-lead/ANALYZE.md](_devsquad/agents/tech-lead/ANALYZE.md)
- **REFACTOR** → [_devsquad/agents/tech-lead/REFACTOR.md](_devsquad/agents/tech-lead/REFACTOR.md)

---

## PASSO 3 — Para cada membro convocado pela task

1. Lucas inicializa e mantém o **HANDOFF** (objeto de passagem de contexto tipado).
2. **Antes de cada membro:** aplicar **Scope Gate** (tabela de quando pular cada membro conforme HANDOFF.scope.type).
3. Ler **SKILL** + **task** do membro em `_devsquad/agents/tech-lead/team/{membro}/`.
4. O membro lê HANDOFF, preenche **apenas o seu campo** e entrega.
5. Capturar output no HANDOFF (não blob de texto).
6. **No REFACTOR:** após Etapa A (Rafael, Diana, Camila), Lucas executa **MERGE STEP** e preenche `merged_target` antes da Etapa B.
7. **No ANALYZE:** aplicar **checkpoints de early exit** (após Nadia+César e após Giovana+Camila).
8. Retornar ao modo Lucas entre membros.

**Regras:** não alterar código sem consentimento; em modo refactor, sugerir passos e perguntar antes de aplicar.

---

## PASSO 4 — Síntese final

Lucas sintetiza a partir do HANDOFF em um **único plano consolidado**.
Ao final, **oferta de deepening** (ver SKILL.md do Lucas — Passo 5).

---

## PASSO 5 — Deepening (quando o usuário escolhe aprofundar)

Se o usuário escolher uma opção de deepening (1–5):

1. **NÃO** reinicializar o HANDOFF — usar o já preenchido na sessão.
2. **NÃO** repetir os passos anteriores.
3. Carregar **apenas** o membro escolhido: `_devsquad/agents/tech-lead/team/{membro}/SKILL.md` e `_devsquad/agents/tech-lead/team/{membro}/{TASK}.md` (mesma task da sessão: CREATE, ANALYZE ou REFACTOR).
4. Passar **HANDOFF completo** como contexto.
5. O membro aprofunda **apenas o seu campo**.
6. Lucas **adiciona** o output ao plano existente (não substitui).

---

## Resumo do caminho base

| O quê | Onde |
|-------|------|
| Lucas (ponto de entrada) | `_devsquad/agents/tech-lead/` |
| Membros da equipe | `_devsquad/agents/tech-lead/team/` |
| Tasks do Lucas | CREATE.md, ANALYZE.md, REFACTOR.md em `tech-lead/` |
