<purpose>
Executar descoberta no nível de profundidade apropriado.
Produz DISCOVERY.md (para Nível 2-3) que informa a criação do PLAN.md.

Chamado pelo passo mandatory_discovery do planejar-fase.md com um parâmetro de profundidade.

NOTA: Para pesquisa abrangente de ecossistema ("como especialistas constroem isso"), use /gsd-pesquisar-fase em vez disso, que produz RESEARCH.md.
</purpose>

<depth_levels>
**Este workflow suporta três níveis de profundidade:**

| Nível | Nome              | Tempo     | Saída                                        | Quando                                        |
| ----- | ----------------- | --------- | -------------------------------------------- | --------------------------------------------- |
| 1     | Verificação Rápida| 2-5 min   | Sem arquivo, prosseguir com conhecimento verificado | Biblioteca única, confirmando sintaxe atual |
| 2     | Padrão            | 15-30 min | DISCOVERY.md                                 | Escolhendo entre opções, nova integração      |
| 3     | Investigação Profunda | 1+ hora | DISCOVERY.md detalhado com gates de validação | Decisões arquiteturais, problemas inéditos    |

**A profundidade é determinada pelo planejar-fase.md antes de rotear para cá.**
</depth_levels>

<source_hierarchy>
**OBRIGATÓRIO: Context7 ANTES de WebSearch**

Os dados de treinamento do Claude estão 6-18 meses desatualizados. Sempre verificar.

1. **Context7 MCP PRIMEIRO** - Docs atuais, sem alucinação
2. **Docs oficiais** - Quando o Context7 não tem cobertura
3. **WebSearch POR ÚLTIMO** - Para comparações e tendências apenas

Ver D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/discovery.md `<discovery_protocol>` para protocolo completo.
</source_hierarchy>

<process>

<step name="determine_depth">
Verificar o parâmetro de profundidade passado pelo planejar-fase.md:
- `depth=verify` → Nível 1 (Verificação Rápida)
- `depth=standard` → Nível 2 (Descoberta Padrão)
- `depth=deep` → Nível 3 (Investigação Profunda)

Rotear para o workflow de nível apropriado abaixo.
</step>

<step name="level_1_quick_verify">
**Nível 1: Verificação Rápida (2-5 minutos)**

Para: Biblioteca única conhecida, confirmando que sintaxe/versão ainda está correta.

**Processo:**

1. Resolver biblioteca no Context7:

   ```
   mcp__context7__resolve-library-id with libraryName: "[biblioteca]"
   ```

2. Buscar docs relevantes:

   ```
   mcp__context7__get-library-docs with:
   - context7CompatibleLibraryID: [do passo 1]
   - topic: [preocupação específica]
   ```

3. Verificar:

   - Versão atual corresponde às expectativas
   - Sintaxe da API inalterada
   - Sem mudanças breaking em versões recentes

4. **Se verificado:** Retornar ao planejar-fase.md com confirmação. Sem necessidade de DISCOVERY.md.

5. **Se preocupações encontradas:** Escalar para Nível 2.

**Saída:** Confirmação verbal para prosseguir, ou escalação para Nível 2.
</step>

<step name="level_2_standard">
**Nível 2: Descoberta Padrão (15-30 minutos)**

Para: Escolhendo entre opções, nova integração externa.

**Processo:**

1. **Identificar o que descobrir:**

   - Quais opções existem?
   - Quais são os critérios de comparação chave?
   - Qual é nosso caso de uso específico?

2. **Context7 para cada opção:**

   ```
   Para cada biblioteca/framework:
   - mcp__context7__resolve-library-id
   - mcp__context7__get-library-docs (mode: "code" para API, "info" para conceitos)
   ```

3. **Docs oficiais** para o que o Context7 não cobre.

4. **WebSearch** para comparações:

   - "[opção A] vs [opção B] {ano_atual}"
   - "[opção] problemas conhecidos"
   - "[opção] com [nossa stack]"

5. **Verificação cruzada:** Qualquer descoberta do WebSearch → confirmar com Context7/docs oficiais.

6. **Criar DISCOVERY.md** usando estrutura de D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/discovery.md:

   - Resumo com recomendação
   - Descobertas chave por opção
   - Exemplos de código do Context7
   - Nível de confiança (deve ser MÉDIO-ALTO para Nível 2)

7. Retornar ao planejar-fase.md.

**Saída:** `.planning/phases/XX-name/DISCOVERY.md`
</step>

<step name="level_3_deep_dive">
**Nível 3: Investigação Profunda (1+ hora)**

Para: Decisões arquiteturais, problemas inéditos, escolhas de alto risco.

**Processo:**

1. **Definir escopo da descoberta** usando D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/discovery.md:

   - Definir escopo claro
   - Definir limites de inclusão/exclusão
   - Listar perguntas específicas a responder

2. **Pesquisa exaustiva no Context7:**

   - Todas as bibliotecas relevantes
   - Padrões e conceitos relacionados
   - Múltiplos tópicos por biblioteca se necessário

