<internal_workflow>

**Este é um workflow INTERNO — NÃO é um comando do usuário.**

Não existe comando `/gsd-transition`. Este workflow é invocado automaticamente por
`execute-phase` durante auto-avanço, ou inline pelo orquestrador após verificação de fase.
Usuários nunca devem ser instruídos a executar `/gsd-transition`.

**Comandos válidos do usuário para progressão de fase:**
- `/gsd-discuss-phase {N}` — discutir uma fase antes de planejar
- `/gsd-plan-phase {N}` — planejar uma fase
- `/gsd-execute-phase {N}` — executar uma fase
- `/gsd-progress` — ver progresso do roadmap

</internal_workflow>

<required_reading>

**Leia estes arquivos AGORA:**

1. `.planning/STATE.md`
2. `.planning/PROJECT.md`
3. `.planning/ROADMAP.md`
4. Arquivos de plano da fase atual (`*-PLAN.md`)
5. Arquivos de sumário da fase atual (`*-SUMMARY.md`)

</required_reading>

<purpose>

Marcar fase atual como completa e avançar para a próxima. Este é o ponto natural onde rastreamento de progresso e evolução do PROJECT.md acontecem.

"Planejar próxima fase" = "fase atual está concluída"

</purpose>

<process>

<step name="load_project_state" priority="first">

Antes da transição, ler estado do projeto:

```bash
cat .planning/STATE.md 2>/dev/null
cat .planning/PROJECT.md 2>/dev/null
```

Analisar posição atual para verificar que estamos fazendo transição da fase correta.
Anotar contexto acumulado que pode precisar de atualização após transição.

</step>

<step name="verify_completion">

Verificar que a fase atual tem todos os sumários de plano:

```bash
ls .planning/phases/XX-current/*-PLAN.md 2>/dev/null | sort
ls .planning/phases/XX-current/*-SUMMARY.md 2>/dev/null | sort
```

**Lógica de verificação:**

- Contar arquivos PLAN
- Contar arquivos SUMMARY
- Se contagens batem: todos os planos completos
- Se contagens não batem: incompleto

<config-check>

```bash
cat .planning/config.json 2>/dev/null
```

</config-check>

**Verificar débito de verificação nesta fase:**

```bash
# Contar itens pendentes na fase atual
OUTSTANDING=""
for f in .planning/phases/XX-current/*-UAT.md .planning/phases/XX-current/*-VERIFICATION.md; do
  [ -f "$f" ] || continue
  grep -q "result: pending\|result: blocked\|status: partial\|status: human_needed\|status: diagnosed" "$f" && OUTSTANDING="$OUTSTANDING\n$(basename $f)"
done
```

**Se OUTSTANDING não estiver vazio:**

Adicionar à mensagem de confirmação de conclusão (independentemente do modo):

```
Itens de verificação pendentes nesta fase:
{listar nomes de arquivos}

Estes serão carregados como débito. Revisar: `/gsd-audit-uat`
```

Isto NÃO bloqueia transição — garante que o usuário veja o débito antes de confirmar.

**Se todos os planos estão completos:**

<if mode="yolo">

```
⚡ Auto-aprovado: Transição Fase [X] → Fase [X+1]
Fase [X] completa — todos os [Y] planos finalizados.

Prosseguindo para marcar como concluída e avançar...
```

Prosseguir diretamente para o passo cleanup_handoff.

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

Perguntar: "Fase [X] completa — todos os [Y] planos finalizados. Pronto para marcar como concluída e avançar para Fase [X+1]?"

Aguardar confirmação antes de prosseguir.

</if>

**Se planos incompletos:**

**RAIL DE SEGURANÇA: always_confirm_destructive se aplica aqui.**
Pular planos incompletos é destrutivo — SEMPRE solicitar confirmação independentemente do modo.

Apresentar:

```
Fase [X] tem planos incompletos:
- {fase}-01-SUMMARY.md ✓ Completo
- {fase}-02-SUMMARY.md ✗ Ausente
- {fase}-03-SUMMARY.md ✗ Ausente

⚠️ Rail de segurança: Pular planos requer confirmação (ação destrutiva)

Opções:
1. Continuar fase atual (executar planos restantes)
2. Marcar como completa mesmo assim (pular planos restantes)
3. Revisar o que falta
```

Aguardar decisão do usuário.

</step>

<step name="cleanup_handoff">

Verificar handoffs pendentes:

```bash
ls .planning/phases/XX-current/.continue-here*.md 2>/dev/null
```

Se encontrados, deletá-los — fase está completa, handoffs são obsoletos.

</step>

<step name="update_roadmap_and_state">

**Delegar atualizações de ROADMAP.md e STATE.md para gsd-tools:**

```bash
TRANSITION=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase complete "${current_phase}")
```

