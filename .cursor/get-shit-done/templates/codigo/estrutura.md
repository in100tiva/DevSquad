# Template de Estrutura

Template para `.planning/codebase/STRUCTURE.md` - captura a organização física dos arquivos.

**Propósito:** Documentar onde as coisas fisicamente vivem no código. Responde "onde coloco X?"

---

## Template do Arquivo

```markdown
# Estrutura do Código

**Data da Análise:** [AAAA-MM-DD]

## Layout de Diretórios

[Árvore ASCII dos diretórios de nível superior com propósito - use caracteres ├── └── │ para visualização da estrutura em árvore apenas]

```
[raiz-do-projeto]/
├── [dir]/          # [Propósito]
├── [dir]/          # [Propósito]
├── [dir]/          # [Propósito]
└── [arquivo]       # [Propósito]
```

## Propósitos dos Diretórios

**[Nome do Diretório]:**
- Propósito: [O que vive aqui]
- Contém: [Tipos de arquivos: ex., "arquivos fonte *.ts", "diretórios de componentes"]
- Arquivos-chave: [Arquivos importantes neste diretório]
- Subdiretórios: [Se aninhado, descreva a estrutura]

**[Nome do Diretório]:**
- Propósito: [O que vive aqui]
- Contém: [Tipos de arquivos]
- Arquivos-chave: [Arquivos importantes]
- Subdiretórios: [Estrutura]

## Localizações de Arquivos-Chave

**Pontos de Entrada:**
- [Caminho]: [Propósito: ex., "ponto de entrada CLI"]
- [Caminho]: [Propósito: ex., "inicialização do servidor"]

**Configuração:**
- [Caminho]: [Propósito: ex., "config do TypeScript"]
- [Caminho]: [Propósito: ex., "configuração de build"]
- [Caminho]: [Propósito: ex., "variáveis de ambiente"]

**Lógica Central:**
- [Caminho]: [Propósito: ex., "serviços de negócio"]
- [Caminho]: [Propósito: ex., "modelos de banco"]
- [Caminho]: [Propósito: ex., "rotas da API"]

**Testes:**
- [Caminho]: [Propósito: ex., "testes unitários"]
- [Caminho]: [Propósito: ex., "fixtures de teste"]

**Documentação:**
- [Caminho]: [Propósito: ex., "docs voltados ao usuário"]
- [Caminho]: [Propósito: ex., "guia do desenvolvedor"]

## Convenções de Nomenclatura

**Arquivos:**
- [Padrão]: [Exemplo: ex., "kebab-case.ts para módulos"]
- [Padrão]: [Exemplo: ex., "PascalCase.tsx para componentes React"]
- [Padrão]: [Exemplo: ex., "*.test.ts para arquivos de teste"]

**Diretórios:**
- [Padrão]: [Exemplo: ex., "kebab-case para diretórios de funcionalidade"]
- [Padrão]: [Exemplo: ex., "nomes no plural para coleções"]

**Padrões Especiais:**
- [Padrão]: [Exemplo: ex., "index.ts para exports de diretório"]
- [Padrão]: [Exemplo: ex., "__tests__ para diretórios de teste"]

## Onde Adicionar Código Novo

**Nova Funcionalidade:**
- Código principal: [Caminho do diretório]
- Testes: [Caminho do diretório]
- Config se necessário: [Caminho do diretório]

**Novo Componente/Módulo:**
- Implementação: [Caminho do diretório]
- Tipos: [Caminho do diretório]
- Testes: [Caminho do diretório]

**Nova Rota/Comando:**
- Definição: [Caminho do diretório]
- Handler: [Caminho do diretório]
- Testes: [Caminho do diretório]

**Utilitários:**
- Helpers compartilhados: [Caminho do diretório]
- Definições de tipo: [Caminho do diretório]

## Diretórios Especiais

[Quaisquer diretórios com significado especial ou geração]

**[Diretório]:**
- Propósito: [ex., "Código gerado", "Saída de build"]
- Origem: [ex., "Auto-gerado por X", "Artefatos de build"]
- Commitado: [Sim/Não - no .gitignore?]

---

*Análise de estrutura: [data]*
*Atualize quando a estrutura de diretórios mudar*
```