3. **Leitura profunda de documentação oficial:**

   - Guias de arquitetura
   - Seções de melhores práticas
   - Guias de migração/upgrade
   - Limitações conhecidas

4. **WebSearch para contexto de ecossistema:**

   - Como outros resolveram problemas similares
   - Experiências em produção
   - Armadilhas e anti-padrões
   - Mudanças/anúncios recentes

5. **Verificar cruzado TODAS as descobertas:**

   - Cada afirmação do WebSearch → verificar com fonte autoritativa
   - Marcar o que é verificado vs assumido
   - Sinalizar contradições

6. **Criar DISCOVERY.md abrangente:**

   - Estrutura completa de D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/discovery.md
   - Relatório de qualidade com atribuição de fonte
   - Confiança por descoberta
   - Se confiança BAIXA em qualquer descoberta crítica → adicionar checkpoints de validação

7. **Gate de confiança:** Se confiança geral for BAIXA, apresentar opções antes de prosseguir.

8. Retornar ao planejar-fase.md.

**Saída:** `.planning/phases/XX-name/DISCOVERY.md` (abrangente)
</step>

<step name="identify_unknowns">
**Para Nível 2-3:** Definir o que precisamos aprender.

Perguntar: O que precisamos aprender antes de planejar esta fase?

- Escolhas tecnológicas?
- Melhores práticas?
- Padrões de API?
- Abordagem de arquitetura?
  </step>

<step name="create_discovery_scope">
Usar D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/discovery.md.

Incluir:

- Objetivo claro de descoberta
- Listas de inclusão/exclusão com escopo
- Preferências de fonte (docs oficiais, Context7, ano atual)
- Estrutura de saída para DISCOVERY.md
  </step>

<step name="execute_discovery">
Executar a descoberta:
- Usar busca web para informações atuais
- Usar Context7 MCP para docs de bibliotecas
- Preferir fontes do ano atual
- Estruturar descobertas conforme template
</step>

<step name="create_discovery_output">
Escrever `.planning/phases/XX-name/DISCOVERY.md`:
- Resumo com recomendação
- Descobertas chave com fontes
- Exemplos de código se aplicável
- Metadados (confiança, dependências, perguntas abertas, suposições)
</step>

<step name="confidence_gate">
Após criar DISCOVERY.md, verificar nível de confiança.

Se confiança for BAIXA:
Usar conversational prompting:

- header: "Conf. Baixa"
- question: "Confiança da descoberta está BAIXA: [razão]. Como deseja prosseguir?"
- options:
  - "Investigar mais" - Fazer mais pesquisa antes de planejar
  - "Prosseguir mesmo assim" - Aceitar incerteza, planejar com ressalvas
  - "Pausar" - Preciso pensar sobre isto

Se confiança for MÉDIA:
Inline: "Descoberta completa (confiança média). [razão breve]. Prosseguir para planejamento?"

Se confiança for ALTA:
Prosseguir diretamente, apenas anotar: "Descoberta completa (confiança alta)."
</step>

<step name="open_questions_gate">
Se DISCOVERY.md tiver open_questions:

Apresentar inline:
"Perguntas abertas da descoberta:

- [Pergunta 1]
- [Pergunta 2]

Estas podem afetar a implementação. Reconhecer e prosseguir? (sim / tratar primeiro)"

Se "tratar primeiro": Coletar input do usuário nas perguntas, atualizar descoberta.
</step>

<step name="offer_next">
```
Descoberta completa: .planning/phases/XX-name/DISCOVERY.md
Recomendação: [resumo de uma linha]
Confiança: [nível]

Qual o próximo passo?

1. Discutir contexto da fase (/gsd-discutir-fase [fase-atual])
2. Criar plano da fase (/gsd-planejar-fase [fase-atual])
3. Refinar descoberta (investigar mais)
4. Revisar descoberta

```

NOTA: DISCOVERY.md NÃO é commitado separadamente. Será commitado com a conclusão da fase.
</step>

</process>

<success_criteria>
**Nível 1 (Verificação Rápida):**
- Context7 consultado para biblioteca/tópico
- Estado atual verificado ou preocupações escaladas
- Confirmação verbal para prosseguir (sem arquivos)

**Nível 2 (Padrão):**
- Context7 consultado para todas as opções
- Descobertas do WebSearch verificadas cruzado
- DISCOVERY.md criado com recomendação
- Nível de confiança MÉDIO ou superior
- Pronto para informar criação do PLAN.md

**Nível 3 (Investigação Profunda):**
- Escopo de descoberta definido
- Context7 consultado exaustivamente
- Todas as descobertas do WebSearch verificadas contra fontes autoritativas
- DISCOVERY.md criado com análise abrangente
- Relatório de qualidade com atribuição de fonte
- Se descobertas com confiança BAIXA → checkpoints de validação definidos
- Gate de confiança passado
- Pronto para informar criação do PLAN.md
</success_criteria>
