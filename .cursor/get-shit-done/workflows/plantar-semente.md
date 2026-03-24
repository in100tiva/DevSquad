<purpose>
Capturar uma ideia voltada para o futuro como um arquivo de semente estruturado com condições de ativação.
Sementes aparecem automaticamente durante /gsd-new-milestone quando as condições de ativação correspondem ao escopo do novo milestone.

Sementes são melhores que itens adiados porque:
- Preservam POR QUE a ideia importa (não apenas O QUE)
- Definem QUANDO aparecer (condições de ativação, não varredura manual)
- Rastreiam pistas (referências de código, decisões relacionadas)
- Aparecem automaticamente no momento certo via varredura do new-milestone
</purpose>

<process>

<step name="parse_idea">
Analise `{{GSD_ARGS}}` para o resumo da ideia.

Se vazio, pergunte:
```
Qual é a ideia? (uma frase)
```

Armazene como `$IDEA`.
</step>

<step name="create_seed_dir">
```bash
mkdir -p .planning/seeds
```
</step>

<step name="gather_context">
Faça perguntas focadas para construir uma semente completa:

```
conversational prompting(
  header: "Gatilho",
  question: "Quando esta ideia deve aparecer? (ex.: 'quando adicionarmos contas de usuário', 'próxima versão major', 'quando performance se tornar prioridade')",
  options: []  // formato livre
)
```

Armazene como `$TRIGGER`.

```
conversational prompting(
  header: "Por quê",
  question: "Por que isso importa? Que problema resolve ou que oportunidade cria?",
  options: []
)
```

Armazene como `$WHY`.

```
conversational prompting(
  header: "Escopo",
  question: "Qual o tamanho disto? (estimativa aproximada)",
  options: [
    { label: "Pequeno", description: "Algumas horas — poderia ser uma tarefa rápida" },
    { label: "Médio", description: "Uma ou duas fases — precisa de planejamento" },
    { label: "Grande", description: "Um milestone inteiro — esforço significativo" }
  ]
)
```

Armazene como `$SCOPE`.
</step>

<step name="collect_breadcrumbs">
Busque no codebase referências relevantes:

```bash
# Encontrar arquivos relacionados às palavras-chave da ideia
grep -rl "$KEYWORD" --include="*.ts" --include="*.js" --include="*.md" . 2>/dev/null | head -10
```

Também verifique:
- STATE.md atual para decisões relacionadas
- ROADMAP.md para fases relacionadas
- todos/ para ideias capturadas relacionadas

Armazene caminhos de arquivos relevantes como `$BREADCRUMBS`.
</step>

<step name="generate_seed_id">
```bash
# Encontrar próximo número de semente
EXISTING=$(ls .planning/seeds/SEED-*.md 2>/dev/null | wc -l)
NEXT=$((EXISTING + 1))
PADDED=$(printf "%03d" $NEXT)
```

Gere slug a partir do resumo da ideia.
</step>

<step name="write_seed">
Escreva `.planning/seeds/SEED-{PADDED}-{slug}.md`:

```markdown
---
id: SEED-{PADDED}
status: dormant
planted: {data ISO}
planted_during: {milestone/fase atual do STATE.md}
trigger_when: {$TRIGGER}
scope: {$SCOPE}
---

# SEED-{PADDED}: {$IDEA}

## Por Que Isso Importa

{$WHY}

## Quando Aparecer

**Gatilho:** {$TRIGGER}

Esta semente deve ser apresentada durante `/gsd-new-milestone` quando o escopo
do milestone corresponder a qualquer uma destas condições:
- {condição de gatilho 1}
- {condição de gatilho 2}

## Estimativa de Escopo

**{$SCOPE}** — {elaboração baseada na escolha de escopo}

## Pistas

Código e decisões relacionadas encontradas no codebase atual:

{lista de $BREADCRUMBS com caminhos de arquivo}

## Notas

{qualquer contexto adicional da sessão atual}
```
</step>

<step name="commit_seed">
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: plantar semente — {$IDEA}" --files .planning/seeds/SEED-{PADDED}-{slug}.md
```
</step>

<step name="confirm">
```
✅ Semente plantada: SEED-{PADDED}

"{$IDEA}"
Gatilho: {$TRIGGER}
Escopo: {$SCOPE}
Arquivo: .planning/seeds/SEED-{PADDED}-{slug}.md

Esta semente aparecerá automaticamente quando você executar /gsd-new-milestone
e o escopo do milestone corresponder à condição de gatilho.
```
</step>

</process>

<success_criteria>
- [ ] Arquivo de semente criado em .planning/seeds/
- [ ] Frontmatter inclui status, gatilho, escopo
- [ ] Pistas coletadas do codebase
- [ ] Commitado no git
- [ ] Confirmação mostrada ao usuário com info do gatilho
</success_criteria>
