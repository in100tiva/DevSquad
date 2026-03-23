---
phase: {N}
slug: {slug-da-fase}
status: draft
nyquist_compliant: false
wave_0_complete: false
created: {data}
---

# Fase {N} — Estratégia de Validação

> Contrato de validação por fase para amostragem de feedback durante execução.

---

## Infraestrutura de Testes

| Propriedade | Valor |
|-------------|-------|
| **Framework** | {pytest 7.x / jest 29.x / vitest / go test / outro} |
| **Arquivo de config** | {caminho ou "nenhum — Wave 0 instala"} |
| **Comando rápido** | `{comando rápido}` |
| **Comando suite completa** | `{comando completo}` |
| **Tempo estimado** | ~{N} segundos |

---

## Taxa de Amostragem

- **Após cada commit de tarefa:** Executar `{comando rápido}`
- **Após cada onda de plano:** Executar `{comando suite completa}`
- **Antes do `/gsd-verify-work`:** Suite completa deve estar verde
- **Latência máxima de feedback:** {N} segundos

---

## Mapa de Verificação por Tarefa

| ID da Tarefa | Plano | Onda | Requisito | Tipo de Teste | Comando Automatizado | Arquivo Existe | Status |
|--------------|-------|------|-----------|---------------|---------------------|----------------|--------|
| {N}-01-01 | 01 | 1 | REQ-{XX} | unit | `{comando}` | ✅ / ❌ W0 | ⬜ pendente |

*Status: ⬜ pendente · ✅ verde · ❌ vermelho · ⚠️ instável*

---

## Requisitos da Wave 0

- [ ] `{tests/test_file.py}` — stubs para REQ-{XX}
- [ ] `{tests/conftest.py}` — fixtures compartilhados
- [ ] `{instalação do framework}` — se nenhum framework detectado

*Se nenhum: "Infraestrutura existente cobre todos os requisitos da fase."*

---

## Verificações Apenas Manuais

| Comportamento | Requisito | Por que Manual | Instruções de Teste |
|---------------|-----------|----------------|---------------------|
| {comportamento} | REQ-{XX} | {motivo} | {passos} |

*Se nenhum: "Todos os comportamentos da fase têm verificação automatizada."*

---

## Aprovação da Validação

- [ ] Todas as tarefas têm verify `<automated>` ou dependências da Wave 0
- [ ] Continuidade de amostragem: nenhuma sequência de 3 tarefas sem verify automatizado
- [ ] Wave 0 cobre todas as referências MISSING
- [ ] Sem flags de watch-mode
- [ ] Latência de feedback < {N}s
- [ ] `nyquist_compliant: true` definido no frontmatter

**Aprovação:** {pendente / aprovado AAAA-MM-DD}
