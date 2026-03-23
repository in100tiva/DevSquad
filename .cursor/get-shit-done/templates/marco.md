# Template de Entrada de Marco

Adicione esta entrada ao `.planning/MILESTONES.md` ao completar um marco:

```markdown
## v[X.Y] [Nome] (Entregue: AAAA-MM-DD)

**Entregue:** [Uma frase descrevendo o que foi entregue]

**Fases completadas:** [X-Y] ([Z] planos no total)

**Principais conquistas:**
- [Conquista principal 1]
- [Conquista principal 2]
- [Conquista principal 3]
- [Conquista principal 4]

**Estatísticas:**
- [X] arquivos criados/modificados
- [Y] linhas de código (linguagem principal)
- [Z] fases, [N] planos, [M] tarefas
- [D] dias do início à entrega (ou de marco a marco)

**Intervalo Git:** `feat(XX-XX)` → `feat(YY-YY)`

**Próximo passo:** [Breve descrição dos objetivos do próximo marco, ou "Projeto concluído"]

---
```

<structure>
Se MILESTONES.md não existir, crie com o cabeçalho:

```markdown
# Marcos do Projeto: [Nome do Projeto]

[Entradas em ordem cronológica reversa - mais recente primeiro]
```
</structure>

<guidelines>
**Quando criar marcos:**
- MVP v1.0 inicial entregue
- Lançamentos de versões principais (v2.0, v3.0)
- Marcos de funcionalidades significativas (v1.1, v1.2)
- Antes de arquivar o planejamento (capturar o que foi entregue)

**Não crie marcos para:**
- Conclusões de fases individuais (fluxo normal)
- Trabalho em progresso (espere até entregar)
- Correções menores que não constituem um release

**Estatísticas a incluir:**
- Contar arquivos modificados: `git diff --stat feat(XX-XX)..feat(YY-YY) | tail -1`
- Contar LOC: `find . -name "*.swift" -o -name "*.ts" | xargs wc -l` (ou extensão relevante)
- Contagens de fases/planos/tarefas do ROADMAP
- Linha do tempo do primeiro commit da fase ao último commit da fase

**Formato do intervalo Git:**
- Primeiro commit do marco → último commit do marco
- Exemplo: `feat(01-01)` → `feat(04-01)` para fases 1-4
</guidelines>

<example>
```markdown
# Marcos do Projeto: WeatherBar

## v1.1 Segurança & Polimento (Entregue: 2025-12-10)

**Entregue:** Fortalecimento de segurança com integração Keychain e tratamento abrangente de erros

**Fases completadas:** 5-6 (3 planos no total)

**Principais conquistas:**
- Migração do armazenamento de chave API de texto plano para macOS Keychain
- Implementação de tratamento abrangente de erros para falhas de rede
- Integração de relatórios de crash com Sentry
- Correção de vazamento de memória no timer de atualização automática

**Estatísticas:**
- 23 arquivos modificados
- 650 linhas de Swift adicionadas
- 2 fases, 3 planos, 12 tarefas
- 8 dias da v1.0 à v1.1

**Intervalo Git:** `feat(05-01)` → `feat(06-02)`

**Próximo passo:** Redesign SwiftUI v2.0 com suporte a widgets

---

## v1.0 MVP (Entregue: 2025-11-25)

**Entregue:** App de clima na barra de menu com condições atuais e previsão de 3 dias

**Fases completadas:** 1-4 (7 planos no total)

**Principais conquistas:**
- App na barra de menu com popover UI (AppKit)
- Integração com API OpenWeather com atualização automática
- Exibição do clima atual com ícone de condições
- Lista de previsão de 3 dias com temperaturas máxima/mínima
- Assinatura de código e notarização para distribuição

**Estatísticas:**
- 47 arquivos criados
- 2.450 linhas de Swift
- 4 fases, 7 planos, 28 tarefas
- 12 dias do início à entrega

**Intervalo Git:** `feat(01-01)` → `feat(04-01)`

**Próximo passo:** Auditoria de segurança e fortalecimento para v1.1
```
</example>
