# Cora Norman — Modo: Refatorar

> Correções cirúrgicas guiadas pelos princípios de Norman.
> Cora prioriza pelo tipo de falha: mistakes antes de slips, slips antes de polish.

---

## Lógica de priorização de Cora

```
PRIORIDADE 1 — Mistakes (modelo conceitual errado)
  Por quê: o usuário não aprende com o erro — repete indefinidamente
  Sintoma: "mas eu pensei que era assim que funcionava"

PRIORIDADE 2 — Slips com consequências irreversíveis
  Por quê: o usuário sabe o que quer fazer mas o sistema não protege
  Sintoma: "deletei sem querer e não tem como voltar"

PRIORIDADE 3 — Slips com alta frequência
  Por quê: mesmo sendo recuperável, gera atrito constante
  Sintoma: "sempre erro nesse campo"

PRIORIDADE 4 — Gulf of execution (discoverability)
  Por quê: feature existe mas não é usada
  Sintoma: "não sabia que isso era possível"

PRIORIDADE 5 — Gulf of evaluation (feedback)
  Por quê: usuário não sabe o estado do sistema
  Sintoma: "não sei se funcionou"
```

---

## Playbook de refatorações por princípio

### RN-01 — Feedback ausente ou insuficiente

**Princípio:** Feedback
**Tipo de erro gerado:** Slip (usuário repete ação por não saber que funcionou)

**Diagnóstico rápido:**
- Botão sem loading state
- Ação completada sem confirmação visual
- Erro exibido só no topo da página sem highlight no campo
- Processo longo sem progress indicator

**Refatoração padrão:**

```jsx
// ANTES — sem feedback
<button onClick={handleSave}>Salvar</button>

// DEPOIS — feedback em 3 estados
<button
  onClick={handleSave}
  disabled={isSaving}
  className={isSaving ? 'btn-loading' : 'btn-primary'}
>
  {isSaving ? (
    <><Spinner size="sm" /> Salvando...</>
  ) : saved ? (
    <><CheckIcon /> Salvo</>
  ) : (
    'Salvar alterações'
  )}
</button>

// Toast de confirmação
{saved && (
  <Toast variant="success" autoDismiss={4000}>
    Alterações salvas com sucesso.
  </Toast>
)}
```

**Princípio aplicado:** Feedback imediato, informativo e proporcional.

---

### RN-02 — Signifier ausente em ação crítica

**Princípio:** Signifiers
**Tipo de erro gerado:** Slip (usuário não encontra a ação) ou Mistake (não sabe o que vai acontecer)

**Diagnóstico rápido:**
- Ícone sem label em ação importante
- Ação acessível só via hover sem indicação
- CTA com verbo genérico ("OK", "Confirmar", "Continuar")

**Refatoração padrão:**

```jsx
// ANTES — ícone ambíguo, sem label
<button aria-label="settings">
  <GearIcon />
</button>

// DEPOIS — ícone + label + tooltip
<button className="btn-icon-label" title="Configurações do projeto">
  <GearIcon />
  <span>Configurações</span>
</button>

// ---

// ANTES — CTA genérico em modal de delete
<button variant="danger">Confirmar</button>

// DEPOIS — CTA específico que descreve a ação e consequência
<button variant="danger">
  Deletar proposta permanentemente
</button>
// Subtexto no modal: "Essa ação não pode ser desfeita."
```

---

### RN-03 — Constraint ausente para ação destrutiva

**Princípio:** Constraints
**Tipo de erro gerado:** Slip com consequência irreversível

**Diagnóstico rápido:**
- Delete sem confirm dialog
- Delete permanente sem período de recuperação
- Ação destrutiva adjacente a ação primária

**Refatoração padrão:**

```jsx
// ANTES — delete direto
const handleDelete = async (id) => {
  await api.delete(`/projects/${id}`)
  refetch()
}

// DEPOIS — soft delete + confirm + undo
const handleDelete = async (id, name) => {
  // 1. Confirmação antes da ação
  const confirmed = await showConfirmDialog({
    title: `Deletar "${name}"?`,
    description: 'Esse projeto e todos os seus dados serão removidos permanentemente.',
    confirmLabel: 'Deletar permanentemente',
    cancelLabel: 'Cancelar',
    variant: 'danger'
  })
  if (!confirmed) return

  // 2. Soft delete — mantém por 30 dias
  await api.patch(`/projects/${id}`, { deleted_at: new Date() })

  // 3. Toast com undo
  showToast({
    message: `"${name}" foi deletado.`,
    action: { label: 'Desfazer', onClick: () => api.patch(`/projects/${id}`, { deleted_at: null }) },
    duration: 8000
  })

  refetch()
}
```

