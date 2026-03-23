# Perfilamento de Usuário: Referência de Heurísticas de Detecção

Este documento de referência define heurísticas de detecção para perfilamento comportamental em 8 dimensões. O agente gsd-user-profiler aplica estas regras ao analisar mensagens de sessão extraídas. Não invente dimensões ou regras de pontuação além do que está definido aqui.

## Como Usar Este Documento

1. O agente gsd-user-profiler lê este documento antes de analisar qualquer mensagem
2. Para cada dimensão, o agente examina mensagens em busca dos padrões de sinal definidos abaixo
3. O agente aplica as heurísticas de detecção para classificar o padrão do desenvolvedor
4. Confiança é pontuada usando os limites definidos por dimensão
5. Citações de evidência são curadas usando as regras na seção de Curadoria de Evidências
6. A saída deve estar em conformidade com o schema JSON na seção Schema de Saída

---

## Dimensões

### 1. Estilo de Comunicação

`dimension_id: communication_style`

**O que estamos medindo:** Como o desenvolvedor formula pedidos, instruções e feedback — o padrão estrutural das mensagens para o Claude.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `terse-direct` | Mensagens curtas, imperativas com contexto mínimo. Vai direto ao ponto imediatamente. |
| `conversational` | Mensagens de comprimento médio misturando instruções com perguntas e pensamento em voz alta. Tom natural, informal. |
| `detailed-structured` | Mensagens longas com estrutura explícita — cabeçalhos, listas numeradas, declarações de problema, pré-análise. |
| `mixed` | Sem padrão dominante; estilo muda baseado no tipo de tarefa ou contexto do projeto. |

**Padrões de sinal:**

1. **Distribuição de comprimento de mensagem** — Contagem média de palavras nas mensagens. Conciso < 50 palavras, conversacional 50-200 palavras, detalhado > 200 palavras.
2. **Proporção imperativo-interrogativo** — Proporção de comandos ("corrija isso", "adicione X") para perguntas ("o que você acha?", "devemos?"). Alta proporção imperativa sugere terse-direct.
3. **Formatação estrutural** — Presença de cabeçalhos markdown, listas numeradas, blocos de código ou bullet points nas mensagens. Formatação frequente sugere detailed-structured.
4. **Preâmbulos de contexto** — Se o desenvolvedor fornece background/contexto antes de fazer um pedido. Preâmbulos sugerem conversational ou detailed-structured.
5. **Completude de frases** — Se mensagens usam frases completas ou fragmentos/abreviações. Fragmentos sugerem terse-direct.
6. **Padrão de acompanhamento** — Se o desenvolvedor fornece contexto adicional em mensagens subsequentes (pedidos multi-mensagem sugerem conversational).

**Heurísticas de detecção:**

1. Se comprimento médio de mensagem < 50 palavras E predominantemente modo imperativo E formatação mínima --> `terse-direct`
2. Se comprimento médio de mensagem 50-200 palavras E mistura de imperativo e interrogativo E formatação ocasional --> `conversational`
3. Se comprimento médio de mensagem > 200 palavras E formatação estrutural frequente E preâmbulos de contexto presentes --> `detailed-structured`
4. Se variância de comprimento de mensagem é alta (desvio padrão > 60% da média) E nenhum padrão único domina (< 60% das mensagens correspondem a um estilo) --> `mixed`
5. Se padrão varia sistematicamente por tipo de projeto (ex: conciso em projetos CLI, detalhado em frontend) --> `mixed` com nota dependente de contexto

**Pontuação de confiança:**

- **HIGH:** 10+ mensagens mostrando padrão consistente (> 70% correspondência), mesmo padrão observado em 2+ projetos
- **MEDIUM:** 5-9 mensagens mostrando padrão, OU padrão consistente dentro de 1 projeto apenas
- **LOW:** < 5 mensagens com sinais relevantes, OU sinais mistos (padrões contraditórios observados em contextos similares)
- **UNSCORED:** 0 mensagens com sinais relevantes para esta dimensão

**Citações de exemplo:**

- **terse-direct:** "corrija o bug de auth" / "adicione paginação ao endpoint de lista" / "esse teste está falhando, faça passar"
- **conversational:** "Estou pensando que provavelmente deveríamos tratar o caso de erro aqui. O que acha de retornar um 422 ao invés de um 500? O cliente precisa saber que foi um problema de validação."
- **detailed-structured:** "## Contexto\nO fluxo de auth atualmente usa cookies de sessão mas precisamos migrar para JWT.\n\n## Requisitos\n1. Access tokens (15min expiração)\n2. Refresh tokens (7 dias)\n3. Cookies httpOnly\n\n## O que tentei\nEu olhei jose e jsonwebtoken..."

**Padrões dependentes de contexto:**

