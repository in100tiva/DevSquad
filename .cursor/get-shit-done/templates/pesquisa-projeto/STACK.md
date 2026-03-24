# Template de Pesquisa de Stack

Template para `.planning/research/STACK.md` — tecnologias recomendadas para o domínio do projeto.

<template>

```markdown
# Pesquisa de Stack

**Domínio:** [tipo de domínio]
**Pesquisado:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Stack Recomendado

### Tecnologias Core

| Tecnologia | Versão | Propósito | Por Que Recomendada |
|------------|--------|-----------|---------------------|
| [nome] | [versão] | [o que faz] | [por que especialistas usam para este domínio] |
| [nome] | [versão] | [o que faz] | [por que especialistas usam para este domínio] |
| [nome] | [versão] | [o que faz] | [por que especialistas usam para este domínio] |

### Bibliotecas de Suporte

| Biblioteca | Versão | Propósito | Quando Usar |
|------------|--------|-----------|-------------|
| [nome] | [versão] | [o que faz] | [caso de uso específico] |
| [nome] | [versão] | [o que faz] | [caso de uso específico] |
| [nome] | [versão] | [o que faz] | [caso de uso específico] |

### Ferramentas de Desenvolvimento

| Ferramenta | Propósito | Notas |
|------------|-----------|-------|
| [nome] | [o que faz] | [dicas de configuração] |
| [nome] | [o que faz] | [dicas de configuração] |

## Instalação

```bash
# Core
npm install [pacotes]

# Suporte
npm install [pacotes]

# Dependências de dev
npm install -D [pacotes]
```

## Alternativas Consideradas

| Recomendado | Alternativa | Quando Usar Alternativa |
|-------------|-------------|-------------------------|
| [nossa escolha] | [outra opção] | [condições onde alternativa é melhor] |
| [nossa escolha] | [outra opção] | [condições onde alternativa é melhor] |

## O Que NÃO Usar

| Evitar | Por Quê | Usar Ao Invés |
|--------|---------|---------------|
| [tecnologia] | [problema específico] | [alternativa recomendada] |
| [tecnologia] | [problema específico] | [alternativa recomendada] |

## Padrões de Stack por Variante

**Se [condição]:**
- Use [variação]
- Porque [razão]

**Se [condição]:**
- Use [variação]
- Porque [razão]

## Compatibilidade de Versões

| Pacote A | Compatível Com | Notas |
|----------|----------------|-------|
| [pacote@versão] | [pacote@versão] | [notas de compatibilidade] |

## Fontes

- [ID da biblioteca Context7] — [tópicos buscados]
- [URL da documentação oficial] — [o que foi verificado]
- [Outra fonte] — [nível de confiança]

---
*Pesquisa de stack para: [domínio]*
*Pesquisado: [data]*
```

</template>

<guidelines>

**Tecnologias Core:**
- Inclua números de versão específicos
- Explique por que é a escolha padrão, não apenas o que faz
- Foque em tecnologias que afetam decisões de arquitetura

**Bibliotecas de Suporte:**
- Inclua bibliotecas comumente necessárias para este domínio
- Note quando cada uma é necessária (nem todos os projetos precisam de todas)

**Alternativas:**
- Não apenas descarte alternativas
- Explique quando alternativas fazem sentido
- Ajuda o usuário a tomar decisões informadas se discordar

**O Que NÃO Usar:**
- Alerte ativamente contra escolhas desatualizadas ou problemáticas
- Explique o problema específico, não apenas "é antigo"
- Forneça a alternativa recomendada

**Compatibilidade de Versões:**
- Note quaisquer problemas de compatibilidade conhecidos
- Crítico para evitar tempo de debugging depois

</guidelines>
