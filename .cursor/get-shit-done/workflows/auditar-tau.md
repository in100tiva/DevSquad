<purpose>
Auditoria entre fases de todos os arquivos de TAU e verificação. Encontra todos os itens pendentes (pendente, pulado, bloqueado, necessita_humano), opcionalmente verifica contra o codebase para detectar docs desatualizados, e produz um plano de testes humano priorizado.
</purpose>

<process>

<step name="initialize">
Executar a auditoria pelo CLI:

```bash
AUDIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" audit-uat --raw)
```

Analisar JSON para array `results` e objeto `summary`.

Se `summary.total_items` for 0:
```
## Tudo Limpo

Nenhum item pendente de TAU ou verificação encontrado em todas as fases.
Todos os testes estão passando, resolvidos ou diagnosticados com planos de correção.
```
Parar aqui.
</step>

<step name="categorize">
Agrupar itens pelo que é acionável AGORA vs. o que precisa de pré-requisitos:

**Testável Agora** (sem dependências externas):
- `pending` — testes nunca executados
- `human_uat` — itens de verificação humana
- `skipped_unresolved` — pulados sem razão clara de bloqueio

**Precisa de Pré-requisitos:**
- `server_blocked` — precisa de servidor externo rodando
- `device_needed` — precisa de dispositivo físico (não simulador)
- `build_needed` — precisa de build release/preview
- `third_party` — precisa de configuração de serviço externo

Para cada item em "Testável Agora", usar Grep/Read para verificar se a funcionalidade subjacente ainda existe no codebase:
- Se o teste referencia um componente/função que não existe mais → marcar como `desatualizado`
- Se o teste referencia código que foi significativamente reescrito → marcar como `precisa_atualização`
- Caso contrário → marcar como `ativo`
</step>

<step name="present">
Apresentar o relatório de auditoria:

```
## Relatório de Auditoria TAU

**{total_items} itens pendentes em {total_files} arquivos de {phase_count} fases**

### Testável Agora ({count})

| # | Fase | Teste | Descrição | Status |
|---|------|-------|-----------|--------|
| 1 | {fase} | {nome_teste} | {esperado} | {ativo/desatualizado/precisa_atualização} |
...

### Precisa de Pré-requisitos ({count})

| # | Fase | Teste | Bloqueado Por | Descrição |
|---|------|-------|---------------|-----------|
| 1 | {fase} | {nome_teste} | {categoria} | {esperado} |
...

### Desatualizado (pode ser encerrado) ({count})

| # | Fase | Teste | Por Que Desatualizado |
|---|------|-------|-----------------------|
| 1 | {fase} | {nome_teste} | {razão} |
...

---

## Ações Recomendadas

1. **Encerrar itens desatualizados:** `/gsd-verify-work {fase}` — marcar testes desatualizados como resolvidos
2. **Executar testes ativos:** Plano de testes humano TAU abaixo
3. **Quando pré-requisitos atendidos:** Retestar itens bloqueados com `/gsd-verify-work {fase}`
```
</step>

<step name="test_plan">
Gerar um plano de testes humano TAU apenas para itens "Testável Agora" + "ativo":

Agrupar pelo que pode ser testado junto (mesma tela, mesma funcionalidade, mesmo pré-requisito):

```
## Plano de Testes Humano TAU

### Grupo 1: {categoria — ex., "Fluxo de Faturamento"}
Pré-requisitos: {o que precisa estar rodando/configurado}

1. **{Nome do teste}** (Fase {N})
   - Navegar para: {onde}
   - Fazer: {ação}
   - Esperado: {comportamento esperado}

2. **{Nome do teste}** (Fase {N})
   ...

### Grupo 2: {categoria}
...
```
</step>

</process>
