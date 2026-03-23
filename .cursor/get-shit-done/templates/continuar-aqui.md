# Template Continuar-Aqui

Copie e preencha esta estrutura para `.planning/phases/XX-nome/.continue-here.md`:

```yaml
---
phase: XX-nome
task: 3
total_tasks: 7
status: in_progress
last_updated: 2025-01-15T14:30:00Z
---
```

```markdown
<current_state>
[Onde exatamente estamos? Qual é o contexto imediato?]
</current_state>

<completed_work>
[O que foi feito nesta sessão - seja específico]

- Tarefa 1: [nome] - Concluída
- Tarefa 2: [nome] - Concluída
- Tarefa 3: [nome] - Em andamento, [o que já foi feito]
</completed_work>

<remaining_work>
[O que falta nesta fase]

- Tarefa 3: [nome] - [o que falta fazer]
- Tarefa 4: [nome] - Não iniciada
- Tarefa 5: [nome] - Não iniciada
</remaining_work>

<decisions_made>
[Decisões-chave e por quê - para que a próxima sessão não rediscuta]

- Decidido usar [X] porque [razão]
- Escolhido [abordagem] em vez de [alternativa] porque [razão]
</decisions_made>

<blockers>
[Qualquer coisa travada ou dependendo de fatores externos]

- [Bloqueio 1]: [status/solução alternativa]
</blockers>

<context>
[Estado mental, "vibe", qualquer coisa que ajude a retomar suavemente]

[O que você estava pensando? Qual era o plano?
Este é o contexto para "retomar exatamente de onde parou".]
</context>

<next_action>
[A primeira coisa a fazer ao retomar]

Começar com: [ação específica]
</next_action>
```

<yaml_fields>
Campos YAML obrigatórios no frontmatter:

- `phase`: Nome do diretório (ex.: `02-autenticacao`)
- `task`: Número da tarefa atual
- `total_tasks`: Quantas tarefas na fase
- `status`: `in_progress`, `blocked`, `almost_done`
- `last_updated`: Timestamp ISO
</yaml_fields>

<guidelines>
- Seja específico o suficiente para que uma nova instância do Claude entenda imediatamente
- Inclua POR QUE as decisões foram tomadas, não apenas o quê
- O `<next_action>` deve ser executável sem precisar ler mais nada
- Este arquivo é EXCLUÍDO após retomada - não é armazenamento permanente
</guidelines>
</output>
