# Workflow de Resumo do Marco

Gerar um resumo abrangente e amigável do projeto a partir de artefatos de marcos concluídos.
Projetado para onboarding de equipe — um novo contribuidor pode ler a saída e entender o projeto inteiro.

---

## Passo 1: Resolver Versão

```bash
VERSION="{{GSD_ARGS}}"
```

Se `{{GSD_ARGS}}` estiver vazio:
1. Verificar `.planning/STATE.md` para versão do marco atual
2. Verificar `.planning/milestones/` para a última versão arquivada
3. Se nenhum encontrado, verificar se `.planning/ROADMAP.md` existe (projeto pode estar no meio do marco)
4. Se nada encontrado: erro "Nenhum marco encontrado. Execute /gsd-new-project ou /gsd-new-milestone primeiro."

Definir `VERSION` para a versão resolvida (ex., "1.0").

## Passo 2: Localizar Artefatos

Determinar se o marco está **arquivado** ou **atual**:

**Marco arquivado** (`.planning/milestones/v${VERSION}-ROADMAP.md` existe):
```
ROADMAP_PATH=".planning/milestones/v${VERSION}-ROADMAP.md"
REQUIREMENTS_PATH=".planning/milestones/v${VERSION}-REQUIREMENTS.md"
AUDIT_PATH=".planning/milestones/v${VERSION}-MILESTONE-AUDIT.md"
```

**Marco atual/em andamento** (sem arquivo ainda):
```
ROADMAP_PATH=".planning/ROADMAP.md"
REQUIREMENTS_PATH=".planning/REQUIREMENTS.md"
AUDIT_PATH=".planning/v${VERSION}-MILESTONE-AUDIT.md"
```

Nota: O arquivo de auditoria move para `.planning/milestones/` no arquivamento (conforme workflow `completar-marco`). Verificar ambos os locais como fallback.

**Sempre disponível:**
```
PROJECT_PATH=".planning/PROJECT.md"
RETRO_PATH=".planning/RETROSPECTIVE.md"
STATE_PATH=".planning/STATE.md"
```

Ler todos os arquivos que existirem. Arquivos faltando são aceitáveis — o resumo se adapta ao que está disponível.

## Passo 3: Descobrir Artefatos de Fase

Encontrar todos os diretórios de fase:

```bash
gsd-tools.cjs init progress
```

Isto retorna metadados de fase. Para cada fase no escopo do marco:

- Ler `{phase_dir}/{padded}-SUMMARY.md` se existir — extrair `one_liner`, `accomplishments`, `decisions`
- Ler `{phase_dir}/{padded}-VERIFICATION.md` se existir — extrair status, lacunas, itens adiados
- Ler `{phase_dir}/{padded}-CONTEXT.md` se existir — extrair decisões chave da seção `<decisions>`
- Ler `{phase_dir}/{padded}-RESEARCH.md` se existir — anotar o que foi pesquisado

Rastrear quais fases têm quais artefatos.

**Se nenhum diretório de fase existir** (marco vazio ou estado pré-construção): pular para Passo 5 e gerar um resumo mínimo anotando "Nenhuma fase foi executada ainda." Não dar erro — o resumo ainda deve capturar conteúdo do PROJECT.md e ROADMAP.md.

## Passo 4: Coletar Estatísticas Git

Tentar cada método em ordem até um ter sucesso:

**Método 1 — Marco com tag** (verificar primeiro):
```bash
git tag -l "v${VERSION}" | head -1
```
Se a tag existir:
```bash
git log v${VERSION} --oneline | wc -l
git diff --stat $(git log --format=%H --reverse v${VERSION} | head -1)..v${VERSION}
```

**Método 2 — Intervalo de datas do STATE.md** (se sem tag):
Ler STATE.md e extrair `started_at` ou data de sessão mais antiga. Usar como limite `--since`:
```bash
git log --oneline --since="<started_at_date>" | wc -l
```

**Método 3 — Commit mais antigo de fase** (se STATE.md não tem data):
Encontrar o commit mais antigo de `.planning/phases/`:
```bash
git log --oneline --diff-filter=A -- ".planning/phases/" | tail -1
```
Usar a data desse commit como limite de início.

**Método 4 — Pular estatísticas** (se nenhum dos anteriores funcionar):
Reportar "Estatísticas git indisponíveis — nenhuma tag ou intervalo de datas pôde ser determinado." Não é um erro — o resumo continua sem a seção de Estatísticas.

Extrair (quando disponível):
- Total de commits no marco
- Arquivos alterados, inserções, deleções
- Cronograma (data início → data fim)
- Contribuidores (dos autores do git log)