Quando o estilo de comunicação varia sistematicamente por projeto ou tipo de tarefa, reporte a divisão ao invés de forçar uma classificação única. Exemplo: "dependente de contexto: terse-direct para correção de bugs e ferramentas CLI, detailed-structured para trabalho de arquitetura e frontend." A orquestração da Fase 3 resolve divisões dependentes de contexto apresentando a divisão ao usuário.

---

### 2. Velocidade de Decisão

`dimension_id: decision_speed`

**O que estamos medindo:** Quão rápido o desenvolvedor faz escolhas quando Claude apresenta opções, alternativas ou trade-offs.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `fast-intuitive` | Decide imediatamente baseado em experiência ou intuição. Deliberação mínima. |
| `deliberate-informed` | Pede comparação ou resumo antes de decidir. Quer entender trade-offs. |
| `research-first` | Adia decisão para pesquisar independentemente. Pode sair e retornar com achados. |
| `delegator` | Delega para recomendação do Claude. Confia na sugestão. |

**Padrões de sinal:**

1. **Latência de resposta a opções** — Quantas mensagens entre Claude apresentar opções e desenvolvedor escolher. Imediato (mesma mensagem ou próxima) sugere fast-intuitive.
2. **Pedidos de comparação** — Presença de "compare estes", "quais são os trade-offs?", "prós e contras?" sugere deliberate-informed.
3. **Indicadores de pesquisa externa** — Mensagens como "eu pesquisei sobre X e...", "de acordo com a documentação...", "eu li que..." sugerem research-first.
4. **Linguagem de delegação** — "só escolha um", "o que você recomendar", "escolha você", "vá com a melhor opção" sugere delegator.
5. **Frequência de reversão de decisão** — Com que frequência o desenvolvedor muda uma decisão após fazê-la. Reversões frequentes podem indicar fast-intuitive com baixa confiança.

**Heurísticas de detecção:**

1. Se desenvolvedor seleciona opções em 1-2 mensagens da apresentação E usa linguagem decisiva ("use X", "vá com A") E raramente pede comparações --> `fast-intuitive`
2. Se desenvolvedor pede análise de trade-off ou tabelas comparativas E decide após receber comparação E faz perguntas de clarificação --> `deliberate-informed`
3. Se desenvolvedor adia decisões com "deixe eu pesquisar sobre isso" E retorna com informação externa E cita documentação ou artigos --> `research-first`
4. Se desenvolvedor usa linguagem de delegação (> 3 instâncias) E raramente sobrescreve escolhas do Claude E diz "parece bom" ou "escolha você" --> `delegator`
5. Se nenhum padrão claro OU evidência é dividida entre múltiplos estilos --> classificar como estilo dominante com nota dependente de contexto

**Pontuação de confiança:**

- **HIGH:** 10+ pontos de decisão observados mostrando padrão consistente, mesmo padrão em 2+ projetos
- **MEDIUM:** 5-9 pontos de decisão, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 pontos de decisão observados, OU estilos de decisão mistos
- **UNSCORED:** 0 mensagens contendo sinais relevantes para decisão

**Citações de exemplo:**

- **fast-intuitive:** "Use Tailwind. Próxima pergunta." / "Opção B, vamos em frente"
- **deliberate-informed:** "Pode comparar Prisma vs Drizzle para este caso de uso? Quero entender a história de migração e diferenças de type safety antes de escolher."
- **research-first:** "Segure a escolha do DB — quero ler a documentação do Drizzle e verificar as issues no GitHub primeiro. Volto com uma decisão."
- **delegator:** "Você sabe mais sobre isso que eu. O que recomendar, vá com isso."

**Padrões dependentes de contexto:**

Velocidade de decisão frequentemente varia por importância. Um desenvolvedor pode ser fast-intuitive para escolhas de estilização mas research-first para decisões de banco de dados ou auth. Quando este padrão é claro, reporte a divisão: "dependente de contexto: fast-intuitive para baixa importância (estilização, nomes), deliberate-informed para alta importância (arquitetura, segurança)."

---

### 3. Profundidade de Explicação

`dimension_id: explanation_depth`

**O que estamos medindo:** Quanta explicação o desenvolvedor quer junto com código — sua preferência por entendimento vs. velocidade.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `code-only` | Quer código funcional com explicação mínima ou nenhuma. Lê e entende código diretamente. |
| `concise` | Quer explicação breve da abordagem com código. Decisões chave anotadas, não exaustiva. |
| `detailed` | Quer walkthrough completo da abordagem, raciocínio e código. Aprecia estrutura. |
| `educational` | Quer explicação conceitual profunda. Trata interações como oportunidades de aprendizado. |

**Padrões de sinal:**

1. **Pedidos explícitos de profundidade** — "apenas mostre o código", "explique por quê", "me ensine sobre X", "pule a explicação"
2. **Reação a explicações** — O desenvolvedor pula explicações? Pede mais detalhe? Diz "demais"?
3. **Profundidade de perguntas de acompanhamento** — Acompanhamentos superficiais ("funciona?") vs. conceituais ("por que esse padrão ao invés de X?")
4. **Sinais de compreensão de código** — O desenvolvedor referencia detalhes de implementação em suas mensagens? Isso sugere que lê e entende código diretamente.
5. **Sinais "eu sei isso"** — Mensagens como "eu conheço X", "pule o básico", "eu sei como hooks funcionam" indicam menor preferência por explicação.

