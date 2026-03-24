<purpose>

Inicia um novo ciclo de marco para um projeto existente. Carrega o contexto do projeto, reúne os objetivos do marco (a partir de MILESTONE-CONTEXT.md ou da conversa), atualiza PROJECT.md e STATE.md, opcionalmente executa pesquisa em paralelo, define requisitos com escopo e REQ-IDs, dispara o roteirista para criar o plano de execução em fases e faz commit de todos os artefatos. Equivalente brownfield de new-project.

</purpose>

<required_reading>

Leia (Read) todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

</required_reading>

<process>

## 1. Carregar contexto

Analise `{{GSD_ARGS}}` antes de qualquer outra coisa:
- flag `--reset-phase-numbers` → optar por reiniciar a numeração das fases do roadmap em `1`
- texto restante → usar como nome do marco, se houver

Se a flag estiver ausente, mantenha o comportamento atual de continuar a numeração das fases a partir do marco anterior.

- Read PROJECT.md (projeto existente, requisitos validados, decisões)
- Read MILESTONES.md (o que já foi entregue)
- Read STATE.md (todos pendentes, bloqueios)
- Verificar MILESTONE-CONTEXT.md (de `/gsd-discutir-marco`)

## 2. Reunir objetivos do marco

**Se MILESTONE-CONTEXT.md existir:**
- Usar funcionalidades e escopo da discussão do marco (`/gsd-discutir-marco`)
- Apresentar resumo para confirmação

**Se não houver arquivo de contexto:**
- Apresentar o que foi entregue no último marco
- Perguntar inline (texto livre, SEM conversational prompting): "O que você quer construir a seguir?"
- Aguardar a resposta e então usar conversational prompting para aprofundar detalhes
- Se o usuário escolher "Outro" em algum momento para texto livre, fazer follow-up em texto simples — não outro conversational prompting

## 3. Definir versão do marco

- Extrair a última versão de MILESTONES.md
- Sugerir a próxima versão (v1.0 → v1.1, ou v2.0 para major)
- Confirmar com o usuário

## 3.5. Verificar entendimento do marco

Antes de gravar qualquer arquivo, apresente um resumo do que foi reunido e peça confirmação.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESUMO DO MARCO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Milestone v[X.Y]: [Name]**

**Goal:** [One sentence]

**Target features:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**Key context:** [Restrições, decisões ou notas importantes do questionamento]
```

conversational prompting:
- header: "Confirmar?"
- question: "Isso reflete o que você quer construir neste marco?"
- options:
  - "Está bom" — Prosseguir para gravar PROJECT.md
  - "Ajustar" — Deixe-me corrigir ou acrescentar detalhes

**Se "Ajustar":** Pergunte o que precisa mudar (texto simples, SEM conversational prompting). Incorpore as mudanças e reapresente o resumo. Repita até selecionar "Está bom".

**Se "Está bom":** Prosseguir para o passo 4.

## 4. Atualizar PROJECT.md

Adicionar/atualizar:

```markdown
## Current Milestone: v[X.Y] [Name]

**Goal:** [Uma frase descrevendo o foco do marco]

**Target features:**
- [Feature 1]
- [Feature 2]
- [Feature 3]
```

Atualizar a seção Active requirements e o rodapé "Last updated".

Garantir que a seção `## Evolution` exista em PROJECT.md. Se faltar (projetos criados antes desse recurso), adicioná-la antes do rodapé:

```markdown
## Evolution

Este documento evolui nas transições de fase e nos limites de marco.

**Após cada transição de fase** (via `/gsd-transicao`):
1. Requisitos invalidados? → Mover para Out of Scope com motivo
2. Requisitos validados? → Mover para Validated com referência à fase
3. Novos requisitos surgiram? → Adicionar em Active
4. Decisões a registrar? → Adicionar em Key Decisions
5. "What This Is" ainda correto? → Atualizar se houver deriva

**Após cada marco** (via `/gsd-completar-marco`):
1. Revisão completa de todas as seções
2. Checagem de Core Value — ainda é a prioridade certa?
3. Auditoria de Out of Scope — os motivos ainda são válidos?
4. Atualizar Context com o estado atual
```

## 5. Atualizar STATE.md

```markdown
## Current Position

Phase: Não iniciada (definindo requisitos)
Plan: —
Status: Definindo requisitos
Last activity: [today] — Marco v[X.Y] iniciado
```

Manter a seção Accumulated Context do marco anterior.

## 6. Limpeza e commit