## Passo 5: Gerar Documento de Resumo

Escrever em `.planning/reports/MILESTONE_SUMMARY-v${VERSION}.md`:

```markdown
# Marco v{VERSION} — Resumo do Projeto

**Gerado:** {data}
**Propósito:** Onboarding de equipe e revisão do projeto

---

## 1. Visão Geral do Projeto

{Do PROJECT.md: "O Que É Isto", proposta de valor core, usuários alvo}
{Se no meio do marco: anotar quais fases estão completas vs em andamento}

## 2. Arquitetura e Decisões Técnicas

{Dos arquivos CONTEXT.md entre fases: escolhas técnicas chave}
{Das decisões do SUMMARY.md: padrões, bibliotecas, frameworks escolhidos}
{Do PROJECT.md: stack tecnológica se documentada}

Apresentar como lista com marcadores de decisões com breve justificativa:
- **Decisão:** {o que foi escolhido}
  - **Por quê:** {justificativa do CONTEXT.md}
  - **Fase:** {qual fase tomou esta decisão}

## 3. Fases Entregues

| Fase | Nome | Status | Resumo |
|------|------|--------|--------|
{Para cada fase: número, nome, status (completa/em andamento/planejada), resumo do SUMMARY.md}

## 4. Cobertura de Requisitos

{Do REQUIREMENTS.md: listar cada requisito com status}
- ✅ {Requisito atendido}
- ⚠️ {Requisito parcialmente atendido — anotar lacuna}
- ❌ {Requisito não atendido — anotar razão}

{Se MILESTONE-AUDIT.md existir: incluir veredito da auditoria}

## 5. Log de Decisões Chave

{Agregar de todas as seções <decisions> dos CONTEXT.md}
{Cada decisão com: ID, descrição, fase, justificativa}

## 6. Dívida Técnica e Itens Adiados

{Dos arquivos VERIFICATION.md: lacunas encontradas, anti-padrões anotados}
{Do RETROSPECTIVE.md: lições aprendidas, o que melhorar}
{Das seções <deferred> dos CONTEXT.md: ideias estacionadas para depois}

## 7. Começando

{Pontos de entrada para novos contribuidores:}
- **Executar o projeto:** {do PROJECT.md ou SUMMARY.md}
- **Diretórios chave:** {da estrutura do codebase}
- **Testes:** {comando de teste do PROJECT.md ou .cursor/rules/}
- **Por onde começar:** {pontos de entrada principais, módulos core}

---

## Estatísticas

- **Cronograma:** {início} → {fim} ({duração})
- **Fases:** {completas} / {total}
- **Commits:** {contagem}
- **Arquivos alterados:** {contagem} (+{inserções} / -{deleções})
- **Contribuidores:** {lista}
```

## Passo 6: Escrever e Commitar

**Guarda de sobrescrita:** Se `.planning/reports/MILESTONE_SUMMARY-v${VERSION}.md` já existir, perguntar ao usuário:
> "Um resumo de marco para v{VERSION} já existe. Sobrescrever, ou ver o existente?"
Se "ver": exibir arquivo existente e pular para Passo 8 (modo interativo). Se "sobrescrever": prosseguir.

Criar o diretório de relatórios se necessário:
```bash
mkdir -p .planning/reports
```

Escrever o resumo, então commitar:
```bash
gsd-tools.cjs commit "docs(v${VERSION}): generate milestone summary for onboarding" \
  --files ".planning/reports/MILESTONE_SUMMARY-v${VERSION}.md"
```

## Passo 7: Apresentar Resumo

Exibir o documento de resumo completo inline.

## Passo 8: Oferecer Modo Interativo

Após apresentar o resumo:

> "Resumo escrito em `.planning/reports/MILESTONE_SUMMARY-v{VERSION}.md`.
>
> Tenho contexto completo dos artefatos de construção. Quer perguntar algo sobre o projeto?
> Decisões de arquitetura, fases específicas, requisitos, dívida técnica — pergunte à vontade."

Se o usuário fizer perguntas:
- Responder dos artefatos já carregados (CONTEXT.md, SUMMARY.md, VERIFICATION.md, etc.)
- Referenciar arquivos e decisões específicos
- Manter-se fundamentado no que foi realmente construído (não especulação)

Se o usuário terminar:
- Sugerir próximos passos: `/gsd-new-milestone`, `/gsd-progress`, ou compartilhar o resumo com a equipe

## Passo 9: Atualizar STATE.md

```bash
gsd-tools.cjs state record-session \
  --stopped-at "Resumo do marco v${VERSION} gerado" \
  --resume-file ".planning/reports/MILESTONE_SUMMARY-v${VERSION}.md"
```
