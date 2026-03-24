<purpose>
Orquestrar agentes de depuração paralelos para investigar lacunas de TAU e encontrar causas raiz.

Após TAU encontrar lacunas, invocar um agente de depuração por lacuna. Cada agente investiga autonomamente com sintomas pré-preenchidos do TAU. Coletar causas raiz, atualizar lacunas do TAU.md com diagnóstico, e então encaminhar para plan-phase --gaps com diagnósticos reais.

O orquestrador se mantém enxuto: analisar lacunas, invocar agentes, coletar resultados, atualizar TAU.
</purpose>

<paths>
DEBUG_DIR=.planning/debug

Arquivos de depuração usam o caminho `.planning/debug/` (diretório oculto com ponto no início).
</paths>

<core_principle>
**Diagnosticar antes de planejar correções.**

O TAU nos diz O QUE está quebrado (sintomas). Agentes de depuração encontram POR QUE (causa raiz). O plan-phase --gaps então cria correções direcionadas baseadas em causas reais, não suposições.

Sem diagnóstico: "Comentário não atualiza" → chutar correção → talvez errado
Com diagnóstico: "Comentário não atualiza" → "useEffect sem dependência" → correção precisa
</core_principle>

<process>

<step name="parse_gaps">
**Extrair lacunas do TAU.md:**

Ler a seção "Lacunas" (formato YAML):
```yaml
- truth: "Comentário aparece imediatamente após envio"
  status: failed
  reason: "Usuário reportou: funciona mas não aparece até eu atualizar a página"
  severity: major
  test: 2
  artifacts: []
  missing: []
```

Para cada lacuna, também ler o teste correspondente da seção "Testes" para obter contexto completo.

Construir lista de lacunas:
```
gaps = [
  {truth: "Comentário aparece imediatamente...", severity: "major", test_num: 2, reason: "..."},
  {truth: "Botão de resposta posicionado corretamente...", severity: "minor", test_num: 5, reason: "..."},
  ...
]
```
</step>

<step name="report_plan">
**Reportar plano de diagnóstico ao usuário:**

```
## Diagnosticando {N} Lacunas

Invocando agentes de depuração paralelos para investigar causas raiz:

| Lacuna (Verdade) | Severidade |
|-------------------|------------|
| Comentário aparece imediatamente após envio | major |
| Botão de resposta posicionado corretamente | minor |
| Deletar remove comentário | blocker |

Cada agente irá:
1. Criar DEBUG-{slug}.md com sintomas pré-preenchidos
2. Investigar autonomamente (ler código, formar hipóteses, testar)
3. Retornar causa raiz

Isto executa em paralelo - todas as lacunas investigadas simultaneamente.
```
</step>

<step name="spawn_agents">
**Invocar agentes de depuração em paralelo:**

Para cada lacuna, preencher o template debug-subagent-prompt e invocar:

```
Task(
  prompt=filled_debug_subagent_prompt + "\n\n<files_to_read>\n- {phase_dir}/{phase_num}-UAT.md\n- .planning/STATE.md\n</files_to_read>",
  subagent_type="gsd-depurador",
  isolation="worktree",
  description="Depurar: {truth_short}"
)
```

**Todos os agentes invocados em mensagem única** (execução paralela).

Placeholders do template:
- `{truth}`: O comportamento esperado que falhou
- `{expected}`: Do teste TAU
- `{actual}`: Descrição literal do usuário do campo reason
- `{errors}`: Quaisquer mensagens de erro do TAU (ou "Nenhum reportado")
- `{reproduction}`: "Teste {test_num} no TAU"
- `{timeline}`: "Descoberto durante TAU"
- `{goal}`: `find_root_cause_only` (fluxo TAU - plan-phase --gaps trata correções)
- `{slug}`: Gerado a partir de truth
</step>

<step name="collect_results">
**Coletar causas raiz dos agentes:**

