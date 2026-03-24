<trigger>
Use este workflow quando:
- Iniciando uma nova sessão em um projeto existente
- Usuário diz "continuar", "o que vem a seguir", "onde paramos", "retomar"
- Qualquer operação de planejamento quando .planning/ já existe
- Usuário retorna após tempo longe do projeto
</trigger>

<purpose>
Restaurar instantaneamente o contexto completo do projeto para que "Onde paramos?" tenha uma resposta imediata e completa.
</purpose>

<required_reading>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/formato-continuacao.md
</required_reading>

<process>

<step name="initialize">
Carregar todo o contexto em uma chamada:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init resume)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar JSON para: `state_exists`, `roadmap_exists`, `project_exists`, `planning_exists`, `has_interrupted_agent`, `interrupted_agent_id`, `commit_docs`.

**Se `state_exists` for true:** Prossiga para load_state
**Se `state_exists` for false mas `roadmap_exists` ou `project_exists` for true:** Ofereça reconstruir STATE.md
**Se `planning_exists` for false:** Este é um novo projeto - direcione para /gsd-new-project
</step>

<step name="load_state">

Ler e analisar STATE.md, depois PROJECT.md:

```bash
cat .planning/STATE.md
cat .planning/PROJECT.md
```

**De STATE.md extrair:**

- **Referência do Projeto**: Valor central e foco atual
- **Posição Atual**: Fase X de Y, Plano A de B, Status
- **Progresso**: Barra de progresso visual
- **Decisões Recentes**: Decisões-chave que afetam o trabalho atual
- **Todos Pendentes**: Ideias capturadas durante sessões
- **Bloqueios/Preocupações**: Problemas herdados
- **Continuidade de Sessão**: Onde paramos, arquivos de retomada

**De PROJECT.md extrair:**

- **O Que É Isto**: Descrição precisa atual
- **Requisitos**: Validados, Ativos, Fora do Escopo
- **Decisões-Chave**: Log completo de decisões com resultados
- **Restrições**: Limites rígidos de implementação

</step>

<step name="check_incomplete_work">
Procurar trabalho incompleto que precisa de atenção:

```bash
# Verificar handoff estruturado (preferido — legível por máquina)
cat .planning/HANDOFF.json 2>/dev/null

# Verificar arquivos continue-here (retomada no meio do plano)
ls .planning/phases/*/.continue-here*.md 2>/dev/null

# Verificar planos sem sumários (execução incompleta)
for plan in .planning/phases/*/*-PLAN.md; do
  summary="${plan/PLAN/SUMMARY}"
  [ ! -f "$summary" ] && echo "Incompleto: $plan"
done 2>/dev/null

# Verificar agentes interrompidos (usar has_interrupted_agent e interrupted_agent_id do init)
if [ "$has_interrupted_agent" = "true" ]; then
  echo "Agente interrompido: $interrupted_agent_id"
fi
```

**Se HANDOFF.json existir:**

- Esta é a fonte primária de retomada — dados estruturados de `/gsd-pause-work`
- Analisar `status`, `phase`, `plan`, `task`, `total_tasks`, `next_action`
- Verificar `blockers` e `human_actions_pending` — mostrar imediatamente
- Verificar `completed_tasks` para itens `in_progress` — estes precisam de atenção primeiro
- Validar `uncommitted_files` contra `git status` — sinalizar divergências
- Usar `context_notes` para restaurar modelo mental
- Sinalizar: "Handoff estruturado encontrado — retomando da tarefa {task}/{total_tasks}"
- **Após retomada bem-sucedida, deletar HANDOFF.json** (é um artefato de uso único)

**Se arquivo .continue-here existir (fallback):**

- Este é um ponto de retomada no meio do plano
- Ler o arquivo para contexto específico de retomada
- Sinalizar: "Checkpoint encontrado no meio do plano"

**Se PLAN sem SUMMARY existir:**

