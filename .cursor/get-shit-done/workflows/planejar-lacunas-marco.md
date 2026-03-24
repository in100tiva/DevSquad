<purpose>
Criar todas as fases necessárias para fechar lacunas identificadas pelo `/gsd-auditar-marco`. Lê o MILESTONE-AUDIT.md, agrupa lacunas em fases lógicas, cria entradas de fase no ROADMAP.md e oferece planejar cada fase. Um comando cria todas as fases de correção — sem `/gsd-adicionar-fase` manual por lacuna.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 1. Carregar Resultados da Auditoria

```bash
# Encontrar o arquivo de auditoria mais recente
ls -t .planning/v*-MILESTONE-AUDIT.md 2>/dev/null | head -1
```

Analisar frontmatter YAML para extrair lacunas estruturadas:
- `gaps.requirements` — requisitos não satisfeitos
- `gaps.integration` — conexões entre fases faltando
- `gaps.flows` — fluxos E2E quebrados

Se nenhum arquivo de auditoria existir ou não tiver lacunas, erro:
```
Nenhuma lacuna de auditoria encontrada. Execute `/gsd-auditar-marco` primeiro.
```

## 2. Priorizar Lacunas

Agrupar lacunas por prioridade do REQUIREMENTS.md:

| Prioridade | Ação |
|------------|------|
| `must` | Criar fase, bloqueia marco |
| `should` | Criar fase, recomendado |
| `nice` | Perguntar ao usuário: incluir ou adiar? |

Para lacunas de integração/fluxo, inferir prioridade dos requisitos afetados.

## 3. Agrupar Lacunas em Fases

Agrupar lacunas relacionadas em fases lógicas:

**Regras de agrupamento:**
- Mesma fase afetada → combinar em uma fase de correção
- Mesmo subsistema (auth, API, UI) → combinar
- Ordem de dependência (corrigir stubs antes de fiação)
- Manter fases focadas: 2-4 tarefas cada

**Exemplo de agrupamento:**
```
Lacuna: DASH-01 não satisfeito (Dashboard não busca dados)
Lacuna: Integração Fase 1→3 (Auth não passado para chamadas API)
Lacuna: Fluxo "Ver dashboard" quebra na busca de dados

→ Fase 6: "Conectar Dashboard à API"
  - Adicionar fetch no Dashboard.tsx
  - Incluir header de auth no fetch
  - Tratar resposta, atualizar estado
  - Renderizar dados do usuário
```

## 4. Determinar Números de Fase

Encontrar a fase existente mais alta:
```bash
# Obter lista ordenada de fases, extrair última
HIGHEST=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phases list --pick directories[-1])
```

Novas fases continuam a partir daí:
- Se Fase 5 é a mais alta, lacunas se tornam Fase 6, 7, 8...

## 5. Apresentar Plano de Fechamento de Lacunas

```markdown
## Plano de Fechamento de Lacunas

**Marco:** {versão}
**Lacunas a fechar:** {N} requisitos, {M} integração, {K} fluxos

### Fases Propostas

**Fase {N}: {Nome}**
Fecha:
- {REQ-ID}: {descrição}
- Integração: {de} → {para}
Tarefas: {contagem}

**Fase {N+1}: {Nome}**
Fecha:
- {REQ-ID}: {descrição}
- Fluxo: {nome do fluxo}
Tarefas: {contagem}

{Se lacunas nice-to-have existirem:}

### Adiados (nice-to-have)

Estas lacunas são opcionais. Incluí-las?
- {descrição da lacuna}
- {descrição da lacuna}

---

Criar estas {X} fases? (sim / ajustar / adiar todos opcionais)
```

Aguardar confirmação do usuário.

## 6. Atualizar ROADMAP.md

Adicionar novas fases ao marco atual:

```markdown
### Fase {N}: {Nome}
**Objetivo:** {derivado das lacunas sendo fechadas}
**Requisitos:** {REQ-IDs sendo satisfeitos}
**Fechamento de Lacunas:** Fecha lacunas da auditoria

### Fase {N+1}: {Nome}
...
```

## 7. Atualizar Tabela de Rastreabilidade do REQUIREMENTS.md (OBRIGATÓRIO)

Para cada REQ-ID atribuído a uma fase de fechamento de lacunas:
- Atualizar a coluna Fase para refletir a nova fase de fechamento
- Resetar Status para `Pendente`

Resetar requisitos marcados que a auditoria encontrou como não satisfeitos:
- Mudar `[x]` → `[ ]` para qualquer requisito marcado como não satisfeito na auditoria
- Atualizar contagem de cobertura no topo do REQUIREMENTS.md