Excluir MILESTONE-CONTEXT.md se existir (consumido).

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: start milestone v[X.Y] [Name]" --files .planning/PROJECT.md .planning/STATE.md
```

## 7. Carregar contexto e resolver modelos

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init new-milestone)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `researcher_model`, `synthesizer_model`, `roadmapper_model`, `commit_docs`, `research_enabled`, `current_milestone`, `project_exists`, `roadmap_exists`, `latest_completed_milestone`, `phase_dir_count`, `phase_archive_path`.

## 7.5 Segurança do reset de fase (somente com `--reset-phase-numbers`)

Se `--reset-phase-numbers` estiver ativo:

1. Definir número inicial da fase como `1` para o roadmap que virá.
2. Se `phase_dir_count > 0`, arquivar os diretórios de fase antigos antes do roadmapping para que novos diretórios `01-*` / `02-*` não colidam com diretórios obsoletos do marco.

Se `phase_dir_count > 0` e `phase_archive_path` estiver disponível:

```bash
mkdir -p "${phase_archive_path}"
find .planning/phases -mindepth 1 -maxdepth 1 -type d -exec mv {} "${phase_archive_path}/" \;
```

Em seguida, verificar se `.planning/phases/` não contém mais diretórios antigos do marco antes de continuar.

Se `phase_dir_count > 0` mas `phase_archive_path` estiver ausente:
- Parar e explicar que reiniciar a numeração é inseguro sem destino de arquivo de marco concluído.
- Dizer ao usuário para completar/arquivar o marco anterior primeiro e executar de novo `/gsd-novo-marco --reset-phase-numbers ${GSD_WS}`.

## 8. Decisão de pesquisa

Verificar `research_enabled` no JSON de init (carregado do config).

**Se `research_enabled` for `true`:**

conversational prompting: "Pesquisar o ecossistema do domínio para novas funcionalidades antes de definir requisitos?"
- "Pesquisar primeiro (recomendado)" — Descobrir padrões, funcionalidades, arquitetura para capacidades NOVAS
- "Pular pesquisa neste marco" — Ir direto aos requisitos (não altera seu padrão)

**Se `research_enabled` for `false`:**

conversational prompting: "Pesquisar o ecossistema do domínio para novas funcionalidades antes de definir requisitos?"
- "Pular pesquisa (padrão atual)" — Ir direto aos requisitos
- "Pesquisar primeiro" — Descobrir padrões, funcionalidades, arquitetura para capacidades NOVAS

**IMPORTANTE:** NÃO persistir essa escolha em config.json. A configuração `workflow.research` é preferência persistente do usuário que controla o comportamento de plan-phase no projeto. Alterá-la aqui mudaria silenciosamente o `/gsd-planejar-fase` no futuro. Para mudar o padrão, use `/gsd-configuracoes`.

**Se o usuário escolher "Pesquisar primeiro":**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISANDO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Disparando 4 pesquisadores em paralelo...
  → Stack, Features, Architecture, Pitfalls
```

```bash
mkdir -p .planning/research
```

Disparar 4 agentes gsd-pesquisador-projeto em paralelo. Cada um usa este modelo com campos específicos da dimensão:

**Estrutura comum aos 4 pesquisadores:**
```
Task(prompt="
<research_type>Pesquisa de projeto — {DIMENSION} para [novas funcionalidades].</research_type>

<milestone_context>
MARCO SUBSEQUENTE — Adicionando [funcionalidades alvo] ao app existente.
{EXISTING_CONTEXT}
Focar APENAS no necessário para as funcionalidades NOVAS.
</milestone_context>

<question>{QUESTION}</question>

<files_to_read>
- .planning/PROJECT.md (contexto do projeto)
</files_to_read>

<downstream_consumer>{CONSUMER}</downstream_consumer>

<quality_gate>{GATES}</quality_gate>

<output>
Gravar em: .planning/research/{FILE}
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/{FILE}
</output>
", subagent_type="gsd-pesquisador-projeto", model="{researcher_model}", description="Pesquisa {DIMENSION}")
```

**Campos específicos por dimensão:**