**Heurísticas de detecção:**

1. Se desenvolvedor diz "só o código" ou "pule a explicação" E raramente faz perguntas conceituais de acompanhamento E referencia detalhes de código diretamente --> `code-only`
2. Se desenvolvedor aceita explicações breves sem pedir mais E faz acompanhamentos focados sobre decisões específicas --> `concise`
3. Se desenvolvedor faz perguntas "por quê" E pede walkthroughs E aprecia explicações estruturadas --> `detailed`
4. Se desenvolvedor faz perguntas conceituais além da tarefa imediata E usa linguagem de aprendizado ("quero entender", "me ensine") --> `educational`

**Pontuação de confiança:**

- **HIGH:** 10+ mensagens mostrando preferência consistente, mesma preferência em 2+ projetos
- **MEDIUM:** 5-9 mensagens, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 mensagens relevantes, OU preferências mudam entre interações
- **UNSCORED:** 0 mensagens com sinais relevantes

**Citações de exemplo:**

- **code-only:** "Só me dê a implementação. Eu leio e entendo." / "Pule a explicação, mostre o código."
- **concise:** "Resumo rápido da abordagem, depois o código por favor." / "Por que você usou um Map aqui ao invés de um objeto?"
- **detailed:** "Me guie por isso passo a passo. Quero entender o fluxo de auth antes de implementar."
- **educational:** "Pode explicar como rotação de refresh token JWT funciona conceitualmente? Quero entender o modelo de segurança, não apenas implementar."

**Padrões dependentes de contexto:**

Profundidade de explicação frequentemente correlaciona com familiaridade do domínio. Um desenvolvedor pode querer code-only para tecnologia bem conhecida mas educational para novos domínios. Reporte divisões quando observadas: "dependente de contexto: code-only para React/TypeScript, detailed para otimização de banco de dados."

---

### 4. Abordagem de Debugging

`dimension_id: debugging_approach`

**O que estamos medindo:** Como o desenvolvedor aborda problemas, erros e comportamento inesperado ao trabalhar com Claude.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `fix-first` | Cola erro, quer corrigido. Interesse mínimo em diagnóstico. Orientado a resultados. |
| `diagnostic` | Compartilha erro com contexto, quer entender a causa antes de corrigir. |
| `hypothesis-driven` | Investiga independentemente primeiro, traz teorias específicas ao Claude para validação. |
| `collaborative` | Quer resolver o problema passo-a-passo com Claude como parceiro. |

**Padrões de sinal:**

1. **Estilo de apresentação de erro** — Apenas colar erro (fix-first) vs. erro + "eu acho que pode ser..." (hypothesis-driven) vs. "Pode me ajudar a entender por que..." (diagnostic)
2. **Indicadores de pré-investigação** — O desenvolvedor compartilha o que já tentou? Menciona ler logs, verificar estado ou isolar o problema?
3. **Interesse em causa raiz** — Após uma correção, o desenvolvedor pergunta "por que isso aconteceu?" ou apenas segue em frente?
4. **Linguagem passo-a-passo** — "Vamos verificar X primeiro", "o que devemos olhar em seguida?", "me guie pelo debugging"
5. **Padrão de aceitação de correção** — O desenvolvedor aplica correções imediatamente ou as questiona primeiro?

**Heurísticas de detecção:**

1. Se desenvolvedor cola erros sem contexto E aceita correções sem perguntas sobre causa raiz E segue em frente imediatamente --> `fix-first`
2. Se desenvolvedor fornece contexto do erro E pergunta "por que isso está acontecendo?" E quer explicação com a correção --> `diagnostic`
3. Se desenvolvedor compartilha sua própria análise E propõe teorias ("eu acho que o problema é X porque...") E pede ao Claude para confirmar ou refutar --> `hypothesis-driven`
4. Se desenvolvedor usa linguagem colaborativa ("vamos", "o que devemos verificar?") E prefere diagnóstico incremental E resolve problemas juntos --> `collaborative`

**Pontuação de confiança:**

- **HIGH:** 10+ interações de debugging mostrando abordagem consistente, mesma abordagem em 2+ projetos
- **MEDIUM:** 5-9 interações de debugging, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 interações de debugging, OU abordagem varia significativamente
- **UNSCORED:** 0 mensagens com sinais relevantes para debugging

**Citações de exemplo:**

- **fix-first:** "Recebi este erro: TypeError: Cannot read properties of undefined. Corrija."
- **diagnostic:** "A API retorna 500 quando envio um POST para /users. Aqui está o corpo do request e o log do servidor. O que está causando isso?"
- **hypothesis-driven:** "Acho que a race condition está no cleanup do useEffect. Verifiquei e a subscription não está sendo cancelada no unmount. Pode confirmar?"
- **collaborative:** "Vamos debugar isso juntos. O teste passa localmente mas falha no CI. O que devemos verificar primeiro?"