O CLI lida com:
- Marcar o checkbox da fase como `[x]` completo com data de hoje
- Atualizar contagem de planos para final (ex.: "3/3 planos completos")
- Atualizar a tabela de Progresso (Status → Completo, adicionando data)
- Avançar STATE.md para próxima fase (Fase Atual, Status → Pronta para planejar, Plano Atual → Não iniciado)
- Detectar se esta é a última fase do milestone

Extrair do resultado: `completed_phase`, `plans_executed`, `next_phase`, `next_phase_name`, `is_last_phase`.

</step>

<step name="archive_prompts">

Se prompts foram gerados para a fase, eles permanecem no lugar.
O padrão de subpasta `completed/` do create-meta-prompts lida com o arquivamento.

</step>

<step name="evolve_project">

Evoluir PROJECT.md para refletir aprendizados da fase concluída.

**Ler sumários da fase:**

```bash
cat .planning/phases/XX-current/*-SUMMARY.md
```

**Avaliar mudanças de requisitos:**

1. **Requisitos validados?**
   - Algum requisito Ativo entregue nesta fase?
   - Mover para Validado com referência de fase: `- ✓ [Requisito] — Fase X`

2. **Requisitos invalidados?**
   - Algum requisito Ativo descoberto como desnecessário ou errado?
   - Mover para Fora do Escopo com razão: `- [Requisito] — [por que invalidado]`

3. **Requisitos emergentes?**
   - Algum novo requisito descoberto durante a construção?
   - Adicionar a Ativos: `- [ ] [Novo requisito]`

4. **Decisões a registrar?**
   - Extrair decisões dos arquivos SUMMARY.md
   - Adicionar à tabela Decisões-Chave com resultado se conhecido

5. **"O Que É Isto" ainda preciso?**
   - Se o produto mudou significativamente, atualizar a descrição
   - Manter atual e preciso

**Atualizar PROJECT.md:**

Fazer as edições inline. Atualizar rodapé "Última atualização":

```markdown
---
*Última atualização: [data] após Fase [X]*
```

**Exemplo de evolução:**

Antes:

```markdown
### Ativos

- [ ] Autenticação JWT
- [ ] Sincronização em tempo real < 500ms
- [ ] Modo offline

### Fora do Escopo

- OAuth2 — complexidade desnecessária para v1
```

Depois (Fase 2 entregou autenticação JWT, descobriu necessidade de rate limiting):

```markdown
### Validados

- ✓ Autenticação JWT — Fase 2

### Ativos

- [ ] Sincronização em tempo real < 500ms
- [ ] Modo offline
- [ ] Rate limiting no endpoint de sincronização

### Fora do Escopo

- OAuth2 — complexidade desnecessária para v1
```

**Passo completo quando:**

- [ ] Sumários da fase revisados para aprendizados
- [ ] Requisitos validados movidos de Ativos
- [ ] Requisitos invalidados movidos para Fora do Escopo com razão
- [ ] Requisitos emergentes adicionados a Ativos
- [ ] Novas decisões registradas com justificativa
- [ ] "O Que É Isto" atualizado se produto mudou
- [ ] Rodapé "Última atualização" reflete esta transição

</step>

<step name="update_current_position_after_transition">

**Nota:** Atualizações básicas de posição (Fase Atual, Status, Plano Atual, Última Atividade) já foram tratadas por `gsd-tools phase complete` no passo update_roadmap_and_state.

Verifique se as atualizações estão corretas lendo STATE.md. Se a barra de progresso precisar de atualização, use:

```bash
PROGRESS=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" progress bar --raw)
```

Atualizar a linha da barra de progresso em STATE.md com o resultado.

**Passo completo quando:**

- [ ] Número da fase incrementado para próxima fase (feito por phase complete)
- [ ] Status do plano resetado para "Não iniciado" (feito por phase complete)
- [ ] Status mostra "Pronta para planejar" (feito por phase complete)
- [ ] Barra de progresso reflete total de planos concluídos

</step>

<step name="update_project_reference">

Atualizar seção Referência do Projeto em STATE.md.

```markdown
## Referência do Projeto

Veja: .planning/PROJECT.md (atualizado [hoje])

**Valor central:** [Valor central atual de PROJECT.md]
**Foco atual:** [Nome da próxima fase]
```

Atualizar a data e foco atual para refletir a transição.

</step>

<step name="review_accumulated_context">

Revisar e atualizar seção Contexto Acumulado em STATE.md.

**Decisões:**

- Anotar decisões recentes desta fase (3-5 máx)
- Log completo vive na tabela Decisões-Chave de PROJECT.md

**Bloqueios/Preocupações:**

- Revisar bloqueios da fase concluída
- Se resolvidos nesta fase: Remover da lista
- Se ainda relevantes para o futuro: Manter com prefixo "Fase X"
- Adicionar novas preocupações dos sumários da fase concluída

