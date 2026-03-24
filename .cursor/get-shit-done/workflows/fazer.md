<purpose>
Analisar texto livre do usuário e rotear para o comando GSD mais apropriado. Este é um despachante — nunca faz o trabalho em si. Combinar a intenção do usuário com o melhor comando, confirmar o roteamento e repassar.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="validate">
**Verificar entrada.**

Se `{{GSD_ARGS}}` estiver vazio, perguntar via conversational prompting:

```
O que você gostaria de fazer? Descreva a tarefa, bug ou ideia e eu vou rotear para o comando GSD correto.
```

Aguardar resposta antes de continuar.
</step>

<step name="check_project">
**Verificar se o projeto existe.**

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state load 2>/dev/null)
```

Rastrear se o diretório `.planning/` existe — algumas rotas o requerem, outras não.
</step>

<step name="route">
**Combinar intenção com comando.**

Avaliar `{{GSD_ARGS}}` contra estas regras de roteamento. Aplicar a **primeira regra correspondente**:

| Se o texto descreve... | Rotear para | Por quê |
|--------------------------|----------|-----|
| Iniciar um novo projeto, "configurar", "inicializar" | `/gsd-novo-projeto` | Precisa de inicialização completa do projeto |
| Mapear ou analisar uma base de código existente | `/gsd-mapear-codigo` | Descoberta de base de código |
| Um bug, erro, crash, falha ou algo quebrado | `/gsd-depurar` | Precisa de investigação sistemática |
| Explorar, pesquisar, comparar ou "como funciona X" | `/gsd-pesquisar-fase` | Pesquisa de domínio antes do planejamento |
| Discutir visão, "como X deveria parecer", brainstorming | `/gsd-discutir-fase` | Precisa de coleta de contexto |
| Uma tarefa complexa: refatoração, migração, arquitetura multi-arquivo, redesign de sistema | `/gsd-adicionar-fase` | Precisa de uma fase completa com ciclo planejar/construir |
| Planejar uma fase específica ou "planejar fase N" | `/gsd-planejar-fase` | Requisição direta de planejamento |
| Executar uma fase ou "construir fase N", "rodar fase N" | `/gsd-executar-fase` | Requisição direta de execução |
| Rodar todas as fases restantes automaticamente | `/gsd-autonomo` | Execução autônoma completa |
| Uma revisão ou preocupação de qualidade sobre trabalho existente | `/gsd-verificar-trabalho` | Precisa de verificação |
| Verificar progresso, status, "onde estou" | `/gsd-progresso` | Verificação de status |
| Retomar trabalho, "continuar de onde parei" | `/gsd-retomar-trabalho` | Restauração de sessão |
| Uma nota, ideia ou "lembrar de..." | `/gsd-adicionar-todo` | Capturar para depois |
| Adicionar testes, "escrever testes", "cobertura de testes" | `/gsd-adicionar-testes` | Geração de testes |
| Concluir um marco, enviar, fazer release | `/gsd-completar-marco` | Ciclo de vida do marco |
| Uma tarefa específica, acionável e pequena (adicionar funcionalidade, corrigir typo, atualizar config) | `/gsd-rapido-garantido` | Auto-contida, executor único |

**Requer diretório `.planning/`:** Todas as rotas exceto `/gsd-novo-projeto`, `/gsd-mapear-codigo`, `/gsd-ajuda` e `/gsd-entrar-discord`. Se o projeto não existir e a rota o requerer, sugerir `/gsd-novo-projeto` primeiro.

**Tratamento de ambiguidade:** Se o texto puder razoavelmente corresponder a múltiplas rotas, perguntar ao usuário via conversational prompting com as 2-3 melhores opções. Por exemplo:

```
"Refatorar o sistema de autenticação" pode ser:
1. /gsd-adicionar-fase — Ciclo completo de planejamento (recomendado para refatorações multi-arquivo)
2. /gsd-rapido-garantido — Execução rápida (se o escopo for pequeno e claro)

Qual abordagem se encaixa melhor?
```
</step>

<step name="display">
**Mostrar a decisão de roteamento.**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► ROTEAMENTO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Entrada:** {primeiros 80 caracteres de {{GSD_ARGS}}}
**Roteando para:** {comando escolhido}
**Razão:** {explicação de uma linha}
```
</step>

<step name="dispatch">
**Invocar o comando escolhido.**

Executar o comando `/gsd-*` selecionado, passando `{{GSD_ARGS}}` como argumentos.

Se o comando escolhido espera um número de fase e um não foi fornecido no texto, extraí-lo do contexto ou perguntar via conversational prompting.

Após invocar o comando, parar. O comando despachado cuida de tudo a partir daqui.
</step>

</process>

<success_criteria>
- [ ] Entrada validada (não vazia)
- [ ] Intenção combinada com exatamente um comando GSD
- [ ] Ambiguidade resolvida via pergunta ao usuário (se necessário)
- [ ] Existência do projeto verificada para rotas que o requerem
- [ ] Decisão de roteamento exibida antes do despacho
- [ ] Comando invocado com argumentos apropriados
- [ ] Nenhum trabalho feito diretamente — apenas despachante
</success_criteria>
