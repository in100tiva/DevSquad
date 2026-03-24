<purpose>
Validar integridade do diretório `.planning/` e reportar problemas acionáveis. Verifica arquivos ausentes, configurações inválidas, estado inconsistente e planos órfãos. Opcionalmente repara problemas auto-corrigíveis.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_args">
**Analisar argumentos:**

Verificar se flag `--repair` está presente nos argumentos do comando.

```
REPAIR_FLAG=""
if arguments contain "--repair"; then
  REPAIR_FLAG="--repair"
fi
```
</step>

<step name="run_health_check">
**Executar validação de saúde:**

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" validate health $REPAIR_FLAG
```

Extrair do JSON de saída:
- `status`: "healthy" | "degraded" | "broken"
- `errors[]`: Problemas críticos (code, message, fix, repairable)
- `warnings[]`: Problemas não-críticos
- `info[]`: Notas informativas
- `repairable_count`: Número de problemas auto-corrigíveis
- `repairs_performed[]`: Ações tomadas se --repair foi usado
</step>

<step name="format_output">
**Formatar e exibir resultados:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Verificação de Saúde GSD
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: SAUDÁVEL | DEGRADADO | QUEBRADO
Erros: N | Avisos: N | Info: N
```

**Se reparos foram realizados:**
```
## Reparos Realizados

- ✓ config.json: Criado com padrões
- ✓ STATE.md: Regenerado a partir do roteiro
```

**Se erros existirem:**
```
## Erros

- [E001] config.json: Erro de parsing JSON na linha 5
  Correção: Execute /gsd-saude --repair para redefinir com padrões

- [E002] PROJECT.md não encontrado
  Correção: Execute /gsd-novo-projeto para criar
```

**Se avisos existirem:**
```
## Avisos

- [W002] STATE.md referencia fase 5, mas apenas fases 1-3 existem
  Correção: Revise STATE.md manualmente antes de alterar; reparo não sobrescreverá STATE.md existente

- [W005] Diretório de fase "1-setup" não segue formato NN-nome
  Correção: Renomear para corresponder ao padrão (ex: 01-setup)
```

**Se info existir:**
```
## Info

- [I001] 02-implementation/02-01-PLAN.md não tem SUMMARY.md
  Nota: Pode estar em andamento
```

**Rodapé (se problemas reparáveis existirem e --repair NÃO foi usado):**
```
---
N problemas podem ser auto-reparados. Execute: /gsd-saude --repair
```
</step>

<step name="offer_repair">
**Se problemas reparáveis existirem e --repair NÃO foi usado:**

Perguntar ao usuário se deseja executar reparos:

```
Gostaria de executar /gsd-saude --repair para corrigir N problemas automaticamente?
```

Se sim, re-executar com flag --repair e exibir resultados.
</step>

<step name="verify_repairs">
**Se reparos foram realizados:**

Re-executar verificação de saúde sem --repair para confirmar que problemas foram resolvidos:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" validate health
```

Reportar status final.
</step>

</process>

<error_codes>

| Código | Severidade | Descrição | Reparável |
|--------|-----------|-----------|-----------|
| E001 | erro | Diretório .planning/ não encontrado | Não |
| E002 | erro | PROJECT.md não encontrado | Não |
| E003 | erro | ROADMAP.md não encontrado | Não |
| E004 | erro | STATE.md não encontrado | Sim |
| E005 | erro | Erro de parsing em config.json | Sim |
| W001 | aviso | PROJECT.md com seção obrigatória ausente | Não |
| W002 | aviso | STATE.md referencia fase inválida | Não |
| W003 | aviso | config.json não encontrado | Sim |
| W004 | aviso | config.json com valor de campo inválido | Não |
| W005 | aviso | Nomenclatura de diretório de fase incompatível | Não |
| W006 | aviso | Fase no ROADMAP mas sem diretório | Não |
| W007 | aviso | Fase no disco mas não no ROADMAP | Não |
| W008 | aviso | config.json: workflow.nyquist_validation ausente (padrão habilitado mas agentes podem pular) | Sim |
| W009 | aviso | Fase tem Arquitetura de Validação no RESEARCH.md mas sem VALIDATION.md | Não |
| I001 | info | Plano sem SUMMARY (pode estar em andamento) | Não |

</error_codes>

<repair_actions>

| Ação | Efeito | Risco |
|------|--------|-------|
| createConfig | Criar config.json com padrões | Nenhum |
| resetConfig | Deletar + recriar config.json | Perde configurações personalizadas |
| regenerateState | Criar STATE.md a partir da estrutura ROADMAP quando ausente | Perde histórico de sessão |
| addNyquistKey | Adicionar workflow.nyquist_validation: true ao config.json | Nenhum — corresponde ao padrão existente |

**Não reparável (muito arriscado):**
- Conteúdo de PROJECT.md, ROADMAP.md
- Renomeação de diretório de fase
- Limpeza de planos órfãos

</repair_actions>

<stale_task_cleanup>
**Específico do Windows:** Verificar diretórios de tarefas do Cursor obsoletos que acumulam em crash/travamento.
Estes são deixados para trás quando subagentes são encerrados à força e consomem espaço em disco.

Quando `--repair` está ativo, detectar e limpar:

```bash
# Verificar diretórios de tarefas obsoletos (mais de 24 horas)
TASKS_DIR="D:/projetos/Estudo/devsquad/.cursor/tasks"
if [ -d "$TASKS_DIR" ]; then
  STALE_COUNT=$(find "$TASKS_DIR" -maxdepth 1 -type d -mtime +1 2>/dev/null | wc -l)
  if [ "$STALE_COUNT" -gt 0 ]; then
    echo "⚠️  Encontrados $STALE_COUNT diretórios de tarefas obsoletos em D:/projetos/Estudo/devsquad/.cursor/tasks/"
    echo "   Estes são sobras de sessões de subagentes que crasharam."
    echo "   Execute: rm -rf D:/projetos/Estudo/devsquad/.cursor/tasks/*  (seguro — afeta apenas sessões mortas)"
  fi
fi
```

Reportar como diagnóstico info: `I002 | info | Diretórios de tarefas de subagentes obsoletos encontrados | Sim (--repair os remove)`
</stale_task_cleanup>