**Exemplo:**

Antes:

```markdown
### Bloqueios/Preocupações

- ⚠️ [Fase 1] Schema do banco de dados não indexado para consultas comuns
- ⚠️ [Fase 2] Comportamento de reconexão WebSocket em redes instáveis desconhecido
```

Depois (se indexação do banco foi resolvida na Fase 2):

```markdown
### Bloqueios/Preocupações

- ⚠️ [Fase 2] Comportamento de reconexão WebSocket em redes instáveis desconhecido
```

**Passo completo quando:**

- [ ] Decisões recentes anotadas (log completo em PROJECT.md)
- [ ] Bloqueios resolvidos removidos da lista
- [ ] Bloqueios não resolvidos mantidos com prefixo de fase
- [ ] Novas preocupações da fase concluída adicionadas

</step>

<step name="update_session_continuity_after_transition">

Atualizar seção Continuidade de Sessão em STATE.md para refletir conclusão da transição.

**Formato:**

```markdown
Última sessão: [hoje]
Parou em: Fase [X] completa, pronta para planejar Fase [X+1]
Arquivo de retomada: Nenhum
```

**Passo completo quando:**

- [ ] Timestamp da última sessão atualizado para data e hora atuais
- [ ] Parou em descreve conclusão da fase e próxima fase
- [ ] Arquivo de retomada confirmado como Nenhum (transições não usam arquivos de retomada)

</step>

<step name="offer_next_phase">

**OBRIGATÓRIO: Verificar status do milestone antes de apresentar próximos passos.**

**Use o resultado da transição de `gsd-tools phase complete`:**

O campo `is_last_phase` do resultado do phase complete informa diretamente:
- `is_last_phase: false` → Mais fases restam → Vá para **Rota A**
- `is_last_phase: true` → Última fase concluída → **Verificar colisões de workstream primeiro**

Os campos `next_phase` e `next_phase_name` fornecem detalhes da próxima fase.

Se precisar de contexto adicional, use:
```bash
ROADMAP=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Isto retorna todas as fases com objetivos, status em disco e informação de conclusão.

---

**Verificação de colisão de workstream (quando `is_last_phase: true`):**

Antes de direcionar para Rota B, verifique se outras workstreams ainda estão ativas.
Isto previne que uma workstream avance ou complete o milestone enquanto
outras workstreams ainda estão trabalhando em suas fases.

**Pule esta verificação se NÃO estiver em modo workstream** (ou seja, `GSD_WORKSTREAM` não está definido / modo flat).
Em modo flat, vá diretamente para **Rota B**.

```bash
# Verificar apenas se estamos em modo workstream
if [ -n "$GSD_WORKSTREAM" ]; then
  WS_LIST=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" workstream list --raw)
fi
```

Analisar resultado JSON. A saída tem `{ mode, workstreams: [...] }`.
Cada entrada de workstream tem: `name`, `status`, `current_phase`, `phase_count`, `completed_phases`.

Filtrar a workstream atual (`$GSD_WORKSTREAM`) e quaisquer workstreams com
status contendo "milestone complete" ou "archived" (case-insensitive).
As entradas restantes são **outras workstreams ativas**.

- **Se outras workstreams ativas existirem** → Vá para **Rota B1**
- **Se NENHUMA outra workstream ativa** (ou modo flat) → Vá para **Rota B**

---

**Rota A: Mais fases restam no milestone**

Ler ROADMAP.md para obter nome e objetivo da próxima fase.

**Verificar se próxima fase tem CONTEXT.md:**

```bash
ls .planning/phases/*[X+1]*/*-CONTEXT.md 2>/dev/null
```

**Se próxima fase existir:**

<if mode="yolo">

**Se CONTEXT.md existir:**

```
Fase [X] marcada como completa.

Próxima: Fase [X+1] — [Nome]

⚡ Auto-continuando: Planejar Fase [X+1] em detalhe
```

Sair da skill e invocar SlashCommand("/gsd-plan-phase [X+1] --auto ${GSD_WS}")

**Se CONTEXT.md NÃO existir:**

```
Fase [X] marcada como completa.

Próxima: Fase [X+1] — [Nome]

⚡ Auto-continuando: Discutir Fase [X+1] primeiro
```

Sair da skill e invocar SlashCommand("/gsd-discuss-phase [X+1] --auto ${GSD_WS}")

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

**Se CONTEXT.md NÃO existir:**

```
## ✓ Fase [X] Completa

---

## ▶ Próximo

**Fase [X+1]: [Nome]** — [Objetivo do ROADMAP.md]

