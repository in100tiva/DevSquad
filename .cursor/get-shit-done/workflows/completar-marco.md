<purpose>

Marcar uma versão entregue (v1.0, v1.1, v2.0) como completa. Cria registro histórico no MILESTONES.md, realiza revisão completa da evolução do PROJECT.md, reorganiza o ROADMAP.md com agrupamentos de marcos, e cria tag de release no git.

</purpose>

<required_reading>

1. templates/marco.md
2. templates/arquivo-marco.md
3. `.planning/ROADMAP.md`
4. `.planning/REQUIREMENTS.md`
5. `.planning/PROJECT.md`

</required_reading>

<archival_behavior>

Quando um marco é completado:

1. Extrair detalhes completos do marco para `.planning/milestones/v[X.Y]-ROADMAP.md`
2. Arquivar requisitos para `.planning/milestones/v[X.Y]-REQUIREMENTS.md`
3. Atualizar ROADMAP.md — substituir detalhes do marco por resumo de uma linha
4. Deletar REQUIREMENTS.md (novo para próximo marco)
5. Realizar revisão completa da evolução do PROJECT.md
6. Oferecer criação do próximo marco inline
7. Arquivar artefatos de UI (`*-UI-SPEC.md`, `*-UI-REVIEW.md`) junto com outros documentos de fase
8. Limpar arquivos de screenshot em `.planning/ui-reviews/` (assets binários, nunca arquivados)

**Eficiência de Contexto:** Arquivos mantêm ROADMAP.md em tamanho constante e REQUIREMENTS.md com escopo de marco.

**Arquivo de ROADMAP** usa `templates/arquivo-marco.md` — inclui cabeçalho do marco (status, fases, data), detalhes completos das fases, resumo do marco (decisões, problemas, dívida técnica).

**Arquivo de REQUIREMENTS** contém todos os requisitos marcados como completos com resultados, tabela de rastreabilidade com status final, notas sobre requisitos alterados.

</archival_behavior>

<process>

<step name="verify_readiness">

**Usar `roadmap analyze` para verificação abrangente de prontidão:**

```bash
ROADMAP=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Isto retorna todas as fases com contagens de plano/resumo e status em disco. Usar para verificar:
- Quais fases pertencem a este marco?
- Todas as fases completas (todos os planos têm resumos)? Verificar `disk_status === 'complete'` para cada.
- `progress_percent` deve ser 100%.

**Verificação de completude de requisitos (OBRIGATÓRIA antes de apresentar):**

Analisar tabela de rastreabilidade do REQUIREMENTS.md:
- Contar total de requisitos v1 vs requisitos marcados (`[x]`)
- Identificar quaisquer linhas não-Completas na tabela de rastreabilidade

Apresentar:

```
Marco: [Nome, ex., "v1.0 MVP"]

Inclui:
- Fase 1: Fundação (2/2 planos completos)
- Fase 2: Autenticação (2/2 planos completos)
- Fase 3: Funcionalidades Core (3/3 planos completos)
- Fase 4: Polimento (1/1 plano completo)

Total: {phase_count} fases, {total_plans} planos, todos completos
Requisitos: {N}/{M} requisitos v1 marcados
```

**Se requisitos incompletos** (N < M):

```
⚠ Requisitos Não Marcados:

- [ ] {REQ-ID}: {descrição} (Fase {X})
- [ ] {REQ-ID}: {descrição} (Fase {Y})
```

DEVE apresentar 3 opções:
1. **Prosseguir mesmo assim** — marcar marco como completo com lacunas conhecidas
2. **Executar auditoria primeiro** — `/gsd-auditar-marco` para avaliar severidade das lacunas
3. **Abortar** — retornar ao desenvolvimento

Se o usuário selecionar "Prosseguir mesmo assim": anotar requisitos incompletos no MILESTONES.md em `### Lacunas Conhecidas` com REQ-IDs e descrições.

<config-check>

```bash
cat .planning/config.json 2>/dev/null
```

</config-check>

<if mode="yolo">

```
⚡ Auto-aprovado: Verificação de escopo do marco
[Mostrar resumo sem solicitar]
Prosseguindo para coleta de estatísticas...
```

Prosseguir para gather_stats.

</if>

<if mode="interactive" OR="custom with gates.confirm_milestone_scope true">