| Campo | Stack | Features | Architecture | Pitfalls |
|-------|-------|----------|-------------|----------|
| EXISTING_CONTEXT | Capacidades validadas existentes (NÃO pesquisar de novo): [de PROJECT.md] | Funcionalidades existentes (já construídas): [de PROJECT.md] | Arquitetura existente: [de PROJECT.md ou mapa do codebase] | Foco em erros comuns ao ADICIONAR essas funcionalidades ao sistema existente |
| QUESTION | Que adições/mudanças de stack são necessárias para [new features]? | Como [target features] costumam funcionar? Comportamento esperado? | Como [target features] se integram à arquitetura existente? | Erros comuns ao adicionar [target features] em [domain]? |
| CONSUMER | Bibliotecas específicas com versões para capacidades NOVAS, pontos de integração, o que NÃO adicionar | Table stakes vs diferenciadores vs anti-features, complexidade anotada, dependências do existente | Pontos de integração, novos componentes, mudanças no fluxo de dados, ordem sugerida de construção | Sinais de alerta, estratégia de prevenção, em qual fase tratar |
| GATES | Versões atuais (verificar com Context7), justificativa explica POR QUÊ, integração considerada | Categorias claras, complexidade anotada, dependências identificadas | Pontos de integração identificados, novo vs modificado explícito, ordem de build considera deps | Armadilhas específicas a adicionar essas funcionalidades, armadilhas de integração cobertas, prevenção acionável |
| FILE | STACK.md | FUNCIONALIDADES.md | ARQUITETURA.md | ARMADILHAS.md |

Depois que os 4 terminarem, disparar o sintetizador:

```
Task(prompt="
Sintetizar saídas da pesquisa em RESUMO.md.

<files_to_read>
- .planning/research/STACK.md
- .planning/research/FUNCIONALIDADES.md
- .planning/research/ARQUITETURA.md
- .planning/research/ARMADILHAS.md
</files_to_read>

Gravar em: .planning/research/RESUMO.md
Usar template: D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/pesquisa-projeto/RESUMO.md
Fazer commit após gravar.
", subagent_type="gsd-sintetizador-pesquisa", model="{synthesizer_model}", description="Sintetizar pesquisa")
```

Exibir achados principais de SUMMARY.md:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISA CONCLUÍDA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Adições de stack:** [de RESUMO.md]
**Table stakes de funcionalidade:** [de RESUMO.md]
**Cuidado com:** [de RESUMO.md]
```

**Se "Pular pesquisa":** Continuar para o passo 9.

## 9. Definir requisitos

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DEFININDO REQUISITOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Read PROJECT.md: core value, objetivos do marco atual, requisitos validados (o que existe).

**Se houver pesquisa:** Read FUNCIONALIDADES.md, extrair categorias de funcionalidade.

Apresentar funcionalidades por categoria:
```
## [Categoria 1]
**Table stakes:** Funcionalidade A, Funcionalidade B
**Differentiators:** Funcionalidade C, Funcionalidade D
**Notas da pesquisa:** [notas relevantes]
```

**Se não houver pesquisa:** Reunir requisitos por conversa. Perguntar: "Quais são as principais coisas que os usuários precisam fazer com [novas funcionalidades]?" Esclarecer, aprofundar capacidades relacionadas, agrupar em categorias.

**Escopo de cada categoria** via conversational prompting (multiSelect: true, header máx. 12 caracteres):
- "[Feature 1]" — [descrição breve]
- "[Feature 2]" — [descrição breve]
- "Nada neste marco" — Adiar categoria inteira

Registrar: Selecionado → este marco. Table stakes não selecionados → futuro. Diferenciadores não selecionados → fora do escopo.

**Identificar lacunas** via conversational prompting:
- "Não, a pesquisa já cobriu" — Prosseguir
- "Sim, quero acrescentar" — Registrar acréscimos

**Gerar REQUIREMENTS.md:**
- Requisitos v1 agrupados por categoria (checkboxes, REQ-IDs)
- Future Requirements (adiados)
- Out of Scope (exclusões explícitas com justificativa)
- Seção Traceability (vazia, preenchida pelo roadmap)

**Formato REQ-ID:** `[CATEGORY]-[NUMBER]` (AUTH-01, NOTIF-02). Continuar numeração a partir do existente.

**Critérios de qualidade de requisito:**

Bons requisitos são:
- **Específicos e testáveis:** "O usuário pode redefinir a senha pelo link por e-mail" (não "Tratar redefinição de senha")
- **Centrados no usuário:** "O usuário pode X" (não "O sistema faz Y")
- **Atômicos:** Uma capacidade por requisito (não "O usuário pode fazer login e gerenciar perfil")
- **Independentes:** Dependências mínimas de outros requisitos

Apresentar a lista COMPLETA de requisitos para confirmação:

```
## Milestone v[X.Y] Requirements

### [Categoria 1]
- [ ] **CAT1-01**: O usuário pode fazer X
- [ ] **CAT1-02**: O usuário pode fazer Y

### [Categoria 2]
- [ ] **CAT2-01**: O usuário pode fazer Z

Isso cobre o que você vai construir? (sim / ajustar)
```

Se "ajustar": Voltar ao escopo.

**Commit dos requisitos:**
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: define milestone v[X.Y] requirements" --files .planning/REQUIREMENTS.md
```

