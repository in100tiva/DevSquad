<purpose>
Auditar lacunas de validação Nyquist para uma fase concluída. Gerar testes faltantes. Atualizar VALIDATION.md.
</purpose>

<required_reading>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md
</required_reading>

<process>

## 0. Inicializar

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`.

```bash
AUDITOR_MODEL=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-auditor-nyquist --raw)
NYQUIST_CFG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-get workflow.nyquist_validation --raw)
```

Se `NYQUIST_CFG` for `false`: sair com "Validação Nyquist está desabilitada. Habilite via /gsd-configuracoes."

Exibir banner: `GSD > VALIDAR FASE {N}: {nome}`

## 1. Detectar Estado de Entrada

```bash
VALIDATION_FILE=$(ls "${PHASE_DIR}"/*-VALIDATION.md 2>/dev/null | head -1)
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
```

- **Estado A** (`VALIDATION_FILE` não vazio): Auditar existente
- **Estado B** (`VALIDATION_FILE` vazio, `SUMMARY_FILES` não vazio): Reconstruir a partir dos artefatos
- **Estado C** (`SUMMARY_FILES` vazio): Sair — "Fase {N} não executada. Execute /gsd-executar-fase {N} ${GSD_WS} primeiro."

## 2. Descoberta

### 2a. Ler Artefatos da Fase

Ler todos os arquivos PLAN e SUMMARY. Extrair: listas de tarefas, IDs de requisitos, arquivos-chave alterados, blocos de verificação.

### 2b. Construir Mapa Requisito-para-Tarefa

Por tarefa: `{ task_id, plan_id, wave, requirement_ids, has_automated_command }`

### 2c. Detectar Infraestrutura de Testes

Estado A: Analisar a partir da tabela Infraestrutura de Testes do VALIDATION.md existente.
Estado B: Varredura do sistema de arquivos:

```bash
find . -name "pytest.ini" -o -name "jest.config.*" -o -name "vitest.config.*" -o -name "pyproject.toml" 2>/dev/null | head -10
find . \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" \) -not -path "*/node_modules/*" 2>/dev/null | head -40
```

### 2d. Referência Cruzada

Associar cada requisito a testes existentes por nome de arquivo, imports, descrições de teste. Registrar: requisito → arquivo_de_teste → status.

## 3. Análise de Lacunas

Classificar cada requisito:

| Status | Critérios |
|--------|----------|
| COBERTO | Teste existe, valida comportamento, passa |
| PARCIAL | Teste existe, falhando ou incompleto |
| AUSENTE | Nenhum teste encontrado |

Construir: `{ task_id, requirement, gap_type, suggested_test_path, suggested_command }`

Sem lacunas → pular para Passo 6, definir `nyquist_compliant: true`.

## 4. Apresentar Plano de Lacunas

Chamar conversational prompting com tabela de lacunas e opções:
1. "Corrigir todas as lacunas" → Passo 5
2. "Pular — marcar como apenas manual" → adicionar a Apenas Manual, Passo 6
3. "Cancelar" → sair

## 5. Invocar gsd-auditor-nyquist

```
Task(
  prompt="Ler D:/projetos/Estudo/devsquad/.cursor/agents/gsd-auditor-nyquist.md para instruções.\n\n" +
    "<files_to_read>{PLAN, SUMMARY, arquivos de implementação, VALIDATION.md}</files_to_read>" +
    "<gaps>{lista de lacunas}</gaps>" +
    "<test_infrastructure>{framework, config, comandos}</test_infrastructure>" +
    "<constraints>Nunca modificar arquivos de implementação. Máx 3 iterações de debug. Escalar bugs de implementação.</constraints>",
  subagent_type="gsd-auditor-nyquist",
  model="{AUDITOR_MODEL}",
  description="Preencher lacunas de validação para Fase {N}"
)
```

Tratar retorno:
- `## GAPS FILLED` → registrar testes + atualizações do mapa, Passo 6
- `## PARTIAL` → registrar resolvidos, mover escalados para apenas manual, Passo 6
- `## ESCALATE` → mover todos para apenas manual, Passo 6

## 6. Gerar/Atualizar VALIDATION.md

**Estado B (criar):**
1. Ler template de `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/VALIDACAO.md`
2. Preencher: frontmatter, Infraestrutura de Testes, Mapa Por-Tarefa, Apenas Manual, Aprovação
3. Escrever em `${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md`

**Estado A (atualizar):**
1. Atualizar status do Mapa Por-Tarefa, adicionar escalados a Apenas Manual, atualizar frontmatter
2. Anexar trilha de auditoria:

```markdown
## Auditoria de Validação {data}
| Métrica | Contagem |
|---------|----------|
| Lacunas encontradas | {N} |
| Resolvidas | {M} |
| Escaladas | {K} |
```

## 7. Commit

```bash
git add {arquivos_de_teste}
git commit -m "test(phase-${PHASE}): adicionar testes de validação Nyquist"

node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PHASE}): adicionar/atualizar estratégia de validação"
```

## 8. Resultados + Roteamento

**Conforme:**
```
GSD > FASE {N} ESTÁ CONFORME COM NYQUIST
Todos os requisitos têm verificação automatizada.
▶ Próximo: /gsd-auditar-marco ${GSD_WS}
```

**Parcial:**
```
GSD > FASE {N} VALIDADA (PARCIAL)
{M} automatizados, {K} apenas manual.
▶ Tentar novamente: /gsd-validar-fase {N} ${GSD_WS}
```

Exibir lembrete de `/clear`.

</process>

<success_criteria>
- [ ] Config Nyquist verificada (sair se desabilitada)
- [ ] Estado de entrada detectado (A/B/C)
- [ ] Estado C sai corretamente
- [ ] Arquivos PLAN/SUMMARY lidos, mapa de requisitos construído
- [ ] Infraestrutura de testes detectada
- [ ] Lacunas classificadas (COBERTO/PARCIAL/AUSENTE)
- [ ] Gate do usuário com tabela de lacunas
- [ ] Auditor invocado com contexto completo
- [ ] Todos os três formatos de retorno tratados
- [ ] VALIDATION.md criado ou atualizado
- [ ] Arquivos de teste commitados separadamente
- [ ] Resultados com roteamento apresentados
</success_criteria>
</output>
