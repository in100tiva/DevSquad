# Template TAU

Template para `.planning/phases/XX-nome/{num_fase}-UAT.md` — rastreamento persistente de sessão TAU (Teste de Aceitação do Usuário).

---

## Template do Arquivo

```markdown
---
status: testing | partial | complete | diagnosed
phase: XX-nome
source: [lista de arquivos SUMMARY.md testados]
started: [timestamp ISO]
updated: [timestamp ISO]
---

## Teste Atual
<!-- SOBRESCREVA cada teste - mostra onde estamos -->

number: [N]
name: [nome do teste]
expected: |
  [o que o usuário deve observar]
awaiting: resposta do usuário

## Testes

### 1. [Nome do Teste]
expected: [comportamento observável - o que o usuário deve ver]
result: [pendente]

### 2. [Nome do Teste]
expected: [comportamento observável]
result: pass

### 3. [Nome do Teste]
expected: [comportamento observável]
result: issue
reported: "[resposta literal do usuário]"
severity: major

### 4. [Nome do Teste]
expected: [comportamento observável]
result: skipped
reason: [motivo do pulo]

### 5. [Nome do Teste]
expected: [comportamento observável]
result: blocked
blocked_by: server | physical-device | release-build | third-party | prior-phase
reason: [motivo do bloqueio]

...

## Resumo

total: [N]
passed: [N]
issues: [N]
pending: [N]
skipped: [N]
blocked: [N]

## Lacunas

<!-- Formato YAML para consumo do plan-phase --gaps -->
- truth: "[comportamento esperado do teste]"
  status: failed
  reason: "Usuário reportou: [resposta literal]"
  severity: blocker | major | minor | cosmetic
  test: [N]
  root_cause: ""     # Preenchido pelo diagnóstico
  artifacts: []      # Preenchido pelo diagnóstico
  missing: []        # Preenchido pelo diagnóstico
  debug_session: ""  # Preenchido pelo diagnóstico
```

---

<section_rules>

**Frontmatter:**
- `status`: SOBRESCREVER - "testing", "partial" ou "complete"
- `phase`: IMUTÁVEL - definido na criação
- `source`: IMUTÁVEL - arquivos SUMMARY sendo testados
- `started`: IMUTÁVEL - definido na criação
- `updated`: SOBRESCREVER - atualizar em cada alteração

**Teste Atual:**
- SOBRESCREVER inteiramente em cada transição de teste
- Mostra qual teste está ativo e o que está sendo aguardado
- Na conclusão: "[testes concluídos]"

**Testes:**
- Cada teste: SOBRESCREVER campo result quando usuário responder
- Valores de `result`: [pendente], pass, issue, skipped, blocked
- Se issue: adicionar `reported` (literal) e `severity` (inferido)
- Se skipped: adicionar `reason` se fornecido
- Se blocked: adicionar `blocked_by` (tag) e `reason` (se fornecido)

**Resumo:**
- SOBRESCREVER contagens após cada resposta
- Rastreia: total, passed, issues, pending, skipped

**Lacunas:**
- ADICIONAR apenas quando issue encontrado (formato YAML)
- Após diagnóstico: preencher `root_cause`, `artifacts`, `missing`, `debug_session`
- Esta seção alimenta diretamente /gsd-plan-phase --gaps

</section_rules>

<diagnosis_lifecycle>

**Após testes concluídos (status: complete), se lacunas existem:**

1. Usuário executa diagnóstico (da oferta do verify-work ou manualmente)
2. Workflow diagnose-issues dispara agentes de debug paralelos
3. Cada agente investiga uma lacuna, retorna causa raiz
4. Seção Lacunas do UAT.md atualizada com diagnóstico:
   - Cada lacuna recebe `root_cause`, `artifacts`, `missing`, `debug_session` preenchidos
5. status → "diagnosed"
6. Pronto para /gsd-plan-phase --gaps com causas raiz

**Após diagnóstico:**
```yaml
## Lacunas

- truth: "Comentário aparece imediatamente após submissão"
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
  debug_session: ".planning/debug/comentario-nao-atualiza.md"
```

</diagnosis_lifecycle>

<lifecycle>

**Criação:** Quando /gsd-verify-work inicia nova sessão
- Extrair testes dos arquivos SUMMARY.md
- Definir status como "testing"
- Teste Atual aponta para teste 1
- Todos os testes têm result: [pendente]

**Durante testes:**
- Apresentar teste da seção Teste Atual
- Usuário responde com confirmação de pass ou descrição do problema
- Atualizar resultado do teste (pass/issue/skipped)
- Atualizar contagens do Resumo
- Se issue: adicionar à seção Lacunas (formato YAML), inferir severidade
- Mover Teste Atual para próximo teste pendente

**Na conclusão:**
- status → "complete"
- Teste Atual → "[testes concluídos]"
- Commitar arquivo
- Apresentar resumo com próximos passos

**Conclusão parcial:**
- status → "partial" (se testes pendentes, bloqueados ou skipped não resolvidos permanecem)
- Teste Atual → "[testes pausados — {N} itens pendentes]"
- Commitar arquivo
- Apresentar resumo com itens pendentes destacados

**Retomando sessão parcial:**
- `/gsd-verify-work {fase}` retoma do primeiro teste pendente/bloqueado
- Quando todos os itens resolvidos, status avança para "complete"

**Retomar após /clear:**
1. Ler frontmatter → saber fase e status
2. Ler Teste Atual → saber onde estamos
3. Encontrar primeiro resultado [pendente] → continuar daí
4. Resumo mostra progresso até o momento

</lifecycle>

<severity_guide>

Severidade é INFERIDA da linguagem natural do usuário, nunca perguntada.

| Usuário descreve | Inferir |
|------------------|---------|
| Crash, erro, exceção, falha completa, inutilizável | blocker |
| Não funciona, nada acontece, comportamento errado, ausente | major |
| Funciona mas..., lento, estranho, menor, pequeno problema | minor |
| Cor, fonte, espaçamento, alinhamento, visual, aparência estranha | cosmetic |

Padrão: **major** (padrão seguro, usuário pode esclarecer se errado)

</severity_guide>

<good_example>
```markdown
---
status: diagnosed
phase: 04-comentarios
source: 04-01-SUMMARY.md, 04-02-SUMMARY.md
started: 2025-01-15T10:30:00Z
updated: 2025-01-15T10:45:00Z
---

## Teste Atual

[testes concluídos]

## Testes

### 1. Ver Comentários no Post
expected: Seção de comentários expande, mostra contagem e lista de comentários
result: pass

### 2. Criar Comentário de Nível Superior
expected: Submeter comentário via editor rich text, aparece na lista com info do autor
result: issue
reported: "funciona mas não aparece até eu atualizar a página"
severity: major

### 3. Responder a um Comentário
expected: Clicar Responder, compositor inline aparece, submeter mostra resposta aninhada
result: pass

### 4. Aninhamento Visual
expected: Thread de 3+ níveis mostra indentação, bordas esquerdas, limita em profundidade razoável
result: pass

### 5. Deletar Próprio Comentário
expected: Clicar deletar no próprio comentário, removido ou mostra [deletado] se tem respostas
result: pass

### 6. Contagem de Comentários
expected: Post mostra contagem precisa, incrementa ao adicionar comentário
result: pass

## Resumo

total: 6
passed: 5
issues: 1
pending: 0
skipped: 0

## Lacunas

- truth: "Comentário aparece imediatamente após submissão na lista"
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
  debug_session: ".planning/debug/comentario-nao-atualiza.md"
```
</good_example>
