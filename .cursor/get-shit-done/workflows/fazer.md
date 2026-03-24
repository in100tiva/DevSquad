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
| Iniciar um novo projeto, "configurar", "inicializar" | `/gsd-new-project` | Precisa de inicialização completa do projeto |
| Mapear ou analisar uma base de código existente | `/gsd-map-codebase` | Descoberta de base de código |
| Um bug, erro, crash, falha ou algo quebrado | `/gsd-debug` | Precisa de investigação sistemática |
| Explorar, pesquisar, comparar ou "como funciona X" | `/gsd-research-phase` | Pesquisa de domínio antes do planejamento |
| Discutir visão, "como X deveria parecer", brainstorming | `/gsd-discuss-phase` | Precisa de coleta de contexto |
| Uma tarefa complexa: refatoração, migração, arquitetura multi-arquivo, redesign de sistema | `/gsd-add-phase` | Precisa de uma fase completa com ciclo planejar/construir |
| Planejar uma fase específica ou "planejar fase N" | `/gsd-plan-phase` | Requisição direta de planejamento |
| Executar uma fase ou "construir fase N", "rodar fase N" | `/gsd-execute-phase` | Requisição direta de execução |
| Rodar todas as fases restantes automaticamente | `/gsd-autonomous` | Execução autônoma completa |
| Uma revisão ou preocupação de qualidade sobre trabalho existente | `/gsd-verify-work` | Precisa de verificação |
| Verificar progresso, status, "onde estou" | `/gsd-progress` | Verificação de status |
| Retomar trabalho, "continuar de onde parei" | `/gsd-resume-work` | Restauração de sessão |
| Uma nota, ideia ou "lembrar de..." | `/gsd-add-todo` | Capturar para depois |
| Adicionar testes, "escrever testes", "cobertura de testes" | `/gsd-add-tests` | Geração de testes |
| Concluir um marco, enviar, fazer release | `/gsd-complete-milestone` | Ciclo de vida do marco |
| Uma tarefa específica, acionável e pequena (adicionar funcionalidade, corrigir typo, atualizar config) | `/gsd-quick` | Auto-contida, executor único |

**Requer diretório `.planning/`:** Todas as rotas exceto `/gsd-new-project`, `/gsd-map-codebase`, `/gsd-help` e `/gsd-join-discord`. Se o projeto não existir e a rota o requerer, sugerir `/gsd-new-project` primeiro.

**Tratamento de ambiguidade:** Se o texto puder razoavelmente corresponder a múltiplas rotas, perguntar ao usuário via conversational prompting com as 2-3 melhores opções. Por exemplo:

```
"Refatorar o sistema de autenticação" pode ser:
1. /gsd-add-phase — Ciclo completo de planejamento (recomendado para refatorações multi-arquivo)
2. /gsd-quick — Execução rápida (se o escopo for pequeno e claro)

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
