<purpose>
Inserir uma fase decimal para trabalho urgente descoberto no meio do marco entre fases inteiras existentes. Usa numeração decimal (72.1, 72.2, etc.) para preservar a sequência lógica das fases planejadas enquanto acomoda inserções urgentes sem renumerar todo o roteiro.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analisar os argumentos do comando:
- Primeiro argumento: número inteiro da fase para inserir depois
- Argumentos restantes: descrição da fase

Exemplo: `/gsd-inserir-fase 72 Corrigir bug crítico de auth`
-> after = 72
-> descrição = "Corrigir bug crítico de auth"

Se argumentos faltando:

```
ERRO: Número da fase e descrição são obrigatórios
Uso: /gsd-inserir-fase <depois> <descrição>
Exemplo: /gsd-inserir-fase 72 Corrigir bug crítico de auth
```

Encerrar.

Validar que o primeiro argumento é um inteiro.
</step>

<step name="init_context">
Carregar contexto da operação de fase:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${after_phase}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Verificar `roadmap_exists` do JSON de init. Se falso:
```
ERRO: Nenhum roteiro encontrado (.planning/ROADMAP.md)
```
Encerrar.
</step>

<step name="insert_phase">
**Delegar a inserção da fase ao gsd-tools:**

```bash
RESULT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase insert "${after_phase}" "${description}")
```

O CLI cuida de:
- Verificar que a fase alvo existe no ROADMAP.md
- Calcular próximo número decimal de fase (verificando decimais existentes em disco)
- Gerar slug a partir da descrição
- Criar o diretório da fase (`.planning/phases/{N.M}-{slug}/`)
- Inserir a entrada da fase no ROADMAP.md após a fase alvo com marcador (INSERIDA)

Extrair do resultado: `phase_number`, `after_phase`, `name`, `slug`, `directory`.
</step>

<step name="update_project_state">
Atualizar STATE.md para refletir a fase inserida:

1. Ler `.planning/STATE.md`
2. Em "## Contexto Acumulado" → "### Evolução do Roteiro" adicionar entrada:
   ```
   - Fase {decimal_phase} inserida após Fase {after_phase}: {descrição} (URGENTE)
   ```

Se a seção "Evolução do Roteiro" não existir, criá-la.
</step>

<step name="completion">
Apresentar resumo de conclusão:

```
Fase {decimal_phase} inserida após Fase {after_phase}:
- Descrição: {descrição}
- Diretório: .planning/phases/{decimal-phase}-{slug}/
- Status: Ainda não planejada
- Marcador: (INSERIDA) - indica trabalho urgente

Roteiro atualizado: .planning/ROADMAP.md
Estado do projeto atualizado: .planning/STATE.md

---

## Próximo Passo

**Fase {decimal_phase}: {descrição}** -- inserção urgente

`/gsd-planejar-fase {decimal_phase}`

<sub>`/clear` primeiro -> janela de contexto limpa</sub>

---

**Também disponível:**
- Revisar impacto da inserção: Verificar se dependências da Fase {next_integer} ainda fazem sentido
- Revisar roteiro

---
```
</step>

</process>

<anti_patterns>

- Não usar isto para trabalho planejado no final do marco (use /gsd-adicionar-fase)
- Não inserir antes da Fase 1 (decimal 0.1 não faz sentido)
- Não renumerar fases existentes
- Não modificar o conteúdo da fase alvo
- Não criar planos ainda (isso é /gsd-planejar-fase)
- Não commitar mudanças (usuário decide quando commitar)
</anti_patterns>

<success_criteria>
A inserção da fase está completa quando:

- [ ] `gsd-tools phase insert` executado com sucesso
- [ ] Diretório da fase criado
- [ ] Roteiro atualizado com nova entrada de fase (inclui marcador "(INSERIDA)")
- [ ] STATE.md atualizado com nota de evolução do roteiro
- [ ] Usuário informado dos próximos passos e implicações de dependência
</success_criteria>