Cada agente retorna com:
```
## CAUSA RAIZ ENCONTRADA

**Sessão de Depuração:** ${DEBUG_DIR}/{slug}.md

**Causa Raiz:** {causa específica com evidência}

**Resumo de Evidências:**
- {descoberta chave 1}
- {descoberta chave 2}
- {descoberta chave 3}

**Arquivos Envolvidos:**
- {arquivo1}: {o que está errado}
- {arquivo2}: {problema relacionado}

**Direção de Correção Sugerida:** {dica breve para plan-phase --gaps}
```

Analisar cada retorno para extrair:
- root_cause: A causa diagnosticada
- files: Arquivos envolvidos
- debug_path: Caminho para o arquivo da sessão de depuração
- suggested_fix: Dica para plano de fechamento de lacunas

Se o agente retornar `## INVESTIGAÇÃO INCONCLUSIVA`:
- root_cause: "Investigação inconclusiva - revisão manual necessária"
- Anotar qual problema precisa de atenção manual
- Incluir possibilidades restantes do retorno do agente
</step>

<step name="update_uat">
**Atualizar lacunas do TAU.md com diagnóstico:**

Para cada lacuna na seção Lacunas, adicionar campos artifacts e missing:

```yaml
- truth: "Comentário aparece imediatamente após envio"
  status: failed
  reason: "Usuário reportou: funciona mas não aparece até eu atualizar a página"
  severity: major
  test: 2
  root_cause: "useEffect no CommentList.tsx sem dependência commentCount"
  artifacts:
    - path: "src/components/CommentList.tsx"
      issue: "useEffect sem dependência"
  missing:
    - "Adicionar commentCount ao array de dependências do useEffect"
    - "Disparar re-render quando novo comentário é adicionado"
  debug_session: .planning/debug/comment-not-refreshing.md
```

Atualizar status no frontmatter para "diagnosed".

Commitar o TAU.md atualizado:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase_num}): add root causes from diagnosis" --files ".planning/phases/XX-name/{phase_num}-UAT.md"
```
</step>

<step name="report_results">
**Reportar resultados do diagnóstico e encaminhar:**

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DIAGNÓSTICO COMPLETO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Lacuna (Verdade) | Causa Raiz | Arquivos |
|-------------------|------------|----------|
| Comentário aparece imediatamente | useEffect sem dependência | CommentList.tsx |
| Botão de resposta posicionado corretamente | Ordem flex CSS incorreta | ReplyButton.tsx |
| Deletar remove comentário | API sem header de auth | api/comments.ts |

Sessões de depuração: ${DEBUG_DIR}/

Prosseguindo para planejar correções...
```

Retornar ao orquestrador verify-work para planejamento automático.
NÃO oferecer próximos passos manuais - verify-work cuida do resto.
</step>

</process>

<context_efficiency>
Agentes começam com sintomas pré-preenchidos do TAU (sem coleta de sintomas).
Agentes apenas diagnosticam — plan-phase --gaps trata correções (sem aplicação de correção).
</context_efficiency>

<failure_handling>
**Agente falha em encontrar causa raiz:**
- Marcar lacuna como "necessita revisão manual"
- Continuar com outras lacunas
- Reportar diagnóstico incompleto

**Agente expira:**
- Verificar DEBUG-{slug}.md para progresso parcial
- Pode retomar com /gsd-debug

**Todos os agentes falham:**
- Algo sistêmico (permissões, git, etc.)
- Reportar para investigação manual
- Fallback para plan-phase --gaps sem causas raiz (menos preciso)
</failure_handling>

<success_criteria>
- [ ] Lacunas analisadas do TAU.md
- [ ] Agentes de depuração invocados em paralelo
- [ ] Causas raiz coletadas de todos os agentes
- [ ] Lacunas do TAU.md atualizadas com artifacts e missing
- [ ] Sessões de depuração salvas em ${DEBUG_DIR}/
- [ ] Encaminhamento para verify-work para planejamento automático
</success_criteria>