**Padrões dependentes de contexto:**

Abordagem de debugging pode variar por urgência. Um desenvolvedor pode ser fix-first sob pressão de prazo mas hypothesis-driven durante desenvolvimento regular. Note padrões temporais se detectados.

---

### 5. Filosofia de UX

`dimension_id: ux_philosophy`

**O que estamos medindo:** Como o desenvolvedor prioriza experiência do usuário, design e qualidade visual relativo a funcionalidade.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `function-first` | Faça funcionar, polimento depois. Preocupação mínima com UX durante implementação. |
| `pragmatic` | Usabilidade básica desde o início. Nada feio ou quebrado, mas sem obsessão por design. |
| `design-conscious` | Design e UX são tratados como tão importantes quanto funcionalidade. Atenção a detalhe visual. |
| `backend-focused` | Constrói principalmente backend/CLI. Exposição ou interesse mínimo em frontend. |

**Padrões de sinal:**

1. **Pedidos relacionados a design** — Menções de estilização, layout, responsividade, animações, esquemas de cor, espaçamento
2. **Timing de polimento** — O desenvolvedor pede polimento visual durante implementação ou adia?
3. **Especificidade de feedback de UI** — Vago ("faça parecer melhor") vs. específico ("aumente o padding para 16px, mude o font weight para 600")
4. **Distribuição frontend vs. backend** — Proporção de pedidos focados em frontend vs. focados em backend
5. **Menções de acessibilidade** — Referências a a11y, leitores de tela, navegação por teclado, atributos ARIA

**Heurísticas de detecção:**

1. Se desenvolvedor raramente menciona UI/UX E foca em lógica, APIs, dados E adia estilização ("faremos bonito depois") --> `function-first`
2. Se desenvolvedor inclui requisitos básicos de UX E menciona usabilidade mas não pixel-perfection E equilibra forma com função --> `pragmatic`
3. Se desenvolvedor fornece requisitos específicos de design E menciona polimento, animações, espaçamento E trata bugs de UI tão seriamente quanto bugs de lógica --> `design-conscious`
4. Se desenvolvedor trabalha principalmente em ferramentas CLI, APIs ou sistemas backend E raramente ou nunca trabalha em frontend E mensagens focam em dados, performance, infraestrutura --> `backend-focused`

**Pontuação de confiança:**

- **HIGH:** 10+ mensagens com sinais relevantes de UX, mesmo padrão em 2+ projetos
- **MEDIUM:** 5-9 mensagens, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 mensagens relevantes, OU filosofia varia por tipo de projeto
- **UNSCORED:** 0 mensagens com sinais relevantes de UX

**Citações de exemplo:**

- **function-first:** "Só faça o formulário funcionar. Estilizamos depois." / "Não me importo como parece, preciso dos dados fluindo."
- **pragmatic:** "Garanta que o estado de loading seja visível e as mensagens de erro sejam claras. Estilização padrão é ok."
- **design-conscious:** "O botão precisa de mais espaço para respirar — adicione 12px de padding vertical e faça a transição do hover state 200ms. Também verifique a razão de contraste."
- **backend-focused:** "Estou construindo uma ferramenta CLI. Não precisa de UI." / "Adicione o endpoint REST, eu cuido do frontend separadamente."

**Padrões dependentes de contexto:**

Filosofia de UX é inerentemente dependente do projeto. Um desenvolvedor construindo uma ferramenta CLI é necessariamente backend-focused para aquele projeto. Quando possível, distinga entre padrões orientados pelo projeto e orientados pela preferência. Se o desenvolvedor só tem projetos backend, note que a classificação reflete dados disponíveis: "backend-focused (nota: todos os projetos analisados são backend/CLI — pode não refletir preferências de frontend)."

---

### 6. Filosofia de Vendor

`dimension_id: vendor_philosophy`

**O que estamos medindo:** Como o desenvolvedor aborda escolha e avaliação de bibliotecas, frameworks e serviços externos.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `pragmatic-fast` | Usa o que funciona, o que Claude sugere, ou o que é mais rápido. Avaliação mínima. |
| `conservative` | Prefere opções bem conhecidas, testadas em batalha, amplamente adotadas. Avesso a risco. |
| `thorough-evaluator` | Pesquisa alternativas, lê docs, compara funcionalidades e trade-offs antes de se comprometer. |
| `opinionated` | Tem preferências fortes e pré-existentes por ferramentas específicas. Sabe o que gosta. |

**Padrões de sinal:**

