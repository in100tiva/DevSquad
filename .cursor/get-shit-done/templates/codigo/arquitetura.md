# Template de Arquitetura

Template para `.planning/codebase/ARCHITECTURE.md` - captura a organização conceitual do código.

**Propósito:** Documentar como o código é organizado em nível conceitual. Complementa o STRUCTURE.md (que mostra localizações físicas dos arquivos).

---

## Template do Arquivo

```markdown
# Arquitetura

**Data da Análise:** [AAAA-MM-DD]

## Visão Geral dos Padrões

**Geral:** [Nome do padrão: ex., "CLI Monolítico", "API Serverless", "Full-stack MVC"]

**Características Principais:**
- [Característica 1: ex., "Executável único"]
- [Característica 2: ex., "Tratamento de requisições stateless"]
- [Característica 3: ex., "Orientado a eventos"]

## Camadas

[Descreva as camadas conceituais e suas responsabilidades]

**[Nome da Camada]:**
- Propósito: [O que esta camada faz]
- Contém: [Tipos de código: ex., "handlers de rota", "lógica de negócio"]
- Depende de: [O que ela usa: ex., "apenas camada de dados"]
- Usada por: [O que a usa: ex., "rotas da API"]

**[Nome da Camada]:**
- Propósito: [O que esta camada faz]
- Contém: [Tipos de código]
- Depende de: [O que ela usa]
- Usada por: [O que a usa]

## Fluxo de Dados

[Descreva o ciclo de vida típico de requisição/execução]

**[Nome do Fluxo] (ex., "Requisição HTTP", "Comando CLI", "Processamento de Evento"):**

1. [Ponto de entrada: ex., "Usuário executa comando"]
2. [Etapa de processamento: ex., "Roteador encontra o caminho"]
3. [Etapa de processamento: ex., "Controller valida entrada"]
4. [Etapa de processamento: ex., "Service executa lógica"]
5. [Saída: ex., "Resposta retornada"]

**Gerenciamento de Estado:**
- [Como o estado é tratado: ex., "Stateless - sem estado persistente", "Banco por requisição", "Cache em memória"]

## Abstrações Principais

[Conceitos/padrões centrais usados em todo o código]

**[Nome da Abstração]:**
- Propósito: [O que ela representa]
- Exemplos: [ex., "UserService, ProjectService"]
- Padrão: [ex., "Singleton", "Factory", "Repository"]

**[Nome da Abstração]:**
- Propósito: [O que ela representa]
- Exemplos: [Exemplos concretos]
- Padrão: [Padrão utilizado]

## Pontos de Entrada

[Onde a execução começa]

**[Ponto de Entrada]:**
- Localização: [Breve: ex., "src/index.ts", "API Gateway triggers"]
- Gatilhos: [O que o invoca: ex., "Invocação CLI", "Requisição HTTP"]
- Responsabilidades: [O que ele faz: ex., "Parsear args, rotear para comando"]

## Tratamento de Erros

**Estratégia:** [Como erros são tratados: ex., "Exceções borbulhando até handler de nível superior", "Middleware de erro por rota"]

**Padrões:**
- [Padrão: ex., "try/catch no nível do controller"]
- [Padrão: ex., "Códigos de erro retornados ao usuário"]

## Preocupações Transversais

[Aspectos que afetam múltiplas camadas]

**Logging:**
- [Abordagem: ex., "Logger Winston, injetado por requisição"]

**Validação:**
- [Abordagem: ex., "Schemas Zod na fronteira da API"]

**Autenticação:**
- [Abordagem: ex., "Middleware JWT nas rotas protegidas"]

---

*Análise de arquitetura: [data]*
*Atualize quando padrões principais mudarem*
```

