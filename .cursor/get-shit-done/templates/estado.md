# Template de Estado

Template para `.planning/STATE.md` — a memória viva do projeto.

---

## Template do Arquivo

```markdown
# Estado do Projeto

## Referência do Projeto

Veja: .planning/PROJECT.md (atualizado [data])

**Valor central:** [Resumo da seção Valor Central do PROJECT.md]
**Foco atual:** [Nome da fase atual]

## Posição Atual

Fase: [X] de [Y] ([Nome da fase])
Plano: [A] de [B] na fase atual
Status: [Pronto para planejar / Planejando / Pronto para executar / Em progresso / Fase concluída]
Última atividade: [AAAA-MM-DD] — [O que aconteceu]

Progresso: [░░░░░░░░░░] 0%

## Métricas de Performance

**Velocidade:**
- Total de planos concluídos: [N]
- Duração média: [X] min
- Tempo total de execução: [X.X] horas

**Por Fase:**

| Fase | Planos | Total | Média/Plano |
|------|--------|-------|-------------|
| - | - | - | - |

**Tendência Recente:**
- Últimos 5 planos: [durações]
- Tendência: [Melhorando / Estável / Degradando]

*Atualizado após cada conclusão de plano*

## Contexto Acumulado

### Decisões

Decisões são registradas na tabela Decisões Chave do PROJECT.md.
Decisões recentes que afetam o trabalho atual:

- [Fase X]: [Resumo da decisão]
- [Fase Y]: [Resumo da decisão]

### Todos Pendentes

[De .planning/todos/pending/ — ideias capturadas durante sessões]

Nenhum ainda.

### Bloqueios/Preocupações

[Problemas que afetam trabalho futuro]

Nenhum ainda.

## Continuidade de Sessão

Última sessão: [AAAA-MM-DD HH:MM]
Parou em: [Descrição da última ação concluída]
Arquivo de retomada: [Caminho para .continue-here*.md se existir, caso contrário "Nenhum"]
```

<purpose>

STATE.md é a memória de curto prazo do projeto abrangendo todas as fases e sessões.

**Problema que resolve:** Informação é capturada em resumos, problemas e decisões mas não é consumida sistematicamente. Sessões começam sem contexto.

**Solução:** Um arquivo único e pequeno que é:
- Lido primeiro em cada workflow
- Atualizado após cada ação significativa
- Contém resumo do contexto acumulado
- Permite restauração instantânea de sessão

</purpose>

<lifecycle>

**Criação:** Após ROADMAP.md ser criado (durante init)
- Referenciar PROJECT.md (leia para contexto atual)
- Inicializar seções de contexto acumulado vazias
- Definir posição como "Fase 1 pronta para planejar"

**Leitura:** Primeiro passo de cada workflow
- progress: Apresentar status ao usuário
- plan: Informar decisões de planejamento
- execute: Saber posição atual
- transition: Saber o que está concluído

**Escrita:** Após cada ação significativa
- execute: Após SUMMARY.md ser criado
  - Atualizar posição (fase, plano, status)
  - Anotar novas decisões (detalhe no PROJECT.md)
  - Adicionar bloqueios/preocupações
- transition: Após fase ser marcada como concluída
  - Atualizar barra de progresso
  - Limpar bloqueios resolvidos
  - Atualizar data da Referência do Projeto

</lifecycle>

<sections>

### Referência do Projeto
Aponta para PROJECT.md para contexto completo. Inclui:
- Valor central (a ÚNICA coisa que importa)
- Foco atual (qual fase)
- Data da última atualização (aciona releitura se desatualizado)

Claude lê PROJECT.md diretamente para requisitos, restrições e decisões.

### Posição Atual
Onde estamos agora:
- Fase X de Y — qual fase
- Plano A de B — qual plano dentro da fase
- Status — estado atual
- Última atividade — o que aconteceu mais recentemente
- Barra de progresso — indicador visual do progresso geral

Cálculo do progresso: (planos concluídos) / (total de planos em todas as fases) × 100%

### Métricas de Performance
Acompanhe a velocidade para entender padrões de execução:
- Total de planos concluídos
- Duração média por plano
- Detalhamento por fase
- Tendência recente (melhorando/estável/degradando)

Atualizado após cada conclusão de plano.

### Contexto Acumulado

**Decisões:** Referência à tabela Decisões Chave do PROJECT.md, mais resumo de decisões recentes para acesso rápido. O log completo de decisões fica no PROJECT.md.

**Todos Pendentes:** Ideias capturadas via /gsd-add-todo
- Contagem de todos pendentes
- Referência a .planning/todos/pending/
- Lista breve se poucos, contagem se muitos (ex., "5 todos pendentes — veja /gsd-check-todos")

**Bloqueios/Preocupações:** Das seções "Prontidão para Próxima Fase"
- Problemas que afetam trabalho futuro
- Prefixar com a fase de origem
- Limpar quando resolvidos

### Continuidade de Sessão
Permite retomada instantânea:
- Quando foi a última sessão
- O que foi concluído por último
- Existe um arquivo .continue-here para retomar

</sections>

<size_constraint>

Mantenha STATE.md com menos de 100 linhas.

É um RESUMO, não um arquivo. Se o contexto acumulado crescer demais:
- Mantenha apenas 3-5 decisões recentes no resumo (log completo no PROJECT.md)
- Mantenha apenas bloqueios ativos, remova os resolvidos

O objetivo é "ler uma vez, saber onde estamos" — se estiver muito longo, isso falha.

</size_constraint>