---

### RN-04 — Modelo conceitual inconsistente (terminologia)

**Princípio:** Modelo Conceitual
**Tipo de erro gerado:** Mistake (usuário aprende errado e transfere o erro)

**Diagnóstico rápido:**
- Mesmo conceito com nomes diferentes em partes do produto
- Ação com comportamento diferente em contextos equivalentes
- Metáfora quebrada (pasta que não é hierárquica, projeto que não agrupa)

**Refatoração padrão:**

```
// Auditoria de terminologia — o que Cora faz primeiro

1. Listar todos os nomes para cada conceito central do produto:
   Conceito: "espaço de trabalho do usuário"
   Ocorrências: "Workspace" (nav), "Área" (settings), "Organização" (API), "Board" (docs)
   → Escolher UM nome e aplicar em todos os lugares

2. Criar glossário interno:
   | Conceito        | Nome correto | Nomes a remover             |
   |---|---|---|
   | Agrupador raiz  | Workspace    | Área, Organização, Board    |
   | Unidade de trabalho | Projeto  | Task, Board, Item           |
   | Membro convidado | Colaborador | Guest, Member, User         |

3. Refatorar na ordem:
   a. UI (labels, botões, headings)
   b. Copy (tooltips, empty states, onboarding)
   c. Emails transacionais
   d. Documentação / help center
   e. API (breaking change — versionar adequadamente)
```

---

### RN-05 — Gulf of Execution — feature não descoberta

**Princípio:** Discoverability
**Tipo de erro gerado:** Mistake (usuário não sabe que a ação existe)

**Diagnóstico rápido:**
- Feature core acessível só por atalho de teclado não documentado
- Ação importante escondida em menu de contexto sem indicação
- Funcionalidade nova lançada sem surfacing no produto

**Refatoração padrão:**

```jsx
// ANTES — ação importante só no right-click (sem indicação)
<tr onContextMenu={showContextMenu}>
  {/* dados da linha */}
</tr>

// DEPOIS — ação visível + complementada por right-click
<tr>
  {/* dados da linha */}
  <td className="actions-cell">
    {/* Ações inline visíveis no hover */}
    <div className="row-actions" role="group">
      <button onClick={handleEdit} title="Editar">
        <EditIcon /><span className="sr-only">Editar</span>
      </button>
      <button onClick={handleArchive} title="Arquivar">
        <ArchiveIcon /><span className="sr-only">Arquivar</span>
      </button>
      {/* Menu de mais ações */}
      <button onClick={showContextMenu} title="Mais ações">
        <DotsIcon /><span className="sr-only">Mais ações</span>
      </button>
    </div>
  </td>
</tr>
```

---

### RN-06 — Mapping ruim — controle distante do efeito

**Princípio:** Mapping
**Tipo de erro gerado:** Slip (usuário clica no elemento errado)

**Diagnóstico rápido:**
- Botão de ação no topo/rodapé que afeta item selecionado na lista
- Configuração de um item em área global de settings
- Ação que afeta um elemento posicionada em outro elemento

**Refatoração padrão:**

```jsx
// ANTES — ação global que afeta seleção local
<PageHeader>
  <h1>Projetos</h1>
  <button onClick={deleteSelected} disabled={!selected.length}>
    Deletar selecionados
  </button>
</PageHeader>

<ProjectsList
  selected={selected}
  onSelect={setSelected}
/>

// DEPOIS — ação contextual próxima à seleção
<ProjectsList
  selected={selected}
  onSelect={setSelected}
/>

{/* Toolbar aparece ABAIXO da lista quando há seleção */}
{selected.length > 0 && (
  <SelectionToolbar>
    <span>{selected.length} projetos selecionados</span>
    <button onClick={archiveSelected}>Arquivar</button>
    <button onClick={deleteSelected} variant="danger">Deletar</button>
    <button onClick={clearSelection}>Cancelar seleção</button>
  </SelectionToolbar>
)}
```

---

## Sequência de entrega de Cora em uma refatoração

Para cada problema encontrado, Cora entrega na ordem:

```markdown
### [RN-XX] Nome do problema

**Princípio de Norman:** [Princípio específico]
**Tipo de erro:** Slip / Mistake
**Estágio da ação afetado:** [1-7]
**Impacto:** [Alto/Médio] — [comportamento observável]

**Antes (o problema):**
[código ou descrição do estado atual]

**Depois (a correção):**
[código ou descrição da solução]

**Por que funciona:**
[Qual princípio está sendo aplicado e como resolve o problema específico]

**Métrica de validação:**
[Como saber que a correção funcionou — o que observar no comportamento do usuário]
```
