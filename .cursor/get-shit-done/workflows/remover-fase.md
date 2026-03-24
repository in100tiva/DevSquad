<purpose>
Remover uma fase futura não iniciada do roteiro do projeto, deletar seu diretório, renumerar todas as fases subsequentes para manter uma sequência linear limpa, e commitar a mudança. O commit git serve como o registro histórico da remoção.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analisar os argumentos do comando:
- Argumento é o número da fase a remover (inteiro ou decimal)
- Exemplo: `/gsd-remover-fase 17` → fase = 17
- Exemplo: `/gsd-remover-fase 16.1` → fase = 16.1

Se nenhum argumento fornecido:

```
ERRO: Número da fase obrigatório
Uso: /gsd-remover-fase <número-da-fase>
Exemplo: /gsd-remover-fase 17
```

Encerrar.
</step>

<step name="init_context">
Carregar contexto da operação de fase:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${target}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair: `phase_found`, `phase_dir`, `phase_number`, `commit_docs`, `roadmap_exists`.

Também ler conteúdo do STATE.md e ROADMAP.md para analisar posição atual.
</step>

<step name="validate_future_phase">
Verificar que a fase é uma fase futura (não iniciada):

1. Comparar fase alvo com fase atual do STATE.md
2. Alvo deve ser > número da fase atual

Se alvo <= fase atual:

```
ERRO: Não é possível remover Fase {target}

Apenas fases futuras podem ser removidas:
- Fase atual: {atual}
- Fase {target} é atual ou concluída

Para abandonar trabalho atual, use /gsd-pausar-trabalho em vez disso.
```

Encerrar.
</step>

<step name="confirm_removal">
Apresentar resumo de remoção e confirmar:

```
Removendo Fase {target}: {Nome}

Isto irá:
- Deletar: .planning/phases/{target}-{slug}/
- Renumerar todas as fases subsequentes
- Atualizar: ROADMAP.md, STATE.md

Prosseguir? (s/n)
```

Aguardar confirmação.
</step>

<step name="execute_removal">
**Delegar toda a operação de remoção ao gsd-tools:**

```bash
RESULT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase remove "${target}")
```

Se a fase tiver planos executados (arquivos SUMMARY.md), gsd-tools retornará erro. Usar `--force` apenas se o usuário confirmar:

```bash
RESULT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase remove "${target}" --force)
```

O CLI cuida de:
- Deletar o diretório da fase
- Renumerar todos os diretórios subsequentes (em ordem reversa para evitar conflitos)
- Renomear todos os arquivos dentro dos diretórios renumerados (PLAN.md, SUMMARY.md, etc.)
- Atualizar ROADMAP.md (remover seção, renumerar todas as referências de fase, atualizar dependências)
- Atualizar STATE.md (decrementar contagem de fases)

Extrair do resultado: `removed`, `directory_deleted`, `renamed_directories`, `renamed_files`, `roadmap_updated`, `state_updated`.
</step>

<step name="commit">
Adicionar ao staging e commitar a remoção:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "chore: remove phase {target} ({nome-original-da-fase})" --files .planning/
```

A mensagem de commit preserva o registro histórico do que foi removido.
</step>

<step name="completion">
Apresentar resumo de conclusão:

```
Fase {target} ({nome-original}) removida.

Mudanças:
- Deletado: .planning/phases/{target}-{slug}/
- Renumerado: {N} diretórios e {M} arquivos
- Atualizado: ROADMAP.md, STATE.md
- Commitado: chore: remove phase {target} ({nome-original})

---

## Qual o Próximo Passo

Deseja:
- `/gsd-progresso` — ver status atualizado do roteiro
- Continuar com a fase atual
- Revisar roteiro

---
```
</step>

</process>

<anti_patterns>

- Não remover fases concluídas (têm arquivos SUMMARY.md) sem --force
- Não remover fases atuais ou passadas
- Não renumerar manualmente — usar `gsd-tools phase remove` que trata toda a renumeração
- Não adicionar notas "fase removida" ao STATE.md — commit git é o registro
- Não modificar diretórios de fases concluídas
</anti_patterns>

<success_criteria>
A remoção da fase está completa quando:

- [ ] Fase alvo validada como futura/não iniciada
- [ ] `gsd-tools phase remove` executado com sucesso
- [ ] Mudanças commitadas com mensagem descritiva
- [ ] Usuário informado das mudanças
</success_criteria>
