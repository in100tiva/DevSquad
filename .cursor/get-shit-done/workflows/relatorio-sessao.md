<purpose>
Gerar um documento de resumo pós-sessão capturando trabalho realizado, resultados alcançados e uso estimado de recursos. Escreve SESSION_REPORT.md em .planning/reports/ para revisão humana e compartilhamento com stakeholders.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="gather_session_data">
Coletar dados da sessão de fontes disponíveis:

1. **STATE.md** — fase atual, milestone, progresso, bloqueios, decisões
2. **Log do Git** — commits feitos durante esta sessão (últimas 24h ou desde último relatório)
3. **Arquivos de Plano/Sumário** — planos executados, sumários escritos
4. **ROADMAP.md** — contexto do milestone e objetivos das fases

```bash
# Obter commits recentes (últimas 24 horas)
git log --oneline --since="24 hours ago" --no-merges 2>/dev/null || echo "Sem commits recentes"

# Contar arquivos alterados
git diff --stat HEAD~10 HEAD 2>/dev/null | tail -1 || echo "Sem diff disponível"
```

Ler `.planning/STATE.md` para obter:
- Milestone e fase atual
- Percentual de progresso
- Bloqueios ativos
- Decisões recentes

Ler `.planning/ROADMAP.md` para obter nome e objetivos do milestone.

Verificar relatórios existentes:
```bash
ls -la .planning/reports/SESSION_REPORT*.md 2>/dev/null || echo "Sem relatórios anteriores"
```
</step>

<step name="estimate_usage">
Estimar uso de tokens a partir de sinais observáveis:

- Contagem de chamadas de ferramenta não está diretamente disponível, então estime a partir da atividade git e operações de arquivo
- Nota: Esta é uma **estimativa** — contagens exatas de tokens requerem instrumentação no nível da API não disponível para hooks

Heurísticas de estimativa:
- Cada commit ≈ 1 ciclo de plano (pesquisa + planejamento + execução + verificação)
- Cada arquivo de plano ≈ 2.000-5.000 tokens de contexto do agente
- Cada arquivo de sumário ≈ 1.000-2.000 tokens gerados
- Spawns de subagentes multiplicam por ~1.5x por tipo de agente usado
</step>

<step name="generate_report">
Criar o diretório e arquivo do relatório:

```bash
mkdir -p .planning/reports
```

Escrever `.planning/reports/SESSION_REPORT.md` (ou `.planning/reports/YYYYMMDD-session-report.md` se relatórios anteriores existirem):

```markdown
# Relatório de Sessão GSD

**Gerado:** [timestamp]
**Projeto:** [do título de PROJECT.md ou nome do diretório]
**Milestone:** [N] — [nome do milestone de ROADMAP.md]

---

## Resumo da Sessão

**Duração:** [estimada do primeiro ao último timestamp de commit, ou "Sessão única"]
**Progresso da Fase:** [de STATE.md]
**Planos Executados:** [contagem de sumários escritos nesta sessão]
**Commits Feitos:** [contagem do log git]

## Trabalho Realizado

### Fases Trabalhadas
[Lista de fases trabalhadas com breve descrição do que foi feito]

### Resultados Principais
[Lista de entregas concretas: arquivos criados, funcionalidades implementadas, bugs corrigidos]

### Decisões Tomadas
[Da tabela de decisões de STATE.md, se alguma foi adicionada nesta sessão]

## Arquivos Alterados

[Resumo de arquivos modificados, criados, deletados — do git diff stat]

## Bloqueios & Itens Abertos

[Bloqueios ativos de STATE.md]
[Quaisquer itens TODO criados durante a sessão]

## Uso Estimado de Recursos

| Métrica | Estimativa |
|---------|------------|
| Commits | [N] |
| Arquivos alterados | [N] |
| Planos executados | [N] |
| Subagentes criados | [estimado] |

> **Nota:** Estimativas de tokens e custos requerem instrumentação no nível da API.
> Estas métricas refletem apenas a atividade observável da sessão.

---

*Gerado por `/gsd-session-report`*
```
</step>

<step name="display_result">
Mostrar ao usuário:

```
## Relatório de Sessão Gerado

📄 `.planning/reports/[nome_arquivo].md`

### Destaques
- **Commits:** [N]
- **Arquivos alterados:** [N]
- **Progresso da fase:** [X]%
- **Planos executados:** [N]
```

Se este for o primeiro relatório, mencionar:
```
💡 Execute `/gsd-session-report` ao final de cada sessão para construir um histórico de atividade do projeto.
```
</step>

</process>

<success_criteria>
- [ ] Dados da sessão coletados de STATE.md, log git e arquivos de plano
- [ ] Relatório escrito em .planning/reports/
- [ ] Relatório inclui resumo do trabalho, resultados e mudanças de arquivo
- [ ] Nome do arquivo inclui data para evitar sobrescrita
- [ ] Resumo do resultado exibido ao usuário
</success_criteria>