1. **Linguagem de seleção de biblioteca** — "use qualquer um", "X é o padrão?", "quero comparar A vs B", "usamos X, ponto final"
2. **Profundidade de avaliação** — O desenvolvedor aceita a primeira sugestão ou pede alternativas?
3. **Preferências declaradas** — Menções explícitas de ferramentas preferidas, experiência passada ou filosofia de ferramentas
4. **Padrões de rejeição** — O desenvolvedor rejeita sugestões do Claude? Com que base (popularidade, experiência pessoal, qualidade de docs)?
5. **Atitude sobre dependências** — "minimizar dependências", "sem deps externas", "adicione o que precisar" — revela filosofia sobre código externo

**Heurísticas de detecção:**

1. Se desenvolvedor aceita sugestões de biblioteca sem resistência E usa frases como "parece bom" ou "vá com isso" E raramente pergunta sobre alternativas --> `pragmatic-fast`
2. Se desenvolvedor pergunta sobre popularidade, manutenção, comunidade E prefere "padrão da indústria" ou "testado em batalha" E evita novo/experimental --> `conservative`
3. Se desenvolvedor pede comparações E lê docs antes de decidir E pergunta sobre edge cases, licença, tamanho do bundle --> `thorough-evaluator`
4. Se desenvolvedor nomeia bibliotecas específicas sem ser provocado E sobrescreve sugestões do Claude E expressa preferências fortes --> `opinionated`

**Pontuação de confiança:**

- **HIGH:** 10+ decisões de vendor/biblioteca observadas, mesmo padrão em 2+ projetos
- **MEDIUM:** 5-9 decisões, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 decisões de vendor observadas, OU padrão varia
- **UNSCORED:** 0 mensagens com sinais de seleção de vendor

**Citações de exemplo:**

- **pragmatic-fast:** "Use qualquer ORM que recomendar. Só preciso funcionando." / "Claro, Tailwind está ótimo."
- **conservative:** "Prisma é o ORM mais usado para isso? Quero algo com uma comunidade grande." / "Vamos ficar com o que a maioria dos times usa."
- **thorough-evaluator:** "Antes de escolher uma biblioteca de gerenciamento de estado, pode comparar Zustand vs Jotai vs Redux Toolkit? Quero entender tamanho do bundle, superfície de API e suporte TypeScript."
- **opinionated:** "Usamos Drizzle, não Prisma. Eu usei ambos e a API tipo-SQL do Drizzle é melhor para queries complexas."

**Padrões dependentes de contexto:**

Filosofia de vendor pode mudar baseada na importância do projeto ou domínio. Projetos pessoais podem usar pragmatic-fast enquanto projetos profissionais usam thorough-evaluator. Reporte a divisão se detectada.

---

### 7. Gatilhos de Frustração

`dimension_id: frustration_triggers`

**O que estamos medindo:** O que causa frustração visível, correção ou sinais emocionais negativos nas mensagens do desenvolvedor para o Claude.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `scope-creep` | Frustrado quando Claude faz coisas que não foram pedidas. Quer execução limitada. |
| `instruction-adherence` | Frustrado quando Claude não segue instruções precisamente. Valoriza exatidão. |
| `verbosity` | Frustrado quando Claude explica demais ou é muito prolixo. Quer concisão. |
| `regression` | Frustrado quando Claude quebra código funcional ao corrigir outra coisa. Valoriza estabilidade. |

**Padrões de sinal:**

1. **Linguagem de correção** — "eu não pedi isso", "não faça X", "eu disse Y não Z", "por que você mudou isso?"
2. **Padrões de repetição** — Repetir a mesma instrução com ênfase sugere frustração com instruction-adherence
3. **Mudanças de tom emocional** — Mudança de neutro para conciso, uso de maiúsculas, pontos de exclamação, palavras explícitas de frustração
4. **Declarações "Não"** — "não adicione funcionalidades extras", "não explique tanto", "não toque nesse arquivo" — o que proíbem revela o que os frustra
5. **Recuperação de frustração** — Quão rápido o desenvolvedor retorna ao tom neutro após evento de frustração

**Heurísticas de detecção:**

1. Se desenvolvedor corrige Claude por fazer trabalho não solicitado E usa linguagem como "eu só pedi X", "pare de adicionar coisas", "se limite ao que pedi" --> `scope-creep`
2. Se desenvolvedor repete instruções E corrige desvios específicos de requisitos declarados E enfatiza precisão ("eu especificamente disse...") --> `instruction-adherence`
3. Se desenvolvedor pede ao Claude para ser mais curto E pula explicações E expressa irritação com comprimento ("demais", "só a resposta") --> `verbosity`
4. Se desenvolvedor expressa frustração com funcionalidade quebrada E verifica por regressões E diz "você quebrou X enquanto corrigia Y" --> `regression`

**Pontuação de confiança:**

- **HIGH:** 10+ eventos de frustração mostrando padrão de gatilho consistente, mesmo gatilho em 2+ projetos
- **MEDIUM:** 5-9 eventos de frustração, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 eventos de frustração observados (nota: contagem baixa de frustração é POSITIVA — significa que o desenvolvedor está geralmente satisfeito, não que dados são insuficientes)
- **UNSCORED:** 0 mensagens com sinais de frustração (nota: "nenhuma frustração detectada" é um achado válido)