- A execução foi iniciada mas não concluída
- Sinalizar: "Execução de plano incompleta encontrada"

**Se agente interrompido encontrado:**

- Subagente foi criado mas a sessão terminou antes da conclusão
- Ler agent-history.json para detalhes da tarefa
- Sinalizar: "Agente interrompido encontrado"
  </step>

<step name="present_status">
Apresentar status completo do projeto ao usuário:

```
╔══════════════════════════════════════════════════════════════╗
║  STATUS DO PROJETO                                            ║
╠══════════════════════════════════════════════════════════════╣
║  Construindo: [uma linha de PROJECT.md "O Que É Isto"]        ║
║                                                               ║
║  Fase: [X] de [Y] - [Nome da fase]                           ║
║  Plano: [A] de [B] - [Status]                                ║
║  Progresso: [██████░░░░] XX%                                  ║
║                                                               ║
║  Última atividade: [data] - [o que aconteceu]                 ║
╚══════════════════════════════════════════════════════════════╝

[Se trabalho incompleto encontrado:]
⚠️  Trabalho incompleto detectado:
    - [arquivo .continue-here ou plano incompleto]

[Se agente interrompido encontrado:]
⚠️  Agente interrompido detectado:
    ID do Agente: [id]
    Tarefa: [descrição da tarefa de agent-history.json]
    Interrompido: [timestamp]

    Retomar com: ferramenta Task (parâmetro resume com ID do agente)

[Se todos pendentes existirem:]
📋 [N] todos pendentes — /gsd-check-todos para revisar

[Se bloqueios existirem:]
⚠️  Preocupações herdadas:
    - [bloqueio 1]
    - [bloqueio 2]

[Se alinhamento não estiver ✓:]
⚠️  Alinhamento do brief: [status] - [avaliação]
```

</step>

<step name="determine_next_action">
Com base no estado do projeto, determinar a ação mais lógica:

**Se agente interrompido existir:**
→ Primário: Retomar agente interrompido (ferramenta Task com parâmetro resume)
→ Opção: Começar do zero (abandonar trabalho do agente)

**Se HANDOFF.json existir:**
→ Primário: Retomar do handoff estruturado (maior prioridade — contexto específico de tarefa/bloqueio)
→ Opção: Descartar handoff e reavaliar a partir dos arquivos

**Se arquivo .continue-here existir:**
→ Fallback: Retomar do checkpoint
→ Opção: Começar do zero no plano atual

**Se plano incompleto (PLAN sem SUMMARY):**
→ Primário: Completar o plano incompleto
→ Opção: Abandonar e seguir em frente

**Se fase em progresso, todos os planos completos:**
→ Primário: Avançar para próxima fase (via workflow interno de transição)
→ Opção: Revisar trabalho concluído

**Se fase pronta para planejar:**
→ Verificar se CONTEXT.md existe para esta fase:

- Se CONTEXT.md ausente:
  → Primário: Discutir visão da fase (como o usuário imagina funcionando)
  → Secundário: Planejar diretamente (pular coleta de contexto)
- Se CONTEXT.md existir:
  → Primário: Planejar a fase
  → Opção: Revisar roadmap

**Se fase pronta para executar:**
→ Primário: Executar próximo plano
→ Opção: Revisar o plano primeiro
</step>

<step name="offer_options">
Apresentar opções contextuais baseadas no estado do projeto:

```
O que você gostaria de fazer?

[Ação primária baseada no estado - ex.:]
1. Retomar agente interrompido [se agente interrompido encontrado]
   OU
1. Executar fase (/gsd-execute-phase {fase} ${GSD_WS})
   OU
1. Discutir contexto da Fase 3 (/gsd-discuss-phase 3 ${GSD_WS}) [se CONTEXT.md ausente]
   OU
1. Planejar Fase 3 (/gsd-plan-phase 3 ${GSD_WS}) [se CONTEXT.md existe ou opção de discussão recusada]

[Opções secundárias:]
2. Revisar status da fase atual
3. Verificar todos pendentes ([N] pendentes)
4. Revisar alinhamento do brief
5. Outra coisa
```

