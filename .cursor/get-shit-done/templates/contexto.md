# Template de Contexto de Fase

Template para `.planning/phases/XX-nome/{num_fase}-CONTEXT.md` - captura decisões de implementação para uma fase.

**Propósito:** Documentar decisões que agentes downstream precisam. O Pesquisador usa isto para saber O QUE investigar. O Planejador usa isto para saber QUAIS escolhas estão travadas vs flexíveis.

**Princípio chave:** Categorias NÃO são predefinidas. Elas emergem do que foi realmente discutido para ESTA fase. Uma fase de CLI tem seções relevantes a CLI, uma fase de UI tem seções relevantes a UI.

**Consumidores downstream:**
- `gsd-phase-researcher` — Lê decisões para focar pesquisa (ex., "layout de cards" → pesquisar padrões de componentes de card)
- `gsd-planner` — Lê decisões para criar tarefas específicas (ex., "scroll infinito" → tarefa inclui virtualização)

---

## Template do Arquivo

```markdown
# Fase [X]: [Nome] - Contexto

**Coletado:** [data]
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

[Declaração clara do que esta fase entrega — a âncora de escopo. Isto vem do ROADMAP.md e é fixo. A discussão esclarece a implementação dentro deste limite.]

</domain>

<decisions>
## Decisões de Implementação

### [Área 1 que foi discutida]
- **D-01:** [Decisão específica tomada]
- **D-02:** [Outra decisão se aplicável]

### [Área 2 que foi discutida]
- **D-03:** [Decisão específica tomada]

### [Área 3 que foi discutida]
- **D-04:** [Decisão específica tomada]

### Critério do Claude
[Áreas onde o usuário explicitamente disse "você decide" — Claude tem flexibilidade aqui durante planejamento/implementação]

</decisions>

<specifics>
## Ideias Específicas

[Quaisquer referências particulares, exemplos, ou momentos "eu quero como X" da discussão. Referências de produto, comportamentos específicos, padrões de interação.]

[Se nenhuma: "Sem requisitos específicos — aberto a abordagens padrão"]

</specifics>

<canonical_refs>
## Referências Canônicas

**Agentes downstream DEVEM ler estas antes de planejar ou implementar.**

[Liste cada spec, ADR, doc de funcionalidade ou doc de design que define requisitos ou restrições para esta fase. Use caminhos relativos completos para que agentes possam lê-los diretamente. Agrupe por área de tópico quando a fase tem múltiplas preocupações.]

### [Área de tópico 1]
- `caminho/para/spec-ou-adr.md` — [O que este doc decide/define que é relevante]
- `caminho/para/doc.md` §N — [Seção específica e o que cobre]

### [Área de tópico 2]
- `caminho/para/feature-doc.md` — [Qual capacidade isto define]

[Se o projeto não tem specs externas: "Sem specs externas — requisitos estão totalmente capturados nas decisões acima"]

</canonical_refs>

<code_context>
## Insights do Código Existente

### Ativos Reutilizáveis
- [Componente/hook/utilitário]: [Como pode ser usado nesta fase]

### Padrões Estabelecidos
- [Padrão]: [Como restringe/habilita esta fase]

### Pontos de Integração
- [Onde código novo se conecta ao sistema existente]

</code_context>

<deferred>
## Ideias Adiadas

[Ideias que surgiram durante a discussão mas pertencem a outras fases. Capturadas aqui para não serem perdidas, mas explicitamente fora do escopo desta fase.]

[Se nenhuma: "Nenhuma — discussão ficou dentro do escopo da fase"]

</deferred>

---

*Fase: XX-nome*
*Contexto coletado: [data]*
```

<good_examples>

**Exemplo 1: Funcionalidade visual (Feed de Posts)**

```markdown
# Fase 3: Feed de Posts - Contexto

**Coletado:** 2025-01-20
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

Exibir posts de usuários seguidos em um feed rolável. Usuários podem ver posts e contagens de engajamento. Criação de posts e interações são fases separadas.

</domain>

<decisions>
## Decisões de Implementação

### Estilo de layout
- Layout baseado em cards, não timeline ou lista
- Cada card mostra: avatar do autor, nome, timestamp, conteúdo completo do post, contagens de reações
- Cards têm sombras sutis, bordas arredondadas — visual moderno

### Comportamento de carregamento
- Scroll infinito, não paginação
- Pull-to-refresh no mobile
- Indicador de novos posts no topo ("3 novos posts") ao invés de auto-inserção

### Estado vazio
- Ilustração amigável + "Siga pessoas para ver posts aqui"
- Sugerir 3-5 contas para seguir baseado em interesses

### Critério do Claude
- Design do skeleton de carregamento
- Espaçamento e tipografia exatos
- Tratamento de estado de erro

</decisions>

<canonical_refs>
## Referências Canônicas

### Exibição do feed
- `docs/features/social-feed.md` — Requisitos do feed, campos do card de post, regras de exibição de engajamento
- `docs/decisions/adr-012-infinite-scroll.md` — Decisão de estratégia de scroll, requisitos de virtualização

### Estados vazios
- `docs/design/empty-states.md` — Padrões de estado vazio, diretrizes de ilustração

</canonical_refs>

<specifics>
## Ideias Específicas

- "Gosto de como o Twitter mostra o indicador de novos posts sem atrapalhar sua posição de scroll"
- Cards devem parecer com os cards de issue do Linear — limpos, sem poluição visual

</specifics>

<deferred>
## Ideias Adiadas

- Comentar em posts — Fase 5
- Salvar posts como favoritos — adicionar ao backlog

</deferred>

---

*Fase: 03-feed-de-posts*
*Contexto coletado: 2025-01-20*
```

