---
phase: {N}
slug: {slug-da-fase}
status: draft
shadcn_initialized: false
preset: none
created: {data}
---

# Fase {N} — Contrato de Design de UI

> Contrato visual e de interação para fases de frontend. Gerado pelo gsd-ui-researcher, verificado pelo gsd-ui-checker.

---

## Sistema de Design

| Propriedade | Valor |
|-------------|-------|
| Ferramenta | {shadcn / nenhuma} |
| Preset | {string do preset ou "não aplicável"} |
| Biblioteca de componentes | {radix / base-ui / nenhuma} |
| Biblioteca de ícones | {biblioteca} |
| Fonte | {fonte} |

---

## Escala de Espaçamento

Valores declarados (devem ser múltiplos de 4):

| Token | Valor | Uso |
|-------|-------|-----|
| xs | 4px | Gaps de ícone, padding inline |
| sm | 8px | Espaçamento compacto de elementos |
| md | 16px | Espaçamento padrão de elementos |
| lg | 24px | Padding de seção |
| xl | 32px | Gaps de layout |
| 2xl | 48px | Quebras de seções principais |
| 3xl | 64px | Espaçamento a nível de página |

Exceções: {listar quaisquer, ou "nenhuma"}

---

## Tipografia

| Função | Tamanho | Peso | Altura da Linha |
|--------|---------|------|-----------------|
| Corpo | {px} | {peso} | {ratio} |
| Rótulo | {px} | {peso} | {ratio} |
| Título | {px} | {peso} | {ratio} |
| Display | {px} | {peso} | {ratio} |

---

## Cor

| Função | Valor | Uso |
|--------|-------|-----|
| Dominante (60%) | {hex} | Background, superfícies |
| Secundária (30%) | {hex} | Cards, sidebar, nav |
| Destaque (10%) | {hex} | {listar elementos específicos apenas} |
| Destrutiva | {hex} | Apenas ações destrutivas |

Destaque reservado para: {lista explícita — nunca "todos os elementos interativos"}

---

## Contrato de Copywriting

| Elemento | Copy |
|----------|------|
| CTA Principal | {verbo + substantivo específico} |
| Título estado vazio | {copy} |
| Corpo estado vazio | {copy + próximo passo} |
| Estado de erro | {problema + caminho de solução} |
| Confirmação destrutiva | {nome da ação}: {copy de confirmação} |

---

## Segurança de Registry

| Registry | Blocos Usados | Gate de Segurança |
|----------|---------------|-------------------|
| shadcn oficial | {lista} | não necessário |
| {nome third-party} | {lista} | shadcn view + diff necessário |

---

## Aprovação do Checker

- [ ] Dimensão 1 Copywriting: PASS
- [ ] Dimensão 2 Visuais: PASS
- [ ] Dimensão 3 Cor: PASS
- [ ] Dimensão 4 Tipografia: PASS
- [ ] Dimensão 5 Espaçamento: PASS
- [ ] Dimensão 6 Segurança de Registry: PASS

**Aprovação:** {pendente / aprovado AAAA-MM-DD}