<good_examples>
```markdown
# Estrutura do Código

**Data da Análise:** 2025-01-20

## Layout de Diretórios

```
get-shit-done/
├── bin/                # Pontos de entrada executáveis
├── commands/           # Definições de slash commands
│   └── gsd/           # Comandos específicos do GSD
├── get-shit-done/     # Recursos da skill
│   ├── references/    # Documentos de princípios
│   ├── templates/     # Templates de arquivo
│   └── workflows/     # Procedimentos multi-etapa
├── src/               # Código fonte (se aplicável)
├── tests/             # Arquivos de teste
├── package.json       # Manifesto do projeto
└── README.md          # Documentação do usuário
```

## Propósitos dos Diretórios

**bin/**
- Propósito: Pontos de entrada CLI
- Contém: install.js (script de instalação)
- Arquivos-chave: install.js - gerencia instalação via npx
- Subdiretórios: Nenhum

**commands/gsd/**
- Propósito: Definições de slash commands para Cursor
- Contém: arquivos *.md (um por comando)
- Arquivos-chave: new-project.md, plan-phase.md, execute-plan.md
- Subdiretórios: Nenhum (estrutura plana)

**get-shit-done/references/**
- Propósito: Documentos de filosofia e orientação central
- Contém: principles.md, questioning.md, plan-format.md
- Arquivos-chave: principles.md - filosofia do sistema
- Subdiretórios: Nenhum

**get-shit-done/templates/**
- Propósito: Templates de documentos para arquivos .planning/
- Contém: Definições de template com frontmatter
- Arquivos-chave: project.md, roadmap.md, plan.md, summary.md
- Subdiretórios: codebase/ (novo - para templates de stack/arquitetura/estrutura)

**get-shit-done/workflows/**
- Propósito: Procedimentos multi-etapa reutilizáveis
- Contém: Definições de workflow chamadas por comandos
- Arquivos-chave: execute-plan.md, research-phase.md
- Subdiretórios: Nenhum

## Localizações de Arquivos-Chave

**Pontos de Entrada:**
- `bin/install.js` - Script de instalação (entrada npx)

**Configuração:**
- `package.json` - Metadados do projeto, dependências, entrada bin
- `.gitignore` - Arquivos excluídos

**Lógica Central:**
- `bin/install.js` - Toda lógica de instalação (cópia de arquivos, substituição de caminhos)

**Testes:**
- `tests/` - Arquivos de teste (se presentes)

**Documentação:**
- `README.md` - Guia de instalação e uso voltado ao usuário
- `.cursor/rules/` - Instruções para o Cursor ao trabalhar neste repo

## Convenções de Nomenclatura

**Arquivos:**
- kebab-case.md: Documentos Markdown
- kebab-case.js: Arquivos fonte JavaScript
- UPPERCASE.md: Arquivos importantes do projeto (README, CLAUDE, CHANGELOG)

**Diretórios:**
- kebab-case: Todos os diretórios
- Plural para coleções: templates/, commands/, workflows/

**Padrões Especiais:**
- {nome-do-comando}.md: Definição de slash command
- *-template.md: Poderia ser usado, mas diretório templates/ é preferido

## Onde Adicionar Código Novo

**Novo Slash Command:**
- Código principal: `commands/gsd/{nome-do-comando}.md`
- Testes: `tests/commands/{nome-do-comando}.test.js` (se testes implementados)
- Documentação: Atualizar `README.md` com novo comando

**Novo Template:**
- Implementação: `get-shit-done/templates/{nome}.md`
- Documentação: Template é autodocumentado (inclui diretrizes)

**Novo Workflow:**
- Implementação: `get-shit-done/workflows/{nome}.md`
- Uso: Referenciar do comando com `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/{nome}.md`

**Novo Documento de Referência:**
- Implementação: `get-shit-done/references/{nome}.md`
- Uso: Referenciar de comandos/workflows conforme necessário

**Utilitários:**
- Nenhum utilitário ainda (`install.js` é monolítico)
- Se extraído: `src/utils/`

## Diretórios Especiais

**get-shit-done/**
- Propósito: Recursos instalados em D:/projetos/Estudo/devsquad/.cursor/
- Origem: Copiado por bin/install.js durante instalação
- Commitado: Sim (fonte da verdade)

**commands/**
- Propósito: Slash commands instalados em D:/projetos/Estudo/devsquad/.cursor/commands/
- Origem: Copiado por bin/install.js durante instalação
- Commitado: Sim (fonte da verdade)

---

*Análise de estrutura: 2025-01-20*
*Atualize quando a estrutura de diretórios mudar*
```
</good_examples>

<guidelines>
**O que pertence ao STRUCTURE.md:**
- Layout de diretórios (árvore ASCII para visualização da estrutura)
- Propósito de cada diretório
- Localizações de arquivos-chave (pontos de entrada, configs, lógica central)
- Convenções de nomenclatura
- Onde adicionar código novo (por tipo)
- Diretórios especiais/gerados

**O que NÃO pertence aqui:**
- Arquitetura conceitual (isso é ARCHITECTURE.md)
- Stack tecnológico (isso é STACK.md)
- Detalhes de implementação de código (defira para leitura de código)
- Cada arquivo individual (foque em diretórios e arquivos-chave)

**Ao preencher este template:**
- Use `tree -L 2` ou similar para visualizar estrutura
- Identifique diretórios de nível superior e seus propósitos
- Note padrões de nomenclatura observando arquivos existentes
- Localize pontos de entrada, configs e áreas de lógica principal
- Mantenha a árvore de diretórios concisa (máx 2-3 níveis)

**Formato de árvore (caracteres ASCII para estrutura apenas):**
```
raiz/
├── dir1/           # Propósito
│   ├── subdir/    # Propósito
│   └── file.ts    # Propósito
├── dir2/          # Propósito
└── file.ts        # Propósito
```

**Útil para planejamento de fase quando:**
- Adicionando novas funcionalidades (onde os arquivos devem ir?)
- Entendendo organização do projeto
- Encontrando onde lógica específica vive
- Seguindo convenções existentes
</guidelines>
