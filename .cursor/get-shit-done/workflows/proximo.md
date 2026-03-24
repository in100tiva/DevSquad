<purpose>
Detectar o estado atual do projeto e avançar automaticamente para a próxima etapa lógica do fluxo GSD.
Lê o estado do projeto para determinar: discutir → planejar → executar → verificar → concluir progressão.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="detect_state">
Ler estado do projeto para determinar posição atual:

```bash
# Obter snapshot do estado
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state json 2>/dev/null || echo "{}"
```

Também ler:
- `.planning/STATE.md` — fase atual, progresso, contagem de planos
- `.planning/ROADMAP.md` — estrutura do marco e lista de fases

Extrair:
- `current_phase` — qual fase está ativa
- `plan_of` / `plans_total` — progresso de execução de planos
- `progress` — porcentagem geral
- `status` — ativo, pausado, etc.

Se nenhum diretório `.planning/` existir:
```
Nenhum projeto GSD detectado. Execute `/gsd-novo-projeto` para começar.
```
Sair.
</step>

<step name="determine_next_action">
Aplicar regras de roteamento baseadas no estado:

**Rota 1: Nenhuma fase existe ainda → discutir**
Se ROADMAP tem fases mas nenhum diretório de fase existe no disco:
→ Próxima ação: `/gsd-discutir-fase <primeira-fase>`

**Rota 2: Fase existe mas não tem CONTEXT.md ou RESEARCH.md → discutir**
Se o diretório da fase atual existe mas não tem nem CONTEXT.md nem RESEARCH.md:
→ Próxima ação: `/gsd-discutir-fase <fase-atual>`

**Rota 3: Fase tem contexto mas sem planos → planejar**
Se a fase atual tem CONTEXT.md (ou RESEARCH.md) mas nenhum arquivo PLAN.md:
→ Próxima ação: `/gsd-planejar-fase <fase-atual>`

**Rota 4: Fase tem planos mas resumos incompletos → executar**
Se planos existem mas nem todos têm resumos correspondentes:
→ Próxima ação: `/gsd-executar-fase <fase-atual>`

**Rota 5: Todos os planos têm resumos → verificar e concluir**
Se todos os planos na fase atual têm resumos:
→ Próxima ação: `/gsd-verificar-trabalho` depois `/gsd-completar-marco`

**Rota 6: Fase concluída, próxima fase existe → avançar**
Se a fase atual está concluída e a próxima fase existe no ROADMAP:
→ Próxima ação: `/gsd-discutir-fase <próxima-fase>`

**Rota 7: Todas as fases concluídas → concluir marco**
Se todas as fases estão concluídas:
→ Próxima ação: `/gsd-completar-marco`

**Rota 8: Pausado → retomar**
Se STATE.md mostra paused_at:
→ Próxima ação: `/gsd-retomar-trabalho`
</step>

<step name="show_and_execute">
Exibir a determinação:

```
## GSD Próximo

**Atual:** Fase [N] — [nome] | [progresso]%
**Status:** [descrição do status]

▶ **Próximo passo:** `/gsd-[comando] [args]`
  [Explicação de uma linha sobre por que esta é a próxima etapa]
```

Então imediatamente invocar o comando determinado via SlashCommand.
Não pedir confirmação — o objetivo inteiro de `/gsd-proximo` é avanço sem fricção.
</step>

</process>

<success_criteria>
- [ ] Estado do projeto corretamente detectado
- [ ] Próxima ação corretamente determinada pelas regras de roteamento
- [ ] Comando invocado imediatamente sem confirmação do usuário
- [ ] Status claro mostrado antes de invocar
</success_criteria>
