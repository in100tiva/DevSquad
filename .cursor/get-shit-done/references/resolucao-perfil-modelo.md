# Resolução de Perfil de Modelo

Resolver perfil de modelo uma vez no início da orquestração, depois usá-lo para todas as criações de Task.

## Padrão de Resolução

```bash
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
```

Padrão: `balanced` se não definido ou config ausente.

## Tabela de Consulta

@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/perfis-modelo.md

Consulte o agente na tabela para o perfil resolvido. Passe o parâmetro de modelo para chamadas Task:

```
Task(
  prompt="...",
  subagent_type="gsd-planner",
  model="{modelo_resolvido}"  # "inherit", "sonnet", ou "haiku"
)
```

**Nota:** Agentes de nível Opus resolvem para `"inherit"` (não `"opus"`). Isso faz o agente usar o modelo da sessão pai, evitando conflitos com políticas organizacionais que podem bloquear versões específicas do opus.

Se `model_profile` for `"inherit"`, todos os agentes resolvem para `"inherit"` (útil para OpenCode `/model`).

## Uso

1. Resolver uma vez no início da orquestração
2. Armazenar o valor do perfil
3. Consultar o modelo de cada agente na tabela ao criar
4. Passar parâmetro de modelo para cada chamada Task (valores: `"inherit"`, `"sonnet"`, `"haiku"`)
