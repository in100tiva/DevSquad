<purpose>
Operador autônomo de reparo para verificação de tarefa que falhou. Invocado por execute-plan quando uma tarefa falha em seus critérios de conclusão. Propõe e tenta correções estruturadas antes de escalar para o usuário.
</purpose>

<inputs>
- FAILED_TASK: Número da tarefa, nome e critérios de conclusão do plano
- ERROR: O que a verificação produziu — resultado real vs esperado
- PLAN_CONTEXT: Tarefas adjacentes e objetivo da fase (para consciência de restrições)
- REPAIR_BUDGET: Máximo de tentativas de reparo restantes (padrão: 2)
</inputs>

<repair_directive>
Analise a falha e escolha exatamente uma estratégia de reparo:

**RETRY** — A abordagem estava certa mas a execução falhou. Tente novamente com um ajuste concreto.
- Use quando: erro de comando, dependência ausente, caminho errado, problema de ambiente, falha transiente
- Saída: `RETRY: [ajuste específico a fazer antes de tentar novamente]`

**DECOMPOSE** — A tarefa é grosseira demais. Quebre em sub-passos menores verificáveis.
- Use quando: critérios de conclusão cobrem múltiplas preocupações, lacunas de implementação são estruturais
- Saída: `DECOMPOSE: [sub-tarefa 1] | [sub-tarefa 2] | ...` (máx 3 sub-tarefas)
- Sub-tarefas devem cada uma ter um resultado verificável único

**PRUNE** — A tarefa é inviável dadas as restrições atuais. Pule com justificativa.
- Use quando: pré-requisito ausente e não corrigível aqui, fora do escopo, contradiz decisão anterior
- Saída: `PRUNE: [justificativa de uma frase]`

**ESCALATE** — Orçamento de reparo esgotado, ou isto é uma decisão arquitetural (Regra 4).
- Use quando: RETRY falhou mais de uma vez com abordagens diferentes, ou correção requer mudança estrutural
- Saída: `ESCALATE: [o que foi tentado] | [que decisão é necessária]`
</repair_directive>

<process>

<step name="diagnose">
Leia o erro e critérios de conclusão cuidadosamente. Pergunte:
1. Este é um problema transiente/ambiental? → RETRY
2. A tarefa é verificavelmente ampla demais? → DECOMPOSE
3. Um pré-requisito está genuinamente ausente e não corrigível no escopo? → PRUNE
4. RETRY já foi tentado com esta tarefa? Verifique REPAIR_BUDGET. Se 0 → ESCALATE
</step>

<step name="execute_retry">
Se RETRY:
1. Aplique o ajuste específico indicado na diretiva
2. Re-execute a implementação da tarefa
3. Re-execute a verificação
4. Se passar → continue normalmente, registre `[Reparo de Node - RETRY] Tarefa [X]: [ajuste feito]`
5. Se falhar novamente → decremente REPAIR_BUDGET, re-invoque reparar-node com contexto atualizado
</step>

<step name="execute_decompose">
Se DECOMPOSE:
1. Substitua a tarefa que falhou inline com as sub-tarefas (não modifique PLAN.md em disco)
2. Execute sub-tarefas sequencialmente, cada uma com sua própria verificação
3. Se todas sub-tarefas passarem → trate tarefa original como bem-sucedida, registre `[Reparo de Node - DECOMPOSE] Tarefa [X] → [N] sub-tarefas`
4. Se uma sub-tarefa falhar → re-invoque reparar-node para essa sub-tarefa (REPAIR_BUDGET se aplica por sub-tarefa)
</step>

<step name="execute_prune">
Se PRUNE:
1. Marque tarefa como pulada com justificativa
2. Registre no SUMMARY "Problemas Encontrados": `[Reparo de Node - PRUNE] Tarefa [X]: [justificativa]`
3. Continue para próxima tarefa
</step>

<step name="execute_escalate">
Se ESCALATE:
1. Apresente ao usuário via verification_failure_gate com histórico completo de reparo
2. Apresente: o que foi tentado (cada tentativa RETRY/DECOMPOSE), qual é o bloqueio, opções disponíveis
3. Aguarde direcionamento do usuário antes de continuar
</step>

</process>

<logging>
Todas as ações de reparo devem aparecer em SUMMARY.md sob "## Desvios do Plano":

| Tipo | Formato |
|------|---------|
| RETRY sucesso | `[Reparo de Node - RETRY] Tarefa X: [ajuste] — resolvido` |
| RETRY falha → ESCALATE | `[Reparo de Node - RETRY] Tarefa X: [N] tentativas esgotadas — escalado ao usuário` |
| DECOMPOSE | `[Reparo de Node - DECOMPOSE] Tarefa X dividida em [N] sub-tarefas — todas passaram` |
| PRUNE | `[Reparo de Node - PRUNE] Tarefa X pulada: [justificativa]` |
</logging>

<constraints>
- REPAIR_BUDGET padrão é 2 por tarefa. Configurável via config.json `workflow.node_repair_budget`.
- Nunca modifique PLAN.md em disco — sub-tarefas decompostas existem apenas em memória.
- Sub-tarefas DECOMPOSE devem ser mais específicas que a original, não reescritas sinônimas.
- Se config.json `workflow.node_repair` for `false`, pule diretamente para verification_failure_gate (usuário mantém comportamento original).
</constraints>
