<purpose>
Criar arquivos estruturados de handoff `.planning/HANDOFF.json` e `.continue-here.md` para preservar o estado completo do trabalho entre sessões. O JSON fornece estado legível por máquina para `/gsd-retomar-trabalho`; o markdown fornece contexto legível por humanos.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="detect">
Encontre o diretório da fase atual a partir dos arquivos mais recentemente modificados:

```bash
# Encontrar diretório da fase mais recente com trabalho
ls -lt .planning/phases/*/PLAN.md 2>/dev/null | head -1 | grep -oP 'phases/\K[^/]+'
```

Se nenhuma fase ativa for detectada, pergunte ao usuário em qual fase está pausando o trabalho.
</step>

<step name="gather">
**Coletar estado completo para handoff:**

1. **Posição atual**: Qual fase, qual plano, qual tarefa
2. **Trabalho concluído**: O que foi feito nesta sessão
3. **Trabalho restante**: O que falta no plano/fase atual
4. **Decisões tomadas**: Decisões-chave e justificativa
5. **Bloqueios/problemas**: Qualquer coisa travada
6. **Ações humanas pendentes**: Coisas que precisam de intervenção manual (setup de MCP, chaves de API, aprovações, testes manuais)
7. **Processos em segundo plano**: Quaisquer servidores/watchers em execução que faziam parte do workflow
8. **Arquivos modificados**: O que mudou mas não foi commitado

Peça esclarecimentos ao usuário se necessário via perguntas conversacionais.

**Também inspecione arquivos SUMMARY.md para conclusões falsas:**
```bash
# Verificar conteúdo placeholder em sumários existentes
grep -l "To be filled\|placeholder\|TBD" .planning/phases/*/*.md 2>/dev/null
```
Relate quaisquer sumários com conteúdo placeholder como itens incompletos.
</step>

<step name="write_structured">
**Escrever handoff estruturado em `.planning/HANDOFF.json`:**

```bash
timestamp=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" current-timestamp full --raw)
```

```json
{
  "version": "1.0",
  "timestamp": "{timestamp}",
  "phase": "{numero_fase}",
  "phase_name": "{nome_fase}",
  "phase_dir": "{dir_fase}",
  "plan": {numero_plano_atual},
  "task": {numero_tarefa_atual},
  "total_tasks": {contagem_total_tarefas},
  "status": "paused",
  "completed_tasks": [
    {"id": 1, "name": "{nome_tarefa}", "status": "done", "commit": "{hash_curto}"},
    {"id": 2, "name": "{nome_tarefa}", "status": "done", "commit": "{hash_curto}"},
    {"id": 3, "name": "{nome_tarefa}", "status": "in_progress", "progress": "{o_que_foi_feito}"}
  ],
  "remaining_tasks": [
    {"id": 4, "name": "{nome_tarefa}", "status": "not_started"},
    {"id": 5, "name": "{nome_tarefa}", "status": "not_started"}
  ],
  "blockers": [
    {"description": "{bloqueio}", "type": "technical|human_action|external", "workaround": "{se houver}"}
  ],
  "human_actions_pending": [
    {"action": "{o que precisa ser feito}", "context": "{por quê}", "blocking": true}
  ],
  "decisions": [
    {"decision": "{o quê}", "rationale": "{por quê}", "phase": "{numero_fase}"}
  ],
  "uncommitted_files": [],
  "next_action": "{ação específica ao retomar}",
  "context_notes": "{estado mental, abordagem, o que estava pensando}"
}
```
</step>

<step name="write">
**Escrever handoff em `.planning/phases/XX-name/.continue-here.md`:**

```markdown
---
phase: XX-name
task: 3
total_tasks: 7
status: in_progress
last_updated: [timestamp de current-timestamp]
---

<current_state>
[Onde exatamente estamos? Contexto imediato]
</current_state>

<completed_work>

- Tarefa 1: [nome] - Concluída
- Tarefa 2: [nome] - Concluída
- Tarefa 3: [nome] - Em progresso, [o que foi feito]
</completed_work>

<remaining_work>

- Tarefa 3: [o que falta]
- Tarefa 4: Não iniciada
- Tarefa 5: Não iniciada
</remaining_work>

<decisions_made>

- Decidiu usar [X] porque [razão]
- Escolheu [abordagem] ao invés de [alternativa] porque [razão]
</decisions_made>

<blockers>
- [Bloqueio 1]: [status/contorno]
</blockers>

<context>
[Estado mental, o que estava pensando, o plano]
</context>

<next_action>
Comece com: [ação específica ao retomar]
</next_action>
```

Seja específico o suficiente para um Claude novo entender imediatamente.

Use `current-timestamp` para o campo last_updated. Você pode usar init todos (que fornece timestamps) ou chamar diretamente:
```bash
timestamp=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" current-timestamp full --raw)
```
</step>

<step name="commit">
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "wip: [nome-fase] pausado na tarefa [X]/[Y]" --files .planning/phases/*/.continue-here.md .planning/HANDOFF.json
```
</step>

<step name="confirm">
```
✓ Handoff criado:
  - .planning/HANDOFF.json (estruturado, legível por máquina)
  - .planning/phases/[XX-name]/.continue-here.md (legível por humanos)

Estado atual:

- Fase: [XX-name]
- Tarefa: [X] de [Y]
- Status: [em_progresso/bloqueado]
- Bloqueios: [contagem] ({contagem de human_actions_pending} precisam de ação humana)
- Commitado como WIP

Para retomar: /gsd-retomar-trabalho

```
</step>

</process>

<success_criteria>
- [ ] .continue-here.md criado no diretório correto da fase
- [ ] Todas as seções preenchidas com conteúdo específico
- [ ] Commitado como WIP
- [ ] Usuário sabe a localização e como retomar
</success_criteria>