<good_examples>
```markdown
# Arquitetura

**Data da Análise:** 2025-01-20

## Visão Geral dos Padrões

**Geral:** Aplicação CLI com Sistema de Plugins

**Características Principais:**
- Executável único com subcomandos
- Extensibilidade baseada em plugins
- Estado baseado em arquivos (sem banco de dados)
- Modelo de execução síncrono

## Camadas

**Camada de Comandos:**
- Propósito: Parsear entrada do usuário e rotear para o handler apropriado
- Contém: Definições de comandos, parsing de argumentos, texto de ajuda
- Localização: `src/commands/*.ts`
- Depende de: Camada de serviço para lógica de negócio
- Usada por: Ponto de entrada CLI (`src/index.ts`)

**Camada de Serviço:**
- Propósito: Lógica de negócio central
- Contém: FileService, TemplateService, InstallService
- Localização: `src/services/*.ts`
- Depende de: Utilitários de sistema de arquivos, ferramentas externas
- Usada por: Handlers de comandos

**Camada de Utilitários:**
- Propósito: Helpers compartilhados e abstrações
- Contém: Wrappers de I/O de arquivos, resolução de caminhos, formatação de strings
- Localização: `src/utils/*.ts`
- Depende de: Apenas built-ins do Node.js
- Usada por: Camada de serviço

## Fluxo de Dados

**Execução de Comando CLI:**

1. Usuário executa: `gsd new-project`
2. Commander parseia args e flags
3. Handler do comando invocado (`src/commands/new-project.ts`)
4. Handler chama métodos do serviço (`src/services/project.ts` → `create()`)
5. Serviço lê templates, processa arquivos, escreve saída
6. Resultados logados no console
7. Processo encerra com código de status

**Gerenciamento de Estado:**
- Baseado em arquivos: Todo estado vive no diretório `.planning/`
- Sem estado persistente em memória
- Cada execução de comando é independente

## Abstrações Principais

**Service:**
- Propósito: Encapsular lógica de negócio para um domínio
- Exemplos: `src/services/file.ts`, `src/services/template.ts`, `src/services/project.ts`
- Padrão: Similar a Singleton (importados como módulos, não instanciados)

**Command:**
- Propósito: Definição de comando CLI
- Exemplos: `src/commands/new-project.ts`, `src/commands/plan-phase.ts`
- Padrão: Registro de comandos Commander.js

**Template:**
- Propósito: Estruturas de documentos reutilizáveis
- Exemplos: Templates PROJECT.md, PLAN.md
- Padrão: Arquivos Markdown com variáveis de substituição

## Pontos de Entrada

**Entrada CLI:**
- Localização: `src/index.ts`
- Gatilhos: Usuário executa `gsd <comando>`
- Responsabilidades: Registrar comandos, parsear args, exibir ajuda

**Comandos:**
- Localização: `src/commands/*.ts`
- Gatilhos: Comando correspondente da CLI
- Responsabilidades: Validar entrada, chamar serviços, formatar saída

## Tratamento de Erros

**Estratégia:** Lançar exceções, capturar no nível do comando, logar e encerrar

**Padrões:**
- Services lançam Error com mensagens descritivas
- Handlers de comandos capturam, logam erro no stderr, exit(1)
- Erros de validação mostrados antes da execução (fail fast)

## Preocupações Transversais

**Logging:**
- Console.log para saída normal
- Console.error para erros
- Chalk para saída colorida

**Validação:**
- Schemas Zod para parsing de arquivos de config
- Validação manual nos handlers de comandos
- Fail fast em entrada inválida

**Operações de Arquivo:**
- Abstração FileService sobre fs-extra
- Todos os caminhos validados antes das operações
- Escritas atômicas (arquivo temporário + rename)

---

*Análise de arquitetura: 2025-01-20*
*Atualize quando padrões principais mudarem*
```
</good_examples>

<guidelines>
**O que pertence ao ARCHITECTURE.md:**
- Padrão arquitetural geral (monolito, microserviços, camadas, etc.)
- Camadas conceituais e seus relacionamentos
- Fluxo de dados / ciclo de vida da requisição
- Abstrações e padrões principais
- Pontos de entrada
- Estratégia de tratamento de erros
- Preocupações transversais (logging, auth, validação)

**O que NÃO pertence aqui:**
- Listagens exaustivas de arquivos (isso é STRUCTURE.md)
- Escolhas de tecnologia (isso é STACK.md)
- Walkthrough linha-por-linha de código (defira para leitura de código)
- Detalhes de implementação de funcionalidades específicas

**Caminhos de arquivo SÃO bem-vindos:**
Inclua caminhos de arquivo como exemplos concretos de abstrações. Use formatação com crase: `src/services/user.ts`. Isso torna o documento de arquitetura acionável para o Claude durante o planejamento.

**Ao preencher este template:**
- Leia os pontos de entrada principais (index, server, main)
- Identifique camadas lendo imports/dependências
- Trace a execução de uma requisição/comando típico
- Note padrões recorrentes (services, controllers, repositories)
- Mantenha as descrições conceituais, não mecânicas

**Útil para planejamento de fase quando:**
- Adicionando novas funcionalidades (onde se encaixa nas camadas?)
- Refatorando (entendendo padrões atuais)
- Identificando onde adicionar código (qual camada trata X?)
- Entendendo dependências entre componentes
</guidelines>