```
Pronto para marcar este marco como entregue?
(sim / esperar / ajustar escopo)
```

Aguardar confirmação.
- "ajustar escopo": Perguntar quais fases incluir.
- "esperar": Parar, usuário retorna quando pronto.

</if>

</step>

<step name="gather_stats">

Calcular estatísticas do marco:

```bash
git log --oneline --grep="feat(" | head -20
git diff --stat FIRST_COMMIT..LAST_COMMIT | tail -1
find . -name "*.swift" -o -name "*.ts" -o -name "*.py" | xargs wc -l 2>/dev/null
git log --format="%ai" FIRST_COMMIT | tail -1
git log --format="%ai" LAST_COMMIT | head -1
```

Apresentar:

```
Estatísticas do Marco:
- Fases: [X-Y]
- Planos: [Z] total
- Tarefas: [N] total (dos resumos das fases)
- Arquivos modificados: [M]
- Linhas de código: [LOC] [linguagem]
- Cronograma: [Dias] dias ([Início] → [Fim])
- Intervalo Git: feat(XX-XX) → feat(YY-YY)
```

</step>

<step name="extract_accomplishments">

Extrair resumos de uma linha dos arquivos SUMMARY.md usando summary-extract:

```bash
# Para cada fase no marco, extrair resumo de uma linha
for summary in .planning/phases/*-*/*-SUMMARY.md; do
  node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" summary-extract "$summary" --fields one_liner --pick one_liner
done
```

Extrair 4-6 realizações principais. Apresentar:

```
Realizações principais deste marco:
1. [Realização da fase 1]
2. [Realização da fase 2]
3. [Realização da fase 3]
4. [Realização da fase 4]
5. [Realização da fase 5]
```

</step>

<step name="create_milestone_entry">

**Nota:** A entrada no MILESTONES.md agora é criada automaticamente por `gsd-tools milestone complete` no passo archive_milestone. A entrada inclui versão, data, contagens de fase/plano/tarefa e realizações extraídas dos arquivos SUMMARY.md.

Se detalhes adicionais forem necessários (ex., resumo "Entregue" fornecido pelo usuário, intervalo git, estatísticas de LOC), adicioná-los manualmente após o CLI criar a entrada base.

</step>

<step name="evolve_project_full_review">

Revisão completa de evolução do PROJECT.md na conclusão do marco.

Ler todos os resumos de fase:

```bash
cat .planning/phases/*-*/*-SUMMARY.md
```

**Checklist de revisão completa:**

1. **Precisão de "O Que É Isto":**
   - Comparar descrição atual com o que foi construído
   - Atualizar se o produto mudou significativamente

2. **Verificação do Valor Core:**
   - Ainda a prioridade certa? A entrega revelou um valor core diferente?
   - Atualizar se A COISA mudou

3. **Auditoria de requisitos:**

   **Seção Validados:**
   - Todos os requisitos Ativos entregues neste marco → Mover para Validados
   - Formato: `- ✓ [Requisito] — v[X.Y]`

   **Seção Ativos:**
   - Remover requisitos movidos para Validados
   - Adicionar novos requisitos para o próximo marco
   - Manter requisitos não tratados

   **Auditoria de Fora do Escopo:**
   - Revisar cada item — justificativa ainda válida?
   - Remover itens irrelevantes
   - Adicionar requisitos invalidados durante o marco

4. **Atualização de contexto:**
   - Estado atual do codebase (LOC, stack tecnológica)
   - Temas de feedback de usuários (se houver)
   - Problemas conhecidos ou dívida técnica

5. **Auditoria de Decisões Chave:**
   - Extrair todas as decisões dos resumos das fases do marco
   - Adicionar à tabela de Decisões Chave com resultados
   - Marcar ✓ Bom, ⚠️ Revisitar, ou — Pendente

6. **Verificação de restrições:**
   - Alguma restrição mudou durante o desenvolvimento? Atualizar conforme necessário

Atualizar PROJECT.md inline. Atualizar rodapé "Última atualização":

```markdown
---
*Última atualização: [data] após marco v[X.Y]*
```

**Exemplo de evolução completa (v1.0 → preparação v1.1):**

Antes:

```markdown
## O Que É Isto

Um quadro branco colaborativo em tempo real para equipes remotas.

## Valor Core

Sincronização em tempo real que parece instantânea.

## Requisitos

### Validados

(Nenhum ainda — entregue para validar)

### Ativos

- [ ] Ferramentas de desenho em canvas
- [ ] Sincronização em tempo real < 500ms
- [ ] Autenticação de usuário
- [ ] Exportar para PNG

### Fora do Escopo

- App mobile — abordagem web-first
- Chat de vídeo — usar ferramentas externas
```

Após v1.0:

```markdown
## O Que É Isto

Um quadro branco colaborativo em tempo real para equipes remotas com sincronização instantânea e ferramentas de desenho.

## Valor Core

Sincronização em tempo real que parece instantânea.

## Requisitos

### Validados

- ✓ Ferramentas de desenho em canvas — v1.0
- ✓ Sincronização em tempo real < 500ms — v1.0 (alcançado 200ms média)
- ✓ Autenticação de usuário — v1.0

### Ativos

- [ ] Exportar para PNG
- [ ] Histórico de desfazer/refazer
- [ ] Ferramentas de forma (retângulos, círculos)

### Fora do Escopo

- App mobile — abordagem web-first, PWA funciona bem
- Chat de vídeo — usar ferramentas externas
- Modo offline — tempo real é o valor core

## Contexto

Entregue v1.0 com 2.400 LOC TypeScript.
Stack: Next.js, Supabase, Canvas API.
Testes iniciais com usuários mostraram demanda por ferramentas de forma.
```

**Passo completo quando:**

- [ ] "O Que É Isto" revisado e atualizado se necessário
- [ ] Valor Core verificado como ainda correto
- [ ] Todos os requisitos entregues movidos para Validados
- [ ] Novos requisitos adicionados aos Ativos para próximo marco
- [ ] Justificativas de Fora do Escopo auditadas
- [ ] Contexto atualizado com estado atual
- [ ] Todas as decisões do marco adicionadas às Decisões Chave
- [ ] Rodapé "Última atualização" reflete conclusão do marco

</step>

<step name="reorganize_roadmap">

Atualizar `.planning/ROADMAP.md` — agrupar fases do marco completado:

```markdown
# Roteiro: [Nome do Projeto]

## Marcos

- ✅ **v1.0 MVP** — Fases 1-4 (entregue AAAA-MM-DD)
- 🚧 **v1.1 Segurança** — Fases 5-6 (em progresso)
- 📋 **v2.0 Redesign** — Fases 7-10 (planejado)

## Fases

<details>
<summary>✅ v1.0 MVP (Fases 1-4) — ENTREGUE AAAA-MM-DD</summary>

- [x] Fase 1: Fundação (2/2 planos) — completada AAAA-MM-DD
- [x] Fase 2: Autenticação (2/2 planos) — completada AAAA-MM-DD
- [x] Fase 3: Funcionalidades Core (3/3 planos) — completada AAAA-MM-DD
- [x] Fase 4: Polimento (1/1 plano) — completada AAAA-MM-DD

</details>

### 🚧 v[Próxima] [Nome] (Em Progresso / Planejado)

- [ ] Fase 5: [Nome] ([N] planos)
- [ ] Fase 6: [Nome] ([N] planos)

## Progresso

| Fase              | Marco     | Planos Completos | Status      | Completada |
| ----------------- | --------- | ---------------- | ----------- | ---------- |
| 1. Fundação       | v1.0      | 2/2              | Completa    | AAAA-MM-DD |
| 2. Autenticação   | v1.0      | 2/2              | Completa    | AAAA-MM-DD |
| 3. Funcionalidades| v1.0      | 3/3              | Completa    | AAAA-MM-DD |
| 4. Polimento      | v1.0      | 1/1              | Completa    | AAAA-MM-DD |
| 5. Auditoria Seg. | v1.1      | 0/1              | Não iniciada| -          |
| 6. Hardening      | v1.1      | 0/2              | Não iniciada| -          |
```

</step>

<step name="archive_milestone">

**Delegar arquivamento ao gsd-tools:**

```bash
ARCHIVE=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" milestone complete "v[X.Y]" --name "[Nome do Marco]")
```

