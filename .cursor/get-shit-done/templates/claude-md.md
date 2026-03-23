# Template .cursor/rules/

Template para o `.cursor/rules/` da raiz do projeto — gerado automaticamente por `gsd-tools generate-claude-md`.

Contém 6 seções delimitadas por marcadores. Cada seção é atualizável independentemente.
O subcomando `generate-claude-md` gerencia 5 seções (projeto, stack, convenções, arquitetura, aplicação do workflow).
A seção de perfil é gerenciada exclusivamente por `generate-claude-profile`.

---

## Templates de Seção

### Seção Projeto
```
<!-- gsd-project-start source:PROJECT.md -->
## Projeto

{{project_content}}
<!-- gsd-project-end -->
```

**Texto de fallback:**
```
Projeto ainda não inicializado. Execute /gsd-new-project para configurar.
```

### Seção Stack
```
<!-- gsd-stack-start source:STACK.md -->
## Stack Tecnológico

{{stack_content}}
<!-- gsd-stack-end -->
```

**Texto de fallback:**
```
Stack tecnológico ainda não documentado. Será preenchido após mapeamento do código-fonte ou primeira fase.
```

### Seção Convenções
```
<!-- gsd-conventions-start source:CONVENTIONS.md -->
## Convenções

{{conventions_content}}
<!-- gsd-conventions-end -->
```

**Texto de fallback:**
```
Convenções ainda não estabelecidas. Serão preenchidas conforme padrões surgirem durante o desenvolvimento.
```

### Seção Arquitetura
```
<!-- gsd-architecture-start source:ARCHITECTURE.md -->
## Arquitetura

{{architecture_content}}
<!-- gsd-architecture-end -->
```

**Texto de fallback:**
```
Arquitetura ainda não mapeada. Siga os padrões existentes encontrados no código-fonte.
```

### Seção Aplicação do Workflow
```
<!-- gsd-workflow-start source:GSD defaults -->
## Aplicação do Workflow GSD

Antes de usar Edit, Write ou outras ferramentas que alteram arquivos, inicie o trabalho através de um comando GSD para que os artefatos de planejamento e o contexto de execução permaneçam sincronizados.

Use estes pontos de entrada:
- `/gsd-quick` para correções pequenas, atualizações de docs e tarefas avulsas
- `/gsd-debug` para investigação e correção de bugs
- `/gsd-execute-phase` para trabalho planejado de fase

Não faça edições diretas no repositório fora de um workflow GSD, a menos que o usuário peça explicitamente para ignorá-lo.
<!-- gsd-workflow-end -->
```

### Seção Perfil (Apenas Placeholder)
```
<!-- gsd-profile-start -->
## Perfil do Desenvolvedor

> Perfil ainda não configurado. Execute `/gsd-profile-user` para gerar seu perfil de desenvolvedor.
> Esta seção é gerenciada por `generate-claude-profile` — não edite manualmente.
<!-- gsd-profile-end -->
```

**Nota:** Esta seção NÃO é gerenciada por `generate-claude-md`. É gerenciada exclusivamente
por `generate-claude-profile`. O placeholder acima é usado apenas ao criar um novo
arquivo .cursor/rules/ quando nenhuma seção de perfil existe ainda.

---

## Ordem das Seções

1. **Projeto** — Identidade e propósito (o que este projeto é)
2. **Stack** — Escolhas tecnológicas (quais ferramentas são usadas)
3. **Convenções** — Padrões e regras de código (como o código é escrito)
4. **Arquitetura** — Estrutura do sistema (como os componentes se encaixam)
5. **Aplicação do Workflow** — Pontos de entrada padrão do GSD para trabalho que altera arquivos
6. **Perfil** — Preferências comportamentais do desenvolvedor (como interagir)

## Formato dos Marcadores

- Início: `<!-- gsd-{nome}-start source:{arquivo} -->`
- Fim: `<!-- gsd-{nome}-end -->`
- O atributo source permite atualizações direcionadas quando os arquivos fonte mudam
- Correspondência parcial no marcador de início (sem o `-->` de fechamento) para detecção

## Comportamento de Fallback

Quando um arquivo fonte está ausente, o texto de fallback fornece orientação acionável para o Claude:
- Guia o comportamento do Claude na ausência de dados
- Não são avisos de placeholder ou mensagens de "ausente"
- Cada fallback diz ao Claude o que fazer, não apenas o que está faltando
