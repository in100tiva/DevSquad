<purpose>
Pesquisar como implementar uma fase. Invoca o gsd-pesquisador-fase com o contexto da fase.

Comando de pesquisa independente. Para a maioria dos fluxos, use `/gsd-planejar-fase` que integra a pesquisa automaticamente.
</purpose>

<process>

## Passo 0: Resolver Perfil de Modelo

@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/resolucao-perfil-modelo.md

Resolver modelo para:
- `gsd-pesquisador-fase`

## Passo 1: Normalizar e Validar Fase

@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/parsing-argumento-fase.md

```bash
PHASE_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

Se `found` for false: Erro e sair.

## Passo 2: Verificar Pesquisa Existente

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null
```

Se existir: Oferecer opções atualizar/visualizar/pular.

## Passo 3: Coletar Contexto da Fase

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# Extrair: phase_dir, padded_phase, phase_number, state_path, requirements_path, context_path
```

## Passo 4: Invocar Pesquisador

```
Task(
  prompt="<objective>
Pesquisar abordagem de implementação para Fase {phase}: {name}
</objective>

<files_to_read>
- {context_path} (DECISÕES DO USUÁRIO do /gsd-discutir-fase)
- {requirements_path} (Requisitos do projeto)
- {state_path} (Decisões e histórico do projeto)
</files_to_read>

<additional_context>
Descrição da fase: {description}
</additional_context>

<output>
Escrever em: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</output>",
  subagent_type="gsd-pesquisador-fase",
  model="{researcher_model}"
)
```

## Passo 5: Tratar Retorno

- `## RESEARCH COMPLETE` — Exibir resumo, oferecer: Planejar/Aprofundar/Revisar/Concluído
- `## CHECKPOINT REACHED` — Apresentar ao usuário, invocar continuação
- `## RESEARCH INCONCLUSIVE` — Mostrar tentativas, oferecer: Adicionar contexto/Tentar modo diferente/Manual

</process>
</output>