O CLI cuida de:
- Criar diretório `.planning/milestones/`
- Arquivar ROADMAP.md para `milestones/v[X.Y]-ROADMAP.md`
- Arquivar REQUIREMENTS.md para `milestones/v[X.Y]-REQUIREMENTS.md` com cabeçalho de arquivo
- Mover arquivo de auditoria para milestones se existir
- Criar/acrescentar entrada no MILESTONES.md com realizações dos arquivos SUMMARY.md
- Atualizar STATE.md (status, última atividade)

Extrair do resultado: `version`, `date`, `phases`, `plans`, `tasks`, `accomplishments`, `archived`.

Verificar: `✅ Marco arquivado em .planning/milestones/`

**Arquivamento de fases (opcional):** Após o arquivamento completar, perguntar ao usuário:

conversational prompting(header="Arquivar Fases", question="Arquivar diretórios de fase para milestones/?", options: "Sim — mover para milestones/v[X.Y]-phases/" | "Pular — manter fases no lugar")

Se "Sim": mover diretórios de fase para o arquivo do marco:
```bash
mkdir -p .planning/milestones/v[X.Y]-phases
# Para cada diretório de fase em .planning/phases/:
mv .planning/phases/{phase-dir} .planning/milestones/v[X.Y]-phases/
```
Verificar: `✅ Diretórios de fase arquivados em .planning/milestones/v[X.Y]-phases/`

Se "Pular": Diretórios de fase permanecem em `.planning/phases/` como histórico bruto de execução. Usar `/gsd-limpeza` depois para arquivar retroativamente.

Após o arquivamento, a IA ainda cuida de:
- Reorganizar ROADMAP.md com agrupamento de marcos (requer julgamento)
- Revisão completa de evolução do PROJECT.md (requer entendimento)
- Deletar ROADMAP.md e REQUIREMENTS.md originais
- Estes NÃO são totalmente delegados porque requerem interpretação de conteúdo pela IA

</step>

<step name="reorganize_roadmap_and_delete_originals">

Após `milestone complete` ter arquivado, reorganizar ROADMAP.md com agrupamentos de marcos, então deletar originais:

**Reorganizar ROADMAP.md** — agrupar fases do marco completado:

```markdown
# Roteiro: [Nome do Projeto]

## Marcos

- ✅ **v1.0 MVP** — Fases 1-4 (entregue AAAA-MM-DD)
- 🚧 **v1.1 Segurança** — Fases 5-6 (em progresso)

## Fases

<details>
<summary>✅ v1.0 MVP (Fases 1-4) — ENTREGUE AAAA-MM-DD</summary>

- [x] Fase 1: Fundação (2/2 planos) — completada AAAA-MM-DD
- [x] Fase 2: Autenticação (2/2 planos) — completada AAAA-MM-DD

</details>
```

**Então deletar originais:**

```bash
rm .planning/ROADMAP.md
rm .planning/REQUIREMENTS.md
```

</step>

<step name="write_retrospective">

**Acrescentar à retrospectiva viva:**

Verificar se existe retrospectiva:
```bash
ls .planning/RETROSPECTIVE.md 2>/dev/null
```

**Se existir:** Ler o arquivo, acrescentar nova seção do marco antes da seção "## Tendências Entre Marcos".

**Se não existir:** Criar a partir do template em `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/retrospectiva.md`.

**Coletar dados da retrospectiva:**

1. Dos arquivos SUMMARY.md: Extrair entregas principais, resumos, decisões técnicas
2. Dos arquivos VERIFICATION.md: Extrair pontuações de verificação, lacunas encontradas
3. Dos arquivos UAT.md: Extrair resultados de testes, problemas encontrados
4. Do git log: Contar commits, calcular cronograma
5. Do trabalho do marco: Refletir sobre o que funcionou e o que não funcionou

**Escrever a seção do marco:**

```markdown
## Marco: v{version} — {nome}

**Entregue:** {data}
**Fases:** {contagem_fases} | **Planos:** {contagem_planos}

### O Que Foi Construído
{Extrair dos resumos de uma linha do SUMMARY.md}

### O Que Funcionou
{Padrões que levaram a execução suave}

### O Que Foi Ineficiente
{Oportunidades perdidas, retrabalho, gargalos}

### Padrões Estabelecidos
{Novas convenções descobertas durante este marco}

### Lições Principais
{Conclusões específicas e acionáveis}

### Observações de Custo
- Mix de modelos: {X}% opus, {Y}% sonnet, {Z}% haiku
- Sessões: {contagem}
- Destaque: {observação de eficiência}
```

