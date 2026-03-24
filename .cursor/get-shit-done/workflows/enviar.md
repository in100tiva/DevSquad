<purpose>
Criar um pull request do trabalho de fase/marco concluído, gerar um corpo rico de PR a partir dos artefatos de planejamento, opcionalmente executar revisão de código e preparar para merge. Fecha o ciclo planejar → executar → verificar → enviar.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="initialize">
Analisar argumentos e carregar estado do projeto:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `commit_docs`.

Também carregar config para estratégia de branching:
```bash
CONFIG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state load)
```

Extrair: `branching_strategy`, `branch_name`.
</step>

<step name="preflight_checks">
Verificar se o trabalho está pronto para envio:

1. **Verificação aprovada?**
   ```bash
   VERIFICATION=$(cat ${PHASE_DIR}/*-VERIFICATION.md 2>/dev/null)
   ```
   Verificar `status: passed` ou `status: human_needed` (com aprovação humana).
   Se não houver VERIFICATION.md ou status for `gaps_found`: avisar e pedir confirmação ao usuário.

2. **Árvore de trabalho limpa?**
   ```bash
   git status --short
   ```
   Se houver mudanças não commitadas: pedir ao usuário para commitar ou fazer stash primeiro.

3. **Na branch correta?**
   ```bash
   CURRENT_BRANCH=$(git branch --show-current)
   ```
   Se estiver em `main`/`master`: avisar — deveria estar em uma branch de feature.
   Se branching_strategy for `none`: oferecer criar uma branch agora.

4. **Remote configurado?**
   ```bash
   git remote -v | head -2
   ```
   Detectar remote `origin`. Se não houver remote: erro — não é possível criar PR.

5. **CLI `gh` disponível?**
   ```bash
   which gh && gh auth status 2>&1
   ```
   Se `gh` não encontrado ou não autenticado: fornecer instruções de configuração e sair.
</step>

<step name="push_branch">
Enviar a branch atual para o remote:

```bash
git push origin ${CURRENT_BRANCH} 2>&1
```

Se o push falhar (ex: sem upstream): definir upstream:
```bash
git push --set-upstream origin ${CURRENT_BRANCH} 2>&1
```

Reportar: "Enviado `{branch}` para origin ({commit_count} commits à frente de main)"
</step>

<step name="generate_pr_body">
Auto-gerar um corpo rico de PR a partir dos artefatos de planejamento:

**1. Título:**
```
Fase {phase_number}: {phase_name}
```
Ou para marco: `Marco {version}: {name}`

**2. Seção de resumo:**
Ler ROADMAP.md para objetivo da fase. Ler VERIFICATION.md para status de verificação.

```markdown
## Resumo

**Fase {N}: {Nome}**
**Objetivo:** {objetivo do ROADMAP.md}
**Status:** Verificado ✓

{Um parágrafo sintetizado dos arquivos SUMMARY.md — o que foi construído}
```

**3. Seção de mudanças:**
Para cada SUMMARY.md no diretório da fase:
```markdown
## Mudanças

### Plano {plan_id}: {plan_name}
{one_liner do frontmatter do SUMMARY.md}

**Arquivos-chave:**
{key-files.created e key-files.modified do frontmatter do SUMMARY.md}
```

**4. Seção de requisitos:**
```markdown
## Requisitos Atendidos

{REQ-IDs do frontmatter do plano, vinculados às descrições do REQUIREMENTS.md}
```

**5. Seção de testes:**
```markdown
## Verificação

- [x] Verificação automatizada: {aprovado/reprovado do VERIFICATION.md}
- {itens de verificação humana do VERIFICATION.md, se houver}
```

**6. Seção de decisões:**
```markdown
## Decisões-Chave

{Decisões do contexto acumulado do STATE.md relevantes a esta fase}
```
</step>

<step name="create_pr">
Criar o PR usando o corpo gerado:

```bash
gh pr create \
  --title "Fase ${PHASE_NUMBER}: ${PHASE_NAME}" \
  --body "${PR_BODY}" \
  --base main
```

Se flag `--draft` foi passada: adicionar `--draft`.

Reportar: "PR #{number} criado: {url}"
</step>

<step name="optional_review">
Perguntar se o usuário quer disparar uma revisão de código:

```
conversational prompting:
  question: "PR criado. Executar revisão de código antes do merge?"
  options:
    - label: "Pular revisão"
      description: "PR está pronto — merge quando CI passar"
    - label: "Auto-revisão"
      description: "Vou revisar o diff no PR eu mesmo"
    - label: "Solicitar revisão"
      description: "Solicitar revisão de um colega"
```

**Se "Solicitar revisão":**
```bash
gh pr edit ${PR_NUMBER} --add-reviewer "${REVIEWER}"
```

**Se "Auto-revisão":**
Reportar a URL do PR e sugerir: "Revise o diff em {url}/files"
</step>

<step name="track_shipping">
Atualizar STATE.md para refletir a ação de envio:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state update "Last Activity" "$(date +%Y-%m-%d)"
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state update "Status" "Fase ${PHASE_NUMBER} enviada — PR #${PR_NUMBER}"
```

Se `commit_docs` for true:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): enviar fase ${PHASE_NUMBER} — PR #${PR_NUMBER}" --files .planning/STATE.md
```
</step>

<step name="report">
```
───────────────────────────────────────────────────────────────

## ✓ Fase {X}: {Nome} — Enviada

PR: #{number} ({url})
Branch: {branch} → main
Commits: {count}
Verificação: ✓ Aprovada
Requisitos: {N} REQ-IDs atendidos

Próximos passos:
- Revisar/aprovar PR
- Merge quando CI passar
- /gsd-complete-milestone (se última fase do marco)
- /gsd-progress (para ver o que vem a seguir)

───────────────────────────────────────────────────────────────
```
</step>

</process>

<offer_next>
Após envio:

- /gsd-complete-milestone — se todas as fases do marco estão concluídas
- /gsd-progress — ver estado geral do projeto
- /gsd-execute-phase {próxima} — continuar para próxima fase
</offer_next>

<success_criteria>
- [ ] Verificações pré-voo aprovadas (verificação, árvore limpa, branch, remote, gh)
- [ ] Branch enviada para remote
- [ ] PR criado com corpo rico auto-gerado
- [ ] STATE.md atualizado com status de envio
- [ ] Usuário sabe número do PR e próximos passos
</success_criteria>
