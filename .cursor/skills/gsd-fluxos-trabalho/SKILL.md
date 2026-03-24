---
name: gsd-fluxos-trabalho
description: "Gerenciar fluxos de trabalho paralelos — listar, criar, trocar, status, progresso, completar e retomar"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-fluxos-trabalho` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de input do usuário, pergunte conversacionalmente:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Criação de Subagentes
Quando o workflow precisar criar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

# /gsd-fluxos-trabalho

Gerenciar fluxos de trabalho paralelos para trabalho concorrente em marcos.

## Uso

`/gsd-fluxos-trabalho [subcomando] [args]`

### Subcomandos

| Comando | Descrição |
|---------|-----------|
| `list` | Listar todos os fluxos de trabalho com status |
| `create <nome>` | Criar um novo fluxo de trabalho |
| `status <nome>` | Status detalhado de um fluxo de trabalho |
| `switch <nome>` | Definir fluxo de trabalho ativo |
| `progress` | Resumo de progresso de todos os fluxos |
| `complete <nome>` | Arquivar um fluxo de trabalho concluído |
| `resume <nome>` | Retomar trabalho em um fluxo |

## Passo 1: Analisar Subcomando

Analise o input do usuário para determinar qual operação de fluxo de trabalho executar.
Se nenhum subcomando for fornecido, usar `list` como padrão.

## Passo 2: Executar Operação

### list
Execute: `node "$GSD_TOOLS" workstream list --raw --cwd "$CWD"`
Exiba os fluxos de trabalho em formato de tabela mostrando nome, status, fase atual e progresso.

### create
Execute: `node "$GSD_TOOLS" workstream create <nome> --raw --cwd "$CWD"`
Após criação, exiba o caminho do novo fluxo de trabalho e sugira próximos passos:
- `/gsd-novo-marco --ws <nome>` para configurar o marco

### status
Execute: `node "$GSD_TOOLS" workstream status <nome> --raw --cwd "$CWD"`
Exiba detalhamento de fases e informações de estado.

### switch
Execute: `node "$GSD_TOOLS" workstream set <nome> --raw --cwd "$CWD"`
Também defina a variável de ambiente `GSD_WORKSTREAM` para a sessão atual.

### progress
Execute: `node "$GSD_TOOLS" workstream progress --raw --cwd "$CWD"`
Exiba uma visão geral do progresso de todos os fluxos de trabalho.

### complete
Execute: `node "$GSD_TOOLS" workstream complete <nome> --raw --cwd "$CWD"`
Arquive o fluxo de trabalho em milestones/.

### resume
Defina o fluxo de trabalho como ativo e sugira `/gsd-retomar-trabalho --ws <nome>`.

## Passo 3: Exibir Resultados

Formate a saída JSON do gsd-tools em exibição legível para humanos.
Inclua a flag `${GSD_WS}` em qualquer sugestão de roteamento.