**Atualizar tendências entre marcos:**

Se a seção "## Tendências Entre Marcos" existir, atualizar as tabelas com novos dados deste marco.

**Commitar:**
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: update retrospective for v${VERSION}" --files .planning/RETROSPECTIVE.md
```

</step>

<step name="update_state">

A maioria das atualizações do STATE.md foi tratada por `milestone complete`, mas verificar e atualizar campos restantes:

**Referência do Projeto:**

```markdown
## Referência do Projeto

Ver: .planning/PROJECT.md (atualizado [hoje])

**Valor core:** [Valor core atual do PROJECT.md]
**Foco atual:** [Próximo marco ou "Planejando próximo marco"]
```

**Contexto Acumulado:**
- Limpar resumo de decisões (log completo no PROJECT.md)
- Limpar bloqueadores resolvidos
- Manter bloqueadores abertos para próximo marco

</step>

<step name="handle_branches">

Verificar estratégia de branches e oferecer opções de merge.

Usar `init milestone-op` para contexto, ou carregar config diretamente:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair `branching_strategy`, `phase_branch_template`, `milestone_branch_template`, e `commit_docs` do JSON de init.

**Se "none":** Pular para git_tag.

**Para estratégia "phase":**

```bash
BRANCH_PREFIX=$(echo "$PHASE_BRANCH_TEMPLATE" | sed 's/{.*//')
PHASE_BRANCHES=$(git branch --list "${BRANCH_PREFIX}*" 2>/dev/null | sed 's/^\*//' | tr -d ' ')
```

**Para estratégia "milestone":**

```bash
BRANCH_PREFIX=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed 's/{.*//')
MILESTONE_BRANCH=$(git branch --list "${BRANCH_PREFIX}*" 2>/dev/null | sed 's/^\*//' | tr -d ' ' | head -1)
```

**Se nenhum branch encontrado:** Pular para git_tag.

**Se branches existirem:**

```
## Branches Git Detectados

Estratégia de branches: {phase/milestone}
Branches: {lista}

Opções:
1. **Merge para main** — Fazer merge do(s) branch(es) para main
2. **Deletar sem merge** — Já foi feito merge ou não é necessário
3. **Manter branches** — Deixar para tratamento manual
```

conversational prompting com opções: Squash merge (Recomendado), Merge com histórico, Deletar sem merge, Manter branches.

**Squash merge:**

```bash
CURRENT_BRANCH=$(git branch --show-current)
git checkout main

if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  for branch in $PHASE_BRANCHES; do
    git merge --squash "$branch"
    # Remover .planning/ do staging se commit_docs for false
    if [ "$COMMIT_DOCS" = "false" ]; then
      git reset HEAD .planning/ 2>/dev/null || true
    fi
    git commit -m "feat: $branch for v[X.Y]"
  done
fi

if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  git merge --squash "$MILESTONE_BRANCH"
  # Remover .planning/ do staging se commit_docs for false
  if [ "$COMMIT_DOCS" = "false" ]; then
    git reset HEAD .planning/ 2>/dev/null || true
  fi
  git commit -m "feat: $MILESTONE_BRANCH for v[X.Y]"
fi

git checkout "$CURRENT_BRANCH"
```

**Merge com histórico:**

```bash
CURRENT_BRANCH=$(git branch --show-current)
git checkout main

if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  for branch in $PHASE_BRANCHES; do
    git merge --no-ff --no-commit "$branch"
    # Remover .planning/ do staging se commit_docs for false
    if [ "$COMMIT_DOCS" = "false" ]; then
      git reset HEAD .planning/ 2>/dev/null || true
    fi
    git commit -m "Merge branch '$branch' for v[X.Y]"
  done
fi

if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  git merge --no-ff --no-commit "$MILESTONE_BRANCH"
  # Remover .planning/ do staging se commit_docs for false
  if [ "$COMMIT_DOCS" = "false" ]; then
    git reset HEAD .planning/ 2>/dev/null || true
  fi
  git commit -m "Merge branch '$MILESTONE_BRANCH' for v[X.Y]"
fi