`/gsd-discuss-phase [X+1] ${GSD_WS}` — coletar contexto e esclarecer abordagem

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-plan-phase [X+1] ${GSD_WS}` — pular discussão, planejar diretamente
- `/gsd-research-phase [X+1] ${GSD_WS}` — investigar incógnitas

---
```

**Se CONTEXT.md existir:**

```
## ✓ Fase [X] Completa

---

## ▶ Próximo

**Fase [X+1]: [Nome]** — [Objetivo do ROADMAP.md]
<sub>✓ Contexto coletado, pronto para planejar</sub>

`/gsd-plan-phase [X+1] ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-discuss-phase [X+1] ${GSD_WS}` — revisitar contexto
- `/gsd-research-phase [X+1] ${GSD_WS}` — investigar incógnitas

---
```

</if>

---

**Rota B1: Workstream concluída, outras workstreams ainda ativas**

Esta rota é alcançada quando `is_last_phase: true` E a verificação de colisão encontrou
outras workstreams ativas. NÃO sugira completar o milestone ou avançar
para o próximo milestone — outras workstreams ainda estão trabalhando.

**Limpar flag de cadeia auto-avanço** — fronteira de workstream é o ponto natural de parada:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false
```

<if mode="yolo">

Sobrescrever auto-avanço: NÃO auto-continuar para conclusão de milestone.
Apresentar informação de bloqueio e parar.

</if>

Apresentar (todos os modos):

```
## ✓ Fase {X}: {Nome da Fase} Completa

As fases desta workstream estão completas. Outras workstreams ainda estão ativas:

| Workstream | Status | Fase | Progresso |
|------------|--------|------|-----------|
| {nome}     | {status} | {fase_atual} | {fases_concluidas}/{contagem_fases} |
| ...        | ...    | ...  | ...       |

---

## Próximos Passos

Arquivar esta workstream:

`/gsd-workstreams complete {nome_ws_atual} ${GSD_WS}`

Ver progresso geral do milestone:

`/gsd-workstreams progress ${GSD_WS}`

<sub>Conclusão do milestone estará disponível quando todas as workstreams terminarem.</sub>

---
```

NÃO sugira `/gsd-complete-milestone` ou `/gsd-new-milestone`.
NÃO auto-invoque nenhum outro slash command.

**Pare aqui.** O usuário deve decidir explicitamente o que fazer a seguir.

---

**Rota B: Milestone completo (todas as fases concluídas)**

**Esta rota só é alcançada quando:**
- `is_last_phase: true` E nenhuma outra workstream ativa existe (ou modo flat)

**Limpar flag de cadeia auto-avanço** — fronteira de milestone é o ponto natural de parada:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false
```

<if mode="yolo">

```
Fase {X} marcada como completa.

🎉 Milestone {versão} está 100% completo — todas as {N} fases finalizadas!

⚡ Auto-continuando: Completar milestone e arquivar
```

Sair da skill e invocar SlashCommand("/gsd-complete-milestone {versão} ${GSD_WS}")

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

```
## ✓ Fase {X}: {Nome da Fase} Completa

🎉 Milestone {versão} está 100% completo — todas as {N} fases finalizadas!

---

## ▶ Próximo

**Completar Milestone {versão}** — arquivar e preparar para o próximo

`/gsd-complete-milestone {versão} ${GSD_WS}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- Revisar conquistas antes de arquivar

---
```

</if>

</step>

</process>

<implicit_tracking>
O rastreamento de progresso é IMPLÍCITO: planejar fase N implica que fases 1-(N-1) estão completas. Nenhum passo separado de progresso — movimento para frente É progresso.
</implicit_tracking>

<partial_completion>

Se o usuário quiser seguir em frente mas a fase não está totalmente completa:

```
Fase [X] tem planos incompletos:
- {fase}-02-PLAN.md (não executado)
- {fase}-03-PLAN.md (não executado)

Opções:
1. Marcar como completa mesmo assim (planos não eram necessários)
2. Adiar trabalho para fase posterior
3. Ficar e finalizar fase atual
```

Respeite o julgamento do usuário — eles sabem se o trabalho importa.

**Se marcar como completa com planos incompletos:**

- Atualizar ROADMAP: "2/3 planos completos" (não "3/3")
- Anotar na mensagem de transição quais planos foram pulados

</partial_completion>

<success_criteria>

Transição está completa quando:

- [ ] Sumários dos planos da fase atual verificados (todos existem ou usuário escolheu pular)
- [ ] Quaisquer handoffs obsoletos deletados
- [ ] ROADMAP.md atualizado com status de conclusão e contagem de planos
- [ ] PROJECT.md evoluído (requisitos, decisões, descrição se necessário)
- [ ] STATE.md atualizado (posição, referência do projeto, contexto, sessão)
- [ ] Tabela de progresso atualizada
- [ ] Usuário sabe próximos passos

</success_criteria>
