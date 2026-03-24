<purpose>
Revisão por pares entre IAs — invocar CLIs externos de IA para revisar planos de fase independentemente.
Cada CLI recebe o mesmo prompt (contexto do PROJECT.md, planos de fase, requisitos) e
produz feedback estruturado. Os resultados são combinados em REVIEWS.md para o planejador
incorporar via flag --reviews.

Isto implementa revisão adversarial: diferentes modelos de IA capturam diferentes pontos cegos.
Um plano que sobrevive à revisão de 2-3 sistemas de IA independentes é mais robusto.
</purpose>

<process>

<step name="detect_clis">
Verificar quais CLIs de IA estão disponíveis no sistema:

```bash
# Verificar cada CLI
command -v gemini >/dev/null 2>&1 && echo "gemini:available" || echo "gemini:missing"
command -v claude >/dev/null 2>&1 && echo "claude:available" || echo "claude:missing"
command -v codex >/dev/null 2>&1 && echo "codex:available" || echo "codex:missing"
```

Analisar flags de `{{GSD_ARGS}}`:
- `--gemini` → incluir Gemini
- `--claude` → incluir Claude
- `--codex` → incluir Codex
- `--all` → incluir todos disponíveis
- Sem flags → incluir todos disponíveis

Se nenhum CLI estiver disponível:
```
Nenhum CLI externo de IA encontrado. Instale pelo menos um:
- gemini: https://github.com/google-gemini/gemini-cli
- codex: https://github.com/openai/codex
- claude: https://github.com/anthropics/claude-code

Então execute /gsd-revisar novamente.
```
Sair.

Se apenas um CLI for o runtime atual (ex: rodando dentro do Claude), pulá-lo para a revisão
para garantir independência. Pelo menos um CLI DIFERENTE deve estar disponível.
</step>

<step name="gather_context">
Coletar artefatos da fase para o prompt de revisão:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Ler do init: `phase_dir`, `phase_number`, `padded_phase`.

Então ler:
1. `.planning/PROJECT.md` (primeiras 80 linhas — contexto do projeto)
2. Seção da fase no `.planning/ROADMAP.md`
3. Todos os arquivos `*-PLAN.md` no diretório da fase
4. `*-CONTEXT.md` se presente (decisões do usuário)
5. `*-RESEARCH.md` se presente (pesquisa de domínio)
6. `.planning/REQUIREMENTS.md` (requisitos que esta fase atende)
</step>

<step name="build_prompt">
Construir um prompt de revisão estruturado:

```markdown
# Solicitação de Revisão de Plano entre IAs

Você está revisando planos de implementação para uma fase de projeto de software.
Forneça feedback estruturado sobre qualidade, completude e riscos do plano.

## Contexto do Projeto
{primeiras 80 linhas do PROJECT.md}

## Fase {N}: {nome da fase}
### Seção do Roteiro
{seção da fase no roteiro}

### Requisitos Atendidos
{requisitos para esta fase}

### Decisões do Usuário (CONTEXT.md)
{contexto se presente}

### Resultados da Pesquisa
{pesquisa se presente}

### Planos para Revisar
{conteúdo de todos os PLAN.md}

## Instruções de Revisão

Analise cada plano e forneça:

1. **Resumo** — Avaliação de um parágrafo
2. **Pontos Fortes** — O que está bem projetado (itens com marcador)
3. **Preocupações** — Problemas potenciais, lacunas, riscos (itens com marcador e severidade: ALTA/MÉDIA/BAIXA)
4. **Sugestões** — Melhorias específicas (itens com marcador)
5. **Avaliação de Risco** — Nível geral de risco (BAIXO/MÉDIO/ALTO) com justificativa

Foque em:
- Casos extremos faltando ou tratamento de erros
- Problemas de ordenação de dependências
- Aumento de escopo ou super-engenharia
- Considerações de segurança
- Implicações de performance
- Se os planos realmente alcançam os objetivos da fase

Forneça sua revisão em formato markdown.
```

Escrever em arquivo temporário: `/tmp/gsd-revisar-prompt-{fase}.md`
</step>

<step name="invoke_reviewers">
Para cada CLI selecionado, invocar em sequência (não em paralelo — evitar limites de taxa):

**Gemini:**
```bash
gemini -p "$(cat /tmp/gsd-revisar-prompt-{fase}.md)" 2>/dev/null > /tmp/gsd-revisar-gemini-{fase}.md
```

**Claude (sessão separada):**
```bash
claude -p "$(cat /tmp/gsd-revisar-prompt-{fase}.md)" --no-input 2>/dev/null > /tmp/gsd-revisar-claude-{fase}.md
```

**Codex:**
```bash
codex -p "$(cat /tmp/gsd-revisar-prompt-{fase}.md)" 2>/dev/null > /tmp/gsd-revisar-codex-{fase}.md
```

Se um CLI falhar, registrar o erro e continuar com os CLIs restantes.

Exibir progresso:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REVISÃO ENTRE IAs — Fase {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Revisando com {CLI}... concluído ✓
◆ Revisando com {CLI}... concluído ✓
```
</step>

<step name="write_reviews">
Combinar todas as respostas de revisão em `{phase_dir}/{padded_phase}-REVIEWS.md`:

```markdown
---
phase: {N}
reviewers: [gemini, claude, codex]
reviewed_at: {timestamp ISO}
plans_reviewed: [{lista de arquivos PLAN.md}]
---

# Revisão de Plano entre IAs — Fase {N}

## Revisão Gemini

{conteúdo da revisão gemini}

---

## Revisão Claude

{conteúdo da revisão claude}

---

## Revisão Codex

{conteúdo da revisão codex}

---

## Resumo de Consenso

{sintetizar preocupações comuns entre todos os revisores}

### Pontos Fortes em Acordo
{pontos fortes mencionados por 2+ revisores}

### Preocupações em Acordo
{preocupações levantadas por 2+ revisores — maior prioridade}

### Visões Divergentes
{onde revisores discordaram — vale investigar}
```

Commitar:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: revisão entre IAs para fase {N}" --files {phase_dir}/{padded_phase}-REVIEWS.md
```
</step>

<step name="present_results">
Exibir resumo:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REVISÃO CONCLUÍDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Fase {N} revisada por {count} sistemas de IA.

Preocupações em consenso:
{top 3 preocupações compartilhadas}

Revisão completa: {padded_phase}-REVIEWS.md

Para incorporar feedback no planejamento:
  /gsd-planejar-fase {N} --reviews
```

Limpar arquivos temporários.
</step>

</process>

<success_criteria>
- [ ] Pelo menos um CLI externo invocado com sucesso
- [ ] REVIEWS.md escrito com feedback estruturado
- [ ] Resumo de consenso sintetizado de múltiplos revisores
- [ ] Arquivos temporários limpos
- [ ] Usuário sabe como usar o feedback (/gsd-planejar-fase --reviews)
</success_criteria>