```bash
# Verificar que tabela de rastreabilidade reflete atribuições de fechamento de lacunas
grep -c "Pending" .planning/REQUIREMENTS.md
```

## 8. Criar Diretórios de Fase

```bash
mkdir -p ".planning/phases/{NN}-{nome}"
```

## 9. Commitar Atualização de Roteiro e Requisitos

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs(roadmap): add gap closure phases {N}-{M}" --files .planning/ROADMAP.md .planning/REQUIREMENTS.md
```

## 10. Oferecer Próximos Passos

```markdown
## ✓ Fases de Fechamento de Lacunas Criadas

**Fases adicionadas:** {N} - {M}
**Lacunas tratadas:** {contagem} requisitos, {contagem} integração, {contagem} fluxos

---

## ▶ Próximo Passo

**Planejar primeira fase de fechamento de lacunas**

`/gsd-planejar-fase {N}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- `/gsd-executar-fase {N}` — se planos já existirem
- `cat .planning/ROADMAP.md` — ver roteiro atualizado

---

**Após todas as fases de lacuna completas:**

`/gsd-auditar-marco` — re-auditar para verificar se lacunas foram fechadas
`/gsd-completar-marco {versão}` — arquivar quando auditoria aprovar
```

</process>

<gap_to_phase_mapping>

## Como Lacunas se Tornam Tarefas

**Lacuna de requisito → Tarefas:**
```yaml
gap:
  id: DASH-01
  description: "Usuário vê seus dados"
  reason: "Dashboard existe mas não busca da API"
  missing:
    - "useEffect com fetch para /api/user/data"
    - "Estado para dados do usuário"
    - "Renderizar dados do usuário no JSX"

becomes:

phase: "Conectar Dados do Dashboard"
tasks:
  - name: "Adicionar busca de dados"
    files: [src/components/Dashboard.tsx]
    action: "Adicionar useEffect que busca /api/user/data na montagem"

  - name: "Adicionar gerenciamento de estado"
    files: [src/components/Dashboard.tsx]
    action: "Adicionar useState para userData, loading, estados de erro"

  - name: "Renderizar dados do usuário"
    files: [src/components/Dashboard.tsx]
    action: "Substituir placeholder por renderização userData.map"
```

**Lacuna de integração → Tarefas:**
```yaml
gap:
  from_phase: 1
  to_phase: 3
  connection: "Token de auth → chamadas API"
  reason: "Chamadas API do dashboard não incluem header de auth"
  missing:
    - "Header de auth em chamadas fetch"
    - "Refresh de token em 401"

becomes:

phase: "Adicionar Auth às Chamadas API do Dashboard"
tasks:
  - name: "Adicionar header de auth aos fetches"
    files: [src/components/Dashboard.tsx, src/lib/api.ts]
    action: "Incluir header Authorization com token em todas as chamadas API"

  - name: "Tratar respostas 401"
    files: [src/lib/api.ts]
    action: "Adicionar interceptor para refresh de token ou redirecionar para login em 401"
```

**Lacuna de fluxo → Tarefas:**
```yaml
gap:
  name: "Usuário visualiza dashboard após login"
  broken_at: "Carga de dados do dashboard"
  reason: "Sem chamada fetch"
  missing:
    - "Buscar dados do usuário na montagem"
    - "Exibir estado de loading"
    - "Renderizar dados do usuário"

becomes:

# Geralmente mesma fase que lacuna de requisito/integração
# Lacunas de fluxo frequentemente se sobrepõem com outros tipos de lacuna
```

</gap_to_phase_mapping>

<success_criteria>
- [ ] MILESTONE-AUDIT.md carregado e lacunas analisadas
- [ ] Lacunas priorizadas (must/should/nice)
- [ ] Lacunas agrupadas em fases lógicas
- [ ] Usuário confirmou plano de fases
- [ ] ROADMAP.md atualizado com novas fases
- [ ] Tabela de rastreabilidade do REQUIREMENTS.md atualizada com atribuições de fase de fechamento
- [ ] Checkboxes de requisitos não satisfeitos resetados (`[x]` → `[ ]`)
- [ ] Contagem de cobertura atualizada no REQUIREMENTS.md
- [ ] Diretórios de fase criados
- [ ] Mudanças commitadas (inclui REQUIREMENTS.md)
- [ ] Usuário sabe executar `/gsd-planejar-fase` em seguida
</success_criteria>