**Citações de exemplo:**

- **scope-creep:** "Eu pedi para corrigir o bug de login, não refatorar o módulo de auth inteiro. Reverta tudo exceto a correção do bug."
- **instruction-adherence:** "Eu disse para usar um Map, não um objeto. Eu fui específico sobre isso. Por favor refaça com um Map."
- **verbosity:** "Explicação demais. Só me mostre a mudança de código, nada mais."
- **regression:** "A busca estava funcionando antes. Agora depois da sua 'correção' no filtro, resultados de busca estão vazios. Não toque em coisas que eu não pedi para mudar."

**Padrões dependentes de contexto:**

Gatilhos de frustração tendem a ser consistentes entre projetos (orientado pela personalidade, não pelo projeto). No entanto, sua intensidade pode variar com a importância do projeto. Se múltiplos gatilhos de frustração são observados, reporte o primário (mais frequente) e note secundários.

---

### 8. Estilo de Aprendizado

`dimension_id: learning_style`

**O que estamos medindo:** Como o desenvolvedor prefere entender novos conceitos, ferramentas ou padrões que encontra.

**Espectro de classificação:**

| Classificação | Descrição |
|----------------|-----------|
| `self-directed` | Lê código diretamente, resolve coisas independentemente. Faz perguntas específicas ao Claude. |
| `guided` | Pede ao Claude para explicar partes relevantes. Prefere entendimento guiado. |
| `documentation-first` | Lê docs oficiais e tutoriais antes de mergulhar. Referencia documentação. |
| `example-driven` | Quer exemplos funcionais para modificar e aprender. Aprendizado por reconhecimento de padrões. |

**Padrões de sinal:**

1. **Iniciação de aprendizado** — O desenvolvedor começa lendo código, pedindo explicação, solicitando docs ou pedindo exemplos?
2. **Referência a fontes externas** — Menções de documentação, tutoriais, Stack Overflow, posts de blog sugerem documentation-first
3. **Pedidos de exemplo** — "me mostre um exemplo", "pode dar uma amostra?", "deixe eu ver como isso fica na prática"
4. **Indicadores de leitura de código** — "eu olhei a implementação", "eu vejo que X chama Y", "pela leitura do código..."
5. **Pedidos de explicação vs. pedidos de código** — Proporção de mensagens "explique X" para "me mostre X"

**Heurísticas de detecção:**

1. Se desenvolvedor referencia leitura de código diretamente E faz perguntas específicas direcionadas E demonstra investigação independente --> `self-directed`
2. Se desenvolvedor pede ao Claude para explicar conceitos E pede walkthroughs E prefere entendimento mediado pelo Claude --> `guided`
3. Se desenvolvedor cita documentação E pede links de docs E menciona ler tutoriais ou guias oficiais --> `documentation-first`
4. Se desenvolvedor pede exemplos E modifica exemplos fornecidos E aprende por reconhecimento de padrões --> `example-driven`

**Pontuação de confiança:**

- **HIGH:** 10+ interações de aprendizado mostrando preferência consistente, mesma preferência em 2+ projetos
- **MEDIUM:** 5-9 interações de aprendizado, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 interações de aprendizado, OU preferência varia por familiaridade do tópico
- **UNSCORED:** 0 mensagens com sinais relevantes de aprendizado

**Citações de exemplo:**

- **self-directed:** "Eu li o código do middleware. O problema é que a verificação de token acontece após o rate limiter. Deveriam ser trocados?"
- **guided:** "Pode me guiar por como o fluxo de auth funciona neste codebase? Comece pelo request de login."
- **documentation-first:** "Eu li os docs do Prisma sobre relações. Pode me ajudar a aplicar o padrão many-to-many do guia deles ao nosso schema?"
- **example-driven:** "Me mostre um exemplo funcional de uma rota de API protegida com validação JWT. Eu adapto para nossos endpoints."

**Padrões dependentes de contexto:**

Estilo de aprendizado frequentemente varia com expertise do domínio. Um desenvolvedor pode ser self-directed em domínios familiares mas guided ou example-driven em novos. Reporte a divisão se detectada: "dependente de contexto: self-directed para TypeScript/Node, example-driven para Rust/programação de sistemas."

---

## Curadoria de Evidências

### Formato de Evidência

Use o formato combinado para cada entrada de evidência:

**Sinal:** [interpretação do padrão — o que a citação demonstra] / **Exemplo:** "[citação aparada, ~100 caracteres]" — projeto: [nome do projeto]

### Metas de Evidência

- **3 citações de evidência por dimensão** (24 total em todas as 8 dimensões)
- Selecione citações que melhor ilustrem o padrão classificado
- Prefira citações de projetos diferentes para demonstrar consistência entre projetos
- Quando menos de 3 citações relevantes existirem, inclua o que está disponível e note a contagem de evidências

