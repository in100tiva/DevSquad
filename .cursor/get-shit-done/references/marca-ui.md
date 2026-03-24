<ui_patterns>

Padrões visuais para saída GSD voltada ao usuário. Orquestradores referenciam este arquivo com @.

## Banners de Etapa

Usar para transições principais de workflow.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► {NOME DA ETAPA}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Nomes de etapa (maiúsculas):**
- `QUESTIONAMENTO`
- `PESQUISANDO`
- `DEFININDO REQUISITOS`
- `CRIANDO ROADMAP`
- `PLANEJANDO FASE {N}`
- `EXECUTANDO ONDA {N}`
- `VERIFICANDO`
- `FASE {N} COMPLETA ✓`
- `MILESTONE COMPLETO 🎉`

---

## Caixas de Ponto de Verificação

Ação do usuário necessária. Largura de 62 caracteres.

```
╔══════════════════════════════════════════════════════════════╗
║  PONTO DE VERIFICAÇÃO: {Tipo}                                ║
╚══════════════════════════════════════════════════════════════╝

{Conteúdo}

──────────────────────────────────────────────────────────────
→ {PROMPT DE AÇÃO}
──────────────────────────────────────────────────────────────
```

**Tipos:**
- `PONTO DE VERIFICAÇÃO: Verificação Necessária` → `→ Digite "aprovado" ou descreva problemas`
- `PONTO DE VERIFICAÇÃO: Decisão Necessária` → `→ Selecione: opção-a / opção-b`
- `PONTO DE VERIFICAÇÃO: Ação Necessária` → `→ Digite "pronto" quando completar`

---

## Símbolos de Status

```
✓  Completo / Passou / Verificado
✗  Falhou / Ausente / Bloqueado
◆  Em Progresso
○  Pendente
⚡ Auto-aprovado
⚠  Aviso
🎉 Milestone completo (apenas em banner)
```

---

## Exibição de Progresso

**Nível de fase/milestone:**
```
Progresso: ████████░░ 80%
```

**Nível de tarefa:**
```
Tarefas: 2/4 completas
```

**Nível de plano:**
```
Planos: 3/5 completos
```

---

## Indicadores de Criação de Agente

```
◆ Criando pesquisador...

◆ Criando 4 pesquisadores em paralelo...
  → Pesquisa de Stack
  → Pesquisa de Funcionalidades
  → Pesquisa de Arquitetura
  → Pesquisa de Armadilhas

✓ Pesquisador completo: STACK.md escrito
```

---

## Bloco Próximo

Sempre ao final de conclusões principais.

```
───────────────────────────────────────────────────────────────

## ▶ Próximo

**{Identificador}: {Nome}** — {descrição de uma linha}

`{comando para copiar e colar}`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

───────────────────────────────────────────────────────────────

**Também disponível:**
- `/gsd-alternativa-1` — descrição
- `/gsd-alternativa-2` — descrição

───────────────────────────────────────────────────────────────
```

---

## Caixa de Erro

```
╔══════════════════════════════════════════════════════════════╗
║  ERRO                                                        ║
╚══════════════════════════════════════════════════════════════╝

{Descrição do erro}

**Para corrigir:** {Passos de resolução}
```

---

## Tabelas

```
| Fase  | Status | Planos | Progresso |
|-------|--------|--------|-----------|
| 1     | ✓      | 3/3    | 100%      |
| 2     | ◆      | 1/4    | 25%       |
| 3     | ○      | 0/2    | 0%        |
```

---

## Anti-Padrões

- Variar largura de caixas/banners
- Misturar estilos de banner (`===`, `---`, `***`)
- Pular prefixo `GSD ►` em banners
- Emoji aleatório (`🚀`, `✨`, `💫`)
- Bloco Próximo ausente após conclusões

</ui_patterns>