## 10. Criar roadmap

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CRIANDO ROADMAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Disparando roteirista...
```

**Número inicial da fase:**
- Se `--reset-phase-numbers` estiver ativo, começar na **Fase 1**
- Caso contrário, continuar a partir do último número de fase do marco anterior (v1.0 terminou na fase 5 → v1.1 começa na fase 6)

```
Task(prompt="
<planning_context>
<files_to_read>
- .planning/PROJECT.md
- .planning/REQUIREMENTS.md
- .planning/research/RESUMO.md (se existir)
- .planning/config.json
- .planning/MILESTONES.md
</files_to_read>
</planning_context>

<instructions>
Criar roadmap para o marco v[X.Y]:
1. Respeitar o modo de numeração escolhido:
   - `--reset-phase-numbers` → começar na Fase 1
   - comportamento padrão → continuar a partir do último número de fase do marco anterior
2. Derivar fases somente dos requisitos DESTE MARCO
3. Mapear cada requisito para exatamente uma fase
4. Derivar 2-5 success criteria por fase (comportamentos observáveis do usuário)
5. Validar cobertura de 100%
6. Gravar arquivos imediatamente (ROADMAP.md, STATE.md, atualizar traceability em REQUIREMENTS.md)
7. Retornar ROADMAP CREATED com resumo

Gravar arquivos primeiro, depois retornar.
</instructions>
", subagent_type="gsd-roteirista", model="{roadmapper_model}", description="Criar roadmap")
```

**Tratar retorno:**

**Se `## ROADMAP BLOCKED`:** Apresentar bloqueio, trabalhar com o usuário, disparar de novo.

**Se `## ROADMAP CREATED`:** Read ROADMAP.md, apresentar inline:

```
## Roadmap proposto

**[N] fases** | **[X] requisitos mapeados** | Tudo coberto ✓

| # | Fase | Objetivo | Requisitos | Success Criteria |
|---|------|----------|------------|------------------|
| [N] | [Name] | [Goal] | [REQ-IDs] | [count] |

### Detalhes da fase

**Fase [N]: [Name]**
Objetivo: [goal]
Requisitos: [REQ-IDs]
Success criteria:
1. [criterion]
2. [criterion]
```

**Pedir aprovação** via conversational prompting:
- "Aprovar" — Fazer commit e continuar
- "Ajustar fases" — Diga o que mudar
- "Revisar arquivo completo" — Mostrar ROADMAP.md bruto

**Se "Ajustar fases":** Obter notas, disparar gsd-roteirista de novo com contexto de revisão, repetir até aprovar.
**Se "Revisar arquivo completo":** Exibir ROADMAP.md bruto e perguntar de novo.

**Commit do roadmap** (após aprovação):
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: create milestone v[X.Y] roadmap ([N] phases)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md
```

## 11. Concluído

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► MARCO INICIALIZADO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Milestone v[X.Y]: [Name]**

| Artefato      | Local                       |
|---------------|-----------------------------|
| Project       | `.planning/PROJECT.md`      |
| Research      | `.planning/research/`       |
| Requirements  | `.planning/REQUIREMENTS.md` |
| Roadmap       | `.planning/ROADMAP.md`      |

**[N] fases** | **[X] requisitos** | Pronto para construir ✓

## ▶ Próximo passo

**Phase [N]: [Phase Name]** — [Goal]

`/gsd-discutir-fase [N] ${GSD_WS}` — reunir contexto e esclarecer abordagem

<sub>`/clear` primeiro → janela de contexto limpa</sub>

Também: `/gsd-planejar-fase [N] ${GSD_WS}` — pular discussão, planejar direto
```

</process>

<success_criteria>
- [ ] PROJECT.md atualizado com a seção Current Milestone
- [ ] STATE.md reiniciado para o novo marco
- [ ] MILESTONE-CONTEXT.md consumido e excluído (se existia)
- [ ] Pesquisa concluída (se selecionada) — 4 agentes em paralelo, com consciência do marco
- [ ] Requisitos reunidos e com escopo por categoria
- [ ] REQUIREMENTS.md criado com REQ-IDs
- [ ] gsd-roteirista disparado com contexto de numeração de fases
- [ ] Arquivos do roadmap gravados na hora (não rascunho)
- [ ] Feedback do usuário incorporado (se houver)
- [ ] Modo de numeração de fases respeitado (continuado ou reiniciado)
- [ ] Todos os commits feitos (se documentos de planejamento forem commitados)
- [ ] Usuário sabe o próximo passo: `/gsd-discutir-fase [N] ${GSD_WS}`

**Commits atômicos:** Cada fase faz commit dos seus artefatos imediatamente.
</success_criteria>
