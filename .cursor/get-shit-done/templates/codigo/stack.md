# Template de Stack Tecnológico

Template para `.planning/codebase/STACK.md` - captura a fundação tecnológica.

**Propósito:** Documentar quais tecnologias executam este código. Focado em "o que executa quando você roda o código."

---

## Template do Arquivo

```markdown
# Stack Tecnológico

**Data da Análise:** [AAAA-MM-DD]

## Linguagens

**Principal:**
- [Linguagem] [Versão] - [Onde usada: ex., "todo o código da aplicação"]

**Secundária:**
- [Linguagem] [Versão] - [Onde usada: ex., "scripts de build, tooling"]

## Runtime

**Ambiente:**
- [Runtime] [Versão] - [ex., "Node.js 20.x"]
- [Requisitos adicionais se houver]

**Gerenciador de Pacotes:**
- [Gerenciador] [Versão] - [ex., "npm 10.x"]
- Lockfile: [ex., "package-lock.json presente"]

## Frameworks

**Core:**
- [Framework] [Versão] - [Propósito: ex., "servidor web", "framework de UI"]

**Testes:**
- [Framework] [Versão] - [ex., "Jest para testes unitários"]
- [Framework] [Versão] - [ex., "Playwright para E2E"]

**Build/Dev:**
- [Ferramenta] [Versão] - [ex., "Vite para bundling"]
- [Ferramenta] [Versão] - [ex., "Compilador TypeScript"]

## Dependências Principais

[Inclua apenas dependências críticas para entender o stack - limite a 5-10 mais importantes]

**Críticas:**
- [Pacote] [Versão] - [Por que importa: ex., "autenticação", "acesso ao banco"]
- [Pacote] [Versão] - [Por que importa]

**Infraestrutura:**
- [Pacote] [Versão] - [ex., "Express para roteamento HTTP"]
- [Pacote] [Versão] - [ex., "Cliente PostgreSQL"]

## Configuração

**Ambiente:**
- [Como configurado: ex., "arquivos .env", "variáveis de ambiente"]
- [Configs principais: ex., "DATABASE_URL, API_KEY obrigatórias"]

**Build:**
- [Arquivos de config de build: ex., "vite.config.ts, tsconfig.json"]

## Requisitos de Plataforma

**Desenvolvimento:**
- [Requisitos de SO ou "qualquer plataforma"]
- [Tooling adicional: ex., "Docker para DB local"]

**Produção:**
- [Alvo de deploy: ex., "Vercel", "AWS Lambda", "Container Docker"]
- [Requisitos de versão]

---

*Análise do stack: [data]*
*Atualize após mudanças importantes em dependências*
```

<good_examples>
```markdown
# Stack Tecnológico

**Data da Análise:** 2025-01-20

## Linguagens

**Principal:**
- TypeScript 5.3 - Todo o código da aplicação

**Secundária:**
- JavaScript - Scripts de build, arquivos de config

## Runtime

**Ambiente:**
- Node.js 20.x (LTS)
- Sem runtime de navegador (apenas ferramenta CLI)

**Gerenciador de Pacotes:**
- npm 10.x
- Lockfile: `package-lock.json` presente

## Frameworks

**Core:**
- Nenhum (CLI vanilla Node.js)

**Testes:**
- Vitest 1.0 - Testes unitários
- tsx - Execução TypeScript sem etapa de build

**Build/Dev:**
- TypeScript 5.3 - Compilação para JavaScript
- esbuild - Usado pelo Vitest para transformações rápidas

## Dependências Principais

**Críticas:**
- commander 11.x - Parsing de argumentos CLI e estrutura de comandos
- chalk 5.x - Estilização de saída no terminal
- fs-extra 11.x - Operações estendidas de sistema de arquivos

**Infraestrutura:**
- Built-ins do Node.js - fs, path, child_process para operações de arquivo

## Configuração

**Ambiente:**
- Sem variáveis de ambiente obrigatórias
- Configuração apenas via flags CLI

**Build:**
- `tsconfig.json` - Opções do compilador TypeScript
- `vitest.config.ts` - Configuração do runner de testes

## Requisitos de Plataforma

**Desenvolvimento:**
- macOS/Linux/Windows (qualquer plataforma com Node.js)
- Sem dependências externas

**Produção:**
- Distribuído como pacote npm
- Instalado globalmente via npm install -g
- Executa na instalação Node.js do usuário

---

*Análise do stack: 2025-01-20*
*Atualize após mudanças importantes em dependências*
```
</good_examples>

<guidelines>
**O que pertence ao STACK.md:**
- Linguagens e versões
- Requisitos de runtime (Node, Bun, Deno, navegador)
- Gerenciador de pacotes e lockfile
- Escolhas de framework
- Dependências críticas (limite a 5-10 mais importantes)
- Tooling de build
- Requisitos de plataforma/deploy

**O que NÃO pertence aqui:**
- Estrutura de arquivos (isso é STRUCTURE.md)
- Padrões arquiteturais (isso é ARCHITECTURE.md)
- Cada dependência do package.json (apenas as críticas)
- Detalhes de implementação (defira para código)

**Ao preencher este template:**
- Verifique package.json para dependências
- Note versão do runtime de .nvmrc ou package.json engines
- Inclua apenas dependências que afetam a compreensão (não cada utilitário)
- Especifique versões apenas quando a versão importa (breaking changes, compatibilidade)

**Útil para planejamento de fase quando:**
- Adicionando novas dependências (verificar compatibilidade)
- Atualizando frameworks (saber o que está em uso)
- Escolhendo abordagem de implementação (deve funcionar com stack existente)
- Entendendo requisitos de build
</guidelines>
