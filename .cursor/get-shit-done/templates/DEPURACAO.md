# Template de Debug

Template para `.planning/debug/[slug].md` — rastreamento de sessão de debug ativa.

---

## Template do Arquivo

```markdown
---
status: gathering | investigating | fixing | verifying | awaiting_human_verify | resolved
trigger: "[input literal do usuário]"
created: [timestamp ISO]
updated: [timestamp ISO]
---

## Foco Atual
<!-- SOBRESCREVA em cada atualização - sempre reflete AGORA -->

hypothesis: [teoria atual sendo testada]
test: [como está testando]
expecting: [o que resultado significa se verdadeiro/falso]
next_action: [próximo passo imediato]

## Sintomas
<!-- Escrito durante coleta, depois imutável -->

expected: [o que deveria acontecer]
actual: [o que realmente acontece]
errors: [mensagens de erro se houver]
reproduction: [como reproduzir]
started: [quando quebrou / sempre esteve quebrado]

## Eliminados
<!-- ADICIONAR apenas - evita re-investigação após /clear -->

- hypothesis: [teoria que estava errada]
  evidence: [o que refutou]
  timestamp: [quando eliminado]

## Evidências
<!-- ADICIONAR apenas - fatos descobertos durante investigação -->

- timestamp: [quando encontrado]
  checked: [o que foi examinado]
  found: [o que foi observado]
  implication: [o que isto significa]

## Resolução
<!-- SOBRESCREVER conforme entendimento evolui -->

root_cause: [vazio até encontrar]
fix: [vazio até aplicar]
verification: [vazio até verificar]
files_changed: []
```

---

<section_rules>

**Frontmatter (status, trigger, timestamps):**
- `status`: SOBRESCREVER - reflete fase atual
- `trigger`: IMUTÁVEL - input literal do usuário, nunca muda
- `created`: IMUTÁVEL - definido uma vez
- `updated`: SOBRESCREVER - atualizar em cada alteração

**Foco Atual:**
- SOBRESCREVER inteiramente em cada atualização
- Sempre reflete o que o Claude está fazendo AGORA
- Se Claude lê isto após /clear, sabe exatamente onde retomar
- Campos: hypothesis, test, expecting, next_action

**Sintomas:**
- Escrito durante fase inicial de coleta
- IMUTÁVEL após coleta completa
- Ponto de referência para o que estamos tentando corrigir
- Campos: expected, actual, errors, reproduction, started

**Eliminados:**
- ADICIONAR apenas - nunca remover entradas
- Evita re-investigação de becos sem saída após reset de contexto
- Cada entrada: hipótese, evidência que refutou, timestamp
- Crítico para eficiência entre limites de /clear

**Evidências:**
- ADICIONAR apenas - nunca remover entradas
- Fatos descobertos durante investigação
- Cada entrada: timestamp, o que verificou, o que encontrou, implicação
- Constrói o caso para causa raiz

**Resolução:**
- SOBRESCREVER conforme entendimento evolui
- Pode atualizar múltiplas vezes conforme correções são tentadas
- Estado final mostra causa raiz confirmada e correção verificada
- Campos: root_cause, fix, verification, files_changed

</section_rules>

<lifecycle>

**Criação:** Imediatamente quando /gsd-debug é chamado
- Criar arquivo com trigger do input do usuário
- Definir status como "gathering"
- Foco Atual: next_action = "coletar sintomas"
- Sintomas: vazio, a ser preenchido

**Durante coleta de sintomas:**
- Atualizar seção Sintomas conforme usuário responde perguntas
- Atualizar Foco Atual com cada pergunta
- Quando completo: status → "investigating"

**Durante investigação:**
- SOBRESCREVER Foco Atual com cada hipótese
- ADICIONAR a Evidências com cada descoberta
- ADICIONAR a Eliminados quando hipótese é refutada
- Atualizar timestamp no frontmatter

**Durante correção:**
- status → "fixing"
- Atualizar Resolution.root_cause quando confirmado
- Atualizar Resolution.fix quando aplicado
- Atualizar Resolution.files_changed

**Durante verificação:**
- status → "verifying"
- Atualizar Resolution.verification com resultados
- Se verificação falha: status → "investigating", tentar novamente

**Após auto-verificação passar:**
- status → "awaiting_human_verify"
- Solicitar confirmação explícita do usuário em um checkpoint
- NÃO mover arquivo para resolvido ainda

**Na resolução:**
- status → "resolved"
- Mover arquivo para .planning/debug/resolved/ (somente após usuário confirmar correção)

</lifecycle>

<resume_behavior>

Quando Claude lê este arquivo após /clear:

1. Parsear frontmatter → saber status
2. Ler Foco Atual → saber exatamente o que estava acontecendo
3. Ler Eliminados → saber o que NÃO tentar novamente
4. Ler Evidências → saber o que foi aprendido
5. Continuar do next_action

O arquivo É o cérebro do debug. Claude deve ser capaz de retomar perfeitamente de qualquer ponto de interrupção.

</resume_behavior>

<size_constraint>

Mantenha arquivos de debug focados:
- Entradas de evidência: 1-2 linhas cada, apenas os fatos
- Eliminados: breve - hipótese + por que falhou
- Sem prosa narrativa - apenas dados estruturados

Se evidências crescerem muito (10+ entradas), considere se está andando em círculos. Verifique Eliminados para garantir que não está re-percorrendo.

</size_constraint>