**Exemplo 2: Ferramenta CLI (Backup de banco de dados)**

```markdown
# Fase 2: Comando de Backup - Contexto

**Coletado:** 2025-01-20
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

Comando CLI para fazer backup do banco de dados em arquivo local ou S3. Suporta backups completos e incrementais. Comando de restauração é uma fase separada.

</domain>

<decisions>
## Decisões de Implementação

### Formato de saída
- JSON para uso programático, formato de tabela para humanos
- Padrão é tabela, flag --json para JSON
- Modo verboso (-v) mostra progresso, silencioso por padrão

### Design de flags
- Flags curtas para opções comuns: -o (output), -v (verbose), -f (force)
- Flags longas para clareza: --incremental, --compress, --encrypt
- Obrigatório: string de conexão do banco (posicional ou --db)

### Recuperação de erros
- Tentar 3 vezes em falha de rede, depois falhar com mensagem clara
- Flag --no-retry para falhar rápido
- Backups parciais são deletados em caso de falha (sem arquivos corrompidos)

### Critério do Claude
- Implementação exata da barra de progresso
- Escolha do algoritmo de compressão
- Manuseio de arquivos temporários

</decisions>

<canonical_refs>
## Referências Canônicas

### CLI de Backup
- `docs/features/backup-restore.md` — Requisitos de backup, backends suportados, spec de criptografia
- `docs/decisions/adr-007-cli-conventions.md` — Nomenclatura de flags, códigos de saída, padrões de formato de saída

</canonical_refs>

<specifics>
## Ideias Específicas

- "Quero que pareça com pg_dump — familiar para pessoas de banco de dados"
- Deve funcionar em pipelines de CI (códigos de saída, sem prompts interativos)

</specifics>

<deferred>
## Ideias Adiadas

- Backups agendados — fase separada
- Rotação/retenção de backup — adicionar ao backlog

</deferred>

---

*Fase: 02-comando-backup*
*Contexto coletado: 2025-01-20*
```

**Exemplo 3: Tarefa de organização (Biblioteca de fotos)**

```markdown
# Fase 1: Organização de Fotos - Contexto

**Coletado:** 2025-01-20
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

Organizar biblioteca de fotos existente em pastas estruturadas. Lidar com duplicatas e aplicar nomenclatura consistente. Etiquetagem e busca são fases separadas.

</domain>

<decisions>
## Decisões de Implementação

### Critérios de agrupamento
- Agrupamento primário por ano, depois por mês
- Eventos detectados por clustering temporal (fotos dentro de 2 horas = mesmo evento)
- Pastas de evento nomeadas por data + localização se disponível

### Tratamento de duplicatas
- Manter versão de maior resolução
- Mover duplicatas para pasta _duplicatas (não deletar)
- Registrar todas as decisões de duplicatas para revisão

### Convenção de nomenclatura
- Formato: AAAA-MM-DD_HH-MM-SS_nomeoriginal.ext
- Preservar nome original como sufixo para buscabilidade
- Lidar com colisões de nome com sufixo incrementado

### Critério do Claude
- Algoritmo exato de clustering
- Como lidar com fotos sem dados EXIF
- Uso de emoji em pastas

</decisions>

<canonical_refs>
## Referências Canônicas

### Regras de organização
- `docs/features/photo-organization.md` — Regras de agrupamento, política de duplicatas, spec de nomenclatura
- `docs/decisions/adr-003-exif-handling.md` — Estratégia de extração EXIF, fallback para metadados ausentes

</canonical_refs>

<specifics>
## Ideias Específicas

- "Quero poder encontrar fotos por aproximadamente quando foram tiradas"
- Não deletar nada — no pior caso, mover para uma pasta de revisão

</specifics>

<deferred>
## Ideias Adiadas

- Agrupamento por detecção facial — fase futura
- Sincronização com nuvem — fora do escopo por enquanto

</deferred>

---

*Fase: 01-organizacao-fotos*
*Contexto coletado: 2025-01-20*
```

</good_examples>

<guidelines>
**Este template captura DECISÕES para agentes downstream.**

A saída deve responder: "O que o pesquisador precisa investigar? Quais escolhas estão travadas para o planejador?"

**Bom conteúdo (decisões concretas):**
- "Layout baseado em cards, não timeline"
- "Tentar 3 vezes em falha de rede, depois falhar"
- "Agrupar por ano, depois por mês"
- "JSON para uso programático, tabela para humanos"

**Conteúdo ruim (muito vago):**
- "Deve parecer moderno e limpo"
- "Boa experiência do usuário"
- "Rápido e responsivo"
- "Fácil de usar"

**Após criação:**
- Arquivo fica no diretório da fase: `.planning/phases/XX-nome/{num_fase}-CONTEXT.md`
- `gsd-phase-researcher` usa decisões para focar investigação E lê canonical_refs para saber QUAIS docs estudar
- `gsd-planner` usa decisões + pesquisa para criar tarefas executáveis E lê canonical_refs para verificar alinhamento
- Agentes downstream NÃO devem precisar perguntar ao usuário novamente sobre decisões capturadas

**CRÍTICO — Referências canônicas:**
- A seção `<canonical_refs>` é OBRIGATÓRIA. Todo CONTEXT.md deve ter uma.
- Se seu projeto tem specs externas, ADRs ou docs de design, liste-os com caminhos relativos completos agrupados por tópico
- Se ROADMAP.md lista `Refs canônicas:` por fase, extraia e expanda essas
- Menções inline como "veja ADR-019" espalhadas nas decisões são inúteis para agentes downstream — eles precisam de caminhos completos e referências de seção em uma seção dedicada que possam encontrar
- Se não existem specs externas, diga explicitamente — não omita silenciosamente a seção
</guidelines>
