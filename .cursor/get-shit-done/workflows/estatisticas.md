<purpose>
Exibir estatísticas abrangentes do projeto incluindo fases, planos, requisitos, métricas git e cronograma.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="gather_stats">
Coletar estatísticas do projeto:

```bash
STATS=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" stats json)
if [[ "$STATS" == @file:* ]]; then STATS=$(cat "${STATS#@file:}"); fi
```

Extrair campos do JSON: `milestone_version`, `milestone_name`, `phases`, `phases_completed`, `phases_total`, `total_plans`, `total_summaries`, `percent`, `plan_percent`, `requirements_total`, `requirements_complete`, `git_commits`, `git_first_commit_date`, `last_activity`.
</step>

<step name="present_stats">
Apresentar ao usuário com este formato:

```
# 📊 Estatísticas do Projeto — {milestone_version} {milestone_name}

## Progresso
[████████░░] X/Y fases (Z%)

## Planos
X/Y planos concluídos (Z%)

## Fases
| Fase | Nome | Planos | Concluídos | Status |
|------|------|--------|------------|--------|
| ...  | ...  | ...    | ...        | ...    |

## Requisitos
✅ X/Y requisitos concluídos

## Git
- **Commits:** N
- **Iniciado:** AAAA-MM-DD
- **Última atividade:** AAAA-MM-DD

## Cronograma
- **Idade do projeto:** N dias
```

Se nenhum diretório `.planning/` existir, informar ao usuário para executar `/gsd-novo-projeto` primeiro.
</step>

</process>

<success_criteria>
- [ ] Estatísticas coletadas do estado do projeto
- [ ] Resultados formatados claramente
- [ ] Exibido ao usuário
</success_criteria>