### Truncamento de Citação

- Apare citações para o sinal comportamental — a parte que demonstra o padrão
- Mire em aproximadamente 100 caracteres por citação
- Preserve o fragmento significativo, não a mensagem completa
- Se o sinal está no meio de uma mensagem longa, use "..." para indicar truncamento
- Nunca inclua a mensagem completa de 500 caracteres quando 50 caracteres capturam o sinal

### Atribuição de Projeto

- Toda citação de evidência deve incluir o nome do projeto
- Atribuição de projeto permite verificação e mostra padrões entre projetos
- Formato: `-- projeto: [nome]`

### Exclusão de Conteúdo Sensível (Camada 1)

O agente profiler nunca deve selecionar citações contendo qualquer um dos seguintes padrões:

- `sk-` (prefixos de chave API)
- `Bearer ` (tokens de auth)
- `password` (credenciais)
- `secret` (segredos)
- `token` (quando usado como valor de credencial, não discussão de conceito)
- `api_key` ou `API_KEY` (referências de chave API)
- Caminhos de arquivo absolutos completos contendo nomes de usuário (ex: `/Users/john/...`, `/home/john/...`)

**Quando conteúdo sensível é encontrado e excluído**, reporte como metadados na saída de análise:

```json
{
  "sensitive_excluded": [
    { "type": "api_key_pattern", "count": 2 },
    { "type": "file_path_with_username", "count": 1 }
  ]
}
```

Estes metadados permitem auditoria de defesa em profundidade. A Camada 2 (filtro regex no passo de write-profile) fornece um segundo passo, mas o profiler deve ainda evitar selecionar citações sensíveis.

### Prioridade de Linguagem Natural

Pese mensagens de linguagem natural mais alto que:
- Saída de log colada (detectada por timestamps, strings de formato repetidas, `[DEBUG]`, `[INFO]`, `[ERROR]`)
- Dumps de contexto de sessão (mensagens começando com "This session is being continued from a previous conversation")
- Grandes colagens de código (mensagens onde > 80% do conteúdo está dentro de blocos de código)

Estes tipos de mensagem são genuínos mas carregam menos sinal comportamental. Desprioritize-os ao selecionar citações de evidência.

---

## Ponderação por Recência

### Diretriz

Sessões recentes (últimos 30 dias) devem ser ponderadas aproximadamente 3x em comparação com sessões mais antigas ao analisar padrões.

### Justificativa

Estilos de desenvolvedores evoluem. Um desenvolvedor que era conciso seis meses atrás pode agora fornecer contexto detalhado estruturado. Comportamento recente é um reflexo mais preciso do estilo de trabalho atual.

### Aplicação

1. Ao contar sinais para pontuação de confiança, sinais recentes contam 3x (ex: 4 sinais recentes = 12 sinais ponderados)
2. Ao selecionar citações de evidência, prefira citações recentes sobre mais antigas quando ambas demonstram o mesmo padrão
3. Quando padrões conflitam entre sessões recentes e antigas, o padrão recente tem precedência para a classificação, mas note a evolução: "recentemente mudou de terse-direct para conversational"
4. A janela de 30 dias é relativa à data de análise, não uma data fixa

### Casos Especiais

- Se TODAS as sessões são mais antigas que 30 dias, não aplique ponderação (todas as sessões são igualmente desatualizadas)
- Se TODAS as sessões estão dentro dos últimos 30 dias, não aplique ponderação (todas as sessões são igualmente recentes)
- O peso 3x é uma diretriz, não um multiplicador rígido — use julgamento quando a contagem ponderada muda um limite de confiança

---

## Tratamento de Dados Escassos

### Limites de Mensagem

| Total de Mensagens Genuínas | Modo | Comportamento |
|-----------------------------|------|---------------|
| > 50 | `full` | Análise completa em todas as 8 dimensões. Questionário opcional (usuário pode optar por suplementar). |
| 20-50 | `hybrid` | Analisar mensagens disponíveis. Pontuar cada dimensão com confiança. Suplementar com questionário para dimensões LOW/UNSCORED. |
| < 20 | `insufficient` | Todas as dimensões pontuadas LOW ou UNSCORED. Recomendar questionário como fonte primária de perfil. Nota: "dados de sessão insuficientes para análise comportamental." |

### Tratamento de Dimensões Insuficientes

Quando uma dimensão específica tem dados insuficientes (mesmo se total de mensagens exceder limites):

- Definir confiança como `UNSCORED`
- Definir resumo como: "Dados insuficientes — nenhum sinal claro detectado para esta dimensão."
- Definir claude_instruction com fallback neutro: "Nenhuma preferência forte detectada. Pergunte ao desenvolvedor quando esta dimensão for relevante."
- Definir evidence_quotes como array vazio `[]`
- Definir evidence_count como `0`

### Suplemento por Questionário