git checkout "$CURRENT_BRANCH"
```

**Deletar sem merge:**

```bash
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  for branch in $PHASE_BRANCHES; do
    git branch -d "$branch" 2>/dev/null || git branch -D "$branch"
  done
fi

if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  git branch -d "$MILESTONE_BRANCH" 2>/dev/null || git branch -D "$MILESTONE_BRANCH"
fi
```

**Manter branches:** Reportar "Branches preservados para tratamento manual"

</step>

<step name="git_tag">

Criar tag git:

```bash
git tag -a v[X.Y] -m "v[X.Y] [Nome]

Entregue: [Uma frase]

Realizações principais:
- [Item 1]
- [Item 2]
- [Item 3]

Ver .planning/MILESTONES.md para detalhes completos."
```

Confirmar: "Tag criada: v[X.Y]"

Perguntar: "Enviar tag para remoto? (s/n)"

Se sim:
```bash
git push origin v[X.Y]
```

</step>

<step name="git_commit_milestone">

Commitar conclusão do marco.

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "chore: complete v[X.Y] milestone" --files .planning/milestones/v[X.Y]-ROADMAP.md .planning/milestones/v[X.Y]-REQUIREMENTS.md .planning/milestones/v[X.Y]-MILESTONE-AUDIT.md .planning/MILESTONES.md .planning/PROJECT.md .planning/STATE.md
```

Confirmar: "Commitado: chore: complete v[X.Y] milestone"

</step>

<step name="offer_next">

```
✅ Marco v[X.Y] [Nome] completo

Entregue:
- [N] fases ([M] planos, [P] tarefas)
- [Uma frase do que foi entregue]

Arquivado:
- milestones/v[X.Y]-ROADMAP.md
- milestones/v[X.Y]-REQUIREMENTS.md

Resumo: .planning/MILESTONES.md
Tag: v[X.Y]

---

## ▶ Próximo Passo

**Iniciar Próximo Marco** — questionamento → pesquisa → requisitos → roteiro

`/gsd-novo-marco`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---
```

</step>

</process>

<milestone_naming>

**Convenções de versão:**
- **v1.0** — MVP Inicial
- **v1.1, v1.2** — Atualizações menores, novas funcionalidades, correções
- **v2.0, v3.0** — Reescritas maiores, mudanças breaking, nova direção

**Nomes:** 1-2 palavras curtas (v1.0 MVP, v1.1 Segurança, v1.2 Performance, v2.0 Redesign).

</milestone_naming>

<what_qualifies>

**Criar marcos para:** Release inicial, releases públicos, conjuntos de funcionalidades entregues, antes de arquivar planejamento.

**Não criar marcos para:** Cada conclusão de fase (muito granular), trabalho em progresso, iterações internas de dev (a menos que realmente entregue).

Heurística: "Isto está implantado/usável/entregue?" Se sim → marco. Se não → continue trabalhando.

</what_qualifies>

<success_criteria>

Conclusão do marco é bem-sucedida quando:

- [ ] Entrada no MILESTONES.md criada com estatísticas e realizações
- [ ] Revisão completa de evolução do PROJECT.md realizada
- [ ] Todos os requisitos entregues movidos para Validados no PROJECT.md
- [ ] Decisões Chave atualizadas com resultados
- [ ] ROADMAP.md reorganizado com agrupamento de marcos
- [ ] Arquivo de roteiro criado (milestones/v[X.Y]-ROADMAP.md)
- [ ] Arquivo de requisitos criado (milestones/v[X.Y]-REQUIREMENTS.md)
- [ ] REQUIREMENTS.md deletado (novo para próximo marco)
- [ ] STATE.md atualizado com referência de projeto atualizada
- [ ] Tag git criada (v[X.Y])
- [ ] Commit do marco feito (inclui arquivos de arquivo e deleção)
- [ ] Completude de requisitos verificada contra tabela de rastreabilidade do REQUIREMENTS.md
- [ ] Requisitos incompletos exibidos com opções prosseguir/auditar/abortar
- [ ] Lacunas conhecidas registradas no MILESTONES.md se usuário prosseguiu com requisitos incompletos
- [ ] RETROSPECTIVE.md atualizado com seção do marco
- [ ] Tendências entre marcos atualizadas
- [ ] Usuário sabe o próximo passo (/gsd-novo-marco)

</success_criteria>
