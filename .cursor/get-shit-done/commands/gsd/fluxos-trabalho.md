---
description: Gerenciar fluxos de trabalho paralelos — listar, criar, alternar, status, progresso, completar e retomar
---

# /gsd-fluxos-trabalho

Gerenciar fluxos de trabalho paralelos para trabalho concorrente em marcos.

## Uso

`/gsd-fluxos-trabalho [subcomando] [args]`

### Subcomandos

| Comando | Descricao |
|---------|-----------|
| `listar` | Listar todos os fluxos de trabalho com status |
| `criar <nome>` | Criar um novo fluxo de trabalho |
| `status <nome>` | Status detalhado de um fluxo de trabalho |
| `alternar <nome>` | Definir fluxo de trabalho ativo |
| `progresso` | Resumo de progresso de todos os fluxos |
| `completar <nome>` | Arquivar um fluxo de trabalho concluido |
| `retomar <nome>` | Retomar trabalho em um fluxo |

## Passo 1: Interpretar Subcomando

Interpretar a entrada do usuario para determinar qual operacao de fluxo de trabalho executar.
Se nenhum subcomando for fornecido, usar `listar` como padrao.

## Passo 2: Executar Operacao

### listar
Executar: `node "$GSD_TOOLS" workstream list --raw --cwd "$CWD"`
Exibir os fluxos de trabalho em formato de tabela mostrando nome, status, fase atual e progresso.

### criar
Executar: `node "$GSD_TOOLS" workstream create <nome> --raw --cwd "$CWD"`
Apos a criacao, exibir o caminho do novo fluxo de trabalho e sugerir proximos passos:
- `/gsd-novo-marco --ws <nome>` para configurar o marco

### status
Executar: `node "$GSD_TOOLS" workstream status <nome> --raw --cwd "$CWD"`
Exibir detalhamento de fases e informacoes de estado.

### alternar
Executar: `node "$GSD_TOOLS" workstream set <nome> --raw --cwd "$CWD"`
Tambem definir a variavel de ambiente `GSD_WORKSTREAM` para a sessao atual.

### progresso
Executar: `node "$GSD_TOOLS" workstream progress --raw --cwd "$CWD"`
Exibir uma visao geral de progresso de todos os fluxos de trabalho.

### completar
Executar: `node "$GSD_TOOLS" workstream complete <nome> --raw --cwd "$CWD"`
Arquivar o fluxo de trabalho em milestones/.

### retomar
Definir o fluxo de trabalho como ativo e sugerir `/gsd-retomar-trabalho --ws <nome>`.

## Passo 3: Exibir Resultados

Formatar a saida JSON do gsd-tools em uma exibicao legivel.
Incluir a flag `${GSD_WS}` em quaisquer sugestoes de roteamento.