**Nota:** Ao oferecer planejamento de fase, verifique a existência de CONTEXT.md primeiro:

```bash
ls .planning/phases/XX-name/*-CONTEXT.md 2>/dev/null
```

Se ausente, sugira discuss-phase antes de plan. Se existir, ofereça plan diretamente.

Aguarde a seleção do usuário.
</step>

<step name="route_to_workflow">
Com base na seleção do usuário, direcione para o workflow apropriado:

- **Executar plano** → Mostrar comando para o usuário executar após limpar:
  ```
  ---

  ## ▶ Próximo

  **{fase}-{plano}: [Nome do Plano]** — [objetivo do PLAN.md]

  `/gsd-execute-phase {fase} ${GSD_WS}`

  <sub>`/clear` primeiro → janela de contexto limpa</sub>

  ---
  ```
- **Planejar fase** → Mostrar comando para o usuário executar após limpar:
  ```
  ---

  ## ▶ Próximo

  **Fase [N]: [Nome]** — [Objetivo do ROADMAP.md]

  `/gsd-plan-phase [numero-fase] ${GSD_WS}`

  <sub>`/clear` primeiro → janela de contexto limpa</sub>

  ---

  **Também disponível:**
  - `/gsd-discuss-phase [N] ${GSD_WS}` — coletar contexto primeiro
  - `/gsd-research-phase [N] ${GSD_WS}` — investigar incógnitas

  ---
  ```
- **Avançar para próxima fase** → ./transicao.md (workflow interno, invocado inline — NÃO é um comando do usuário)
- **Verificar todos** → Ler .planning/todos/pending/, apresentar resumo
- **Revisar alinhamento** → Ler PROJECT.md, comparar com estado atual
- **Outra coisa** → Perguntar o que precisam
</step>

<step name="update_session">
Antes de prosseguir para o workflow direcionado, atualizar continuidade da sessão:

Atualizar STATE.md:

```markdown
## Continuidade da Sessão

Última sessão: [agora]
Parou em: Sessão retomada, prosseguindo para [ação]
Arquivo de retomada: [atualizado se aplicável]
```

Isto garante que se a sessão terminar inesperadamente, a próxima retomada saiba o estado.
</step>

</process>

<reconstruction>
Se STATE.md estiver ausente mas outros artefatos existirem:

"STATE.md ausente. Reconstruindo a partir dos artefatos..."

1. Ler PROJECT.md → Extrair "O Que É Isto" e Valor Central
2. Ler ROADMAP.md → Determinar fases, encontrar posição atual
3. Escanear arquivos \*-SUMMARY.md → Extrair decisões, preocupações
4. Contar todos pendentes em .planning/todos/pending/
5. Verificar arquivos .continue-here → Continuidade de sessão

Reconstruir e escrever STATE.md, depois prosseguir normalmente.

Isto lida com casos onde:

- Projeto é anterior à introdução do STATE.md
- Arquivo foi deletado acidentalmente
- Clonando repo sem estado completo de .planning/
  </reconstruction>

<quick_resume>
Se o usuário disser "continuar" ou "vamos":
- Carregar estado silenciosamente
- Determinar ação primária
- Executar imediatamente sem apresentar opções

"Continuando de [estado]... [ação]"
</quick_resume>

<success_criteria>
Retomada está completa quando:

- [ ] STATE.md carregado (ou reconstruído)
- [ ] Trabalho incompleto detectado e sinalizado
- [ ] Status claro apresentado ao usuário
- [ ] Próximas ações contextuais oferecidas
- [ ] Usuário sabe exatamente onde o projeto está
- [ ] Continuidade da sessão atualizada
      </success_criteria>
