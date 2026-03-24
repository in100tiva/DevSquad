<purpose>
Adicionar uma nova fase inteira ao final do marco atual no roteiro. Calcula automaticamente o próximo número de fase, cria o diretório da fase e atualiza a estrutura do roteiro.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analise os argumentos do comando:
- Todos os argumentos se tornam a descrição da fase
- Exemplo: `/gsd-add-phase Adicionar autenticação` → descrição = "Adicionar autenticação"
- Exemplo: `/gsd-add-phase Corrigir problemas críticos de performance` → descrição = "Corrigir problemas críticos de performance"

Se nenhum argumento fornecido:

```
ERRO: Descrição da fase obrigatória
Uso: /gsd-add-phase <descrição>
Exemplo: /gsd-add-phase Adicionar sistema de autenticação
```

Encerrar.
</step>

<step name="init_context">
Carregar contexto da operação de fase:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "0")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Verificar `roadmap_exists` do JSON de init. Se falso:
```
ERRO: Nenhum roteiro encontrado (.planning/ROADMAP.md)
Execute /gsd-new-project para inicializar.
```
Encerrar.
</step>

<step name="add_phase">
**Delegar a adição da fase ao gsd-tools:**

```bash
RESULT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase add "${description}")
```

O CLI cuida de:
- Encontrar o maior número inteiro de fase existente
- Calcular o próximo número de fase (max + 1)
- Gerar slug a partir da descrição
- Criar o diretório da fase (`.planning/phases/{NN}-{slug}/`)
- Inserir a entrada da fase no ROADMAP.md com seções de Objetivo, Depende de e Planos

Extrair do resultado: `phase_number`, `padded`, `name`, `slug`, `directory`.
</step>

<step name="update_project_state">
Atualizar STATE.md para refletir a nova fase:

1. Ler `.planning/STATE.md`
2. Em "## Contexto Acumulado" → "### Evolução do Roteiro" adicionar entrada:
   ```
   - Fase {N} adicionada: {descrição}
   ```

Se a seção "Evolução do Roteiro" não existir, criá-la.
</step>

<step name="completion">
Apresentar resumo de conclusão:

```
Fase {N} adicionada ao marco atual:
- Descrição: {descrição}
- Diretório: .planning/phases/{fase-num}-{slug}/
- Status: Ainda não planejada

Roteiro atualizado: .planning/ROADMAP.md

---

## ▶ Próximo Passo

**Fase {N}: {descrição}**

`/gsd-plan-phase {N}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-add-phase <descrição>` — adicionar outra fase
- Revisar roteiro

---
```
</step>

</process>

<success_criteria>
- [ ] `gsd-tools phase add` executado com sucesso
- [ ] Diretório da fase criado
- [ ] Roteiro atualizado com nova entrada de fase
- [ ] STATE.md atualizado com nota de evolução do roteiro
- [ ] Usuário informado dos próximos passos
</success_criteria>