Quando operando em modo `hybrid`, o questionário preenche lacunas para dimensões onde análise de sessão produziu confiança LOW ou UNSCORED. As classificações derivadas do questionário usam:
- Confiança **MEDIUM** para escolhas fortes e definitivas
- Confiança **LOW** para seleções "varia" ou ambíguas

Se análise de sessão e questionário concordam em uma dimensão, confiança pode ser elevada (ex: sessão LOW + questionário MEDIUM concordância = MEDIUM).

---

## Schema de Saída

O agente profiler deve retornar JSON correspondendo a este schema exato, envolvido em tags `<analysis>`.

```json
{
  "profile_version": "1.0",
  "analyzed_at": "ISO-8601 timestamp",
  "data_source": "session_analysis",
  "projects_analyzed": ["nome-projeto-1", "nome-projeto-2"],
  "messages_analyzed": 0,
  "message_threshold": "full|hybrid|insufficient",
  "sensitive_excluded": [
    { "type": "string", "count": 0 }
  ],
  "dimensions": {
    "communication_style": {
      "rating": "terse-direct|conversational|detailed-structured|mixed",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [
        {
          "signal": "Interpretação do padrão descrevendo o que a citação demonstra",
          "quote": "Citação aparada, aproximadamente 100 caracteres",
          "project": "nome-do-projeto"
        }
      ],
      "summary": "Descrição de uma a duas frases do padrão observado",
      "claude_instruction": "Diretiva imperativa para Claude: 'Corresponda estilo de comunicação estruturado' não 'Você tende a fornecer contexto estruturado'"
    },
    "decision_speed": {
      "rating": "fast-intuitive|deliberate-informed|research-first|delegator",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "explanation_depth": {
      "rating": "code-only|concise|detailed|educational",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "debugging_approach": {
      "rating": "fix-first|diagnostic|hypothesis-driven|collaborative",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "ux_philosophy": {
      "rating": "function-first|pragmatic|design-conscious|backend-focused",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "vendor_philosophy": {
      "rating": "pragmatic-fast|conservative|thorough-evaluator|opinionated",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "frustration_triggers": {
      "rating": "scope-creep|instruction-adherence|verbosity|regression",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "learning_style": {
      "rating": "self-directed|guided|documentation-first|example-driven",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    }
  }
}
```

### Notas do Schema

- **`profile_version`**: Sempre `"1.0"` para esta versão do schema
- **`analyzed_at`**: Timestamp ISO-8601 de quando a análise foi realizada
- **`data_source`**: `"session_analysis"` para perfilamento baseado em sessão, `"questionnaire"` para apenas questionário, `"hybrid"` para combinado
- **`projects_analyzed`**: Lista de nomes de projeto que contribuíram mensagens
- **`messages_analyzed`**: Número total de mensagens genuínas de usuário processadas
- **`message_threshold`**: Qual modo de limite foi acionado (`full`, `hybrid`, `insufficient`)
- **`sensitive_excluded`**: Array de tipos de conteúdo sensível excluídos com contagens (array vazio se nenhum encontrado)
- **`claude_instruction`**: Deve ser escrito na forma imperativa direcionada ao Claude. Este campo é como o perfil se torna acionável.
  - Bom: "Forneça respostas estruturadas com cabeçalhos e listas numeradas para corresponder ao estilo de comunicação deste desenvolvedor."
  - Ruim: "Você tende a gostar de respostas estruturadas."
  - Bom: "Pergunte antes de fazer mudanças além do pedido declarado — este desenvolvedor valoriza execução limitada."
  - Ruim: "O desenvolvedor fica frustrado quando você faz trabalho extra."

---

## Consistência Entre Projetos

### Avaliação

Para cada dimensão, avalie se o padrão observado é consistente entre os projetos analisados:

- **`cross_project_consistent: true`** — Mesma classificação se aplicaria independente de qual projeto é analisado. Evidência de 2+ projetos mostra o mesmo padrão.
- **`cross_project_consistent: false`** — Padrão varia por projeto. Inclua uma nota dependente de contexto no resumo.

### Reportando Divisões

Quando `cross_project_consistent` é false, o resumo deve descrever a divisão:

- "Dependente de contexto: terse-direct para projetos CLI/backend (gsd-tools, api-server), detailed-structured para projetos frontend (dashboard, landing-page)."
- "Dependente de contexto: fast-intuitive para tech familiar (React, Node), research-first para novos domínios (Rust, ML)."

O campo de classificação deve refletir o padrão **dominante** (mais evidência). O resumo descreve a nuance.

### Resolução da Fase 3

Divisões dependentes de contexto são resolvidas durante a orquestração da Fase 3. O orquestrador apresenta a divisão ao desenvolvedor e pergunta qual padrão representa sua preferência geral. Até ser resolvido, Claude usa o padrão dominante com consciência da variação dependente de contexto.

---

*Versão do documento de referência: 1.0*
*Dimensões: 8*
*Schema: profile_version 1.0*
