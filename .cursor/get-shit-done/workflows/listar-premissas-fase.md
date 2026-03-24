<purpose>
Exibir as suposições do Claude sobre uma fase antes do planejamento, permitindo que usuários corrijam equívocos cedo.

Diferença chave do discuss-phase: Isto é ANÁLISE do que o Claude pensa, não COLETA do que o usuário sabe. Sem arquivo de saída - puramente conversacional para provocar discussão.
</purpose>

<process>

<step name="validate_phase" priority="first">
Número da fase: {{GSD_ARGS}} (obrigatório)

**Se argumento ausente:**

```
Erro: Número da fase obrigatório.

Uso: /gsd-list-phase-assumptions [número-da-fase]
Exemplo: /gsd-list-phase-assumptions 3
```

Encerrar workflow.

**Se argumento fornecido:**
Validar que a fase existe no roteiro:

```bash
cat .planning/ROADMAP.md | grep -i "Phase ${PHASE}"
```

**Se fase não encontrada:**

```
Erro: Fase ${PHASE} não encontrada no roteiro.

Fases disponíveis:
[listar fases do roteiro]
```

Encerrar workflow.

**Se fase encontrada:**
Analisar detalhes da fase do roteiro:

- Número da fase
- Nome da fase
- Descrição/objetivo da fase
- Quaisquer detalhes de escopo mencionados

Continuar para analyze_phase.
</step>

<step name="analyze_phase">
Baseado na descrição do roteiro e contexto do projeto, identificar suposições em cinco áreas:

**1. Abordagem Técnica:**
Quais bibliotecas, frameworks, padrões ou ferramentas o Claude usaria?
- "Eu usaria a biblioteca X porque..."
- "Eu seguiria o padrão Y porque..."
- "Eu estruturaria isso como Z porque..."

**2. Ordem de Implementação:**
O que o Claude construiria primeiro, segundo, terceiro?
- "Eu começaria com X porque é fundamental"
- "Depois Y porque depende de X"
- "Finalmente Z porque..."

**3. Limites de Escopo:**
O que está incluído vs excluído na interpretação do Claude?
- "Esta fase inclui: A, B, C"
- "Esta fase NÃO inclui: D, E, F"
- "Ambiguidades de limite: G pode ir para qualquer lado"

**4. Áreas de Risco:**
Onde o Claude espera complexidade ou desafios?
- "A parte complicada é X porque..."
- "Problemas potenciais: Y, Z"
- "Eu ficaria atento a..."

**5. Dependências:**
O que o Claude assume que existe ou precisa estar pronto?
- "Isto assume X de fases anteriores"
- "Dependências externas: Y, Z"
- "Isto será consumido por..."

Ser honesto sobre incerteza. Marcar suposições com níveis de confiança:
- "Razoavelmente confiante: ..." (claro do roteiro)
- "Assumindo: ..." (inferência razoável)
- "Incerto: ..." (pode ir para múltiplas direções)
</step>

<step name="present_assumptions">
Apresentar suposições em formato claro e escaneável:

```
## Minhas Suposições para Fase ${PHASE}: ${PHASE_NAME}

### Abordagem Técnica
[Listar suposições sobre como implementar]

### Ordem de Implementação
[Listar suposições sobre sequenciamento]

### Limites de Escopo
**No escopo:** [o que está incluído]
**Fora do escopo:** [o que está excluído]
**Ambíguo:** [o que pode ir para qualquer lado]

### Áreas de Risco
[Listar desafios antecipados]

### Dependências
**De fases anteriores:** [o que é necessário]
**Externas:** [necessidades de terceiros]
**Alimenta:** [o que fases futuras precisam disto]

---

**O que você acha?**

Essas suposições estão corretas? Me diga:
- O que eu acertei
- O que eu errei
- O que está faltando
```

Aguardar resposta do usuário.
</step>

<step name="gather_feedback">
**Se o usuário fornecer correções:**

Reconhecer as correções:

```
Correções principais:
- [correção 1]
- [correção 2]

Isto muda meu entendimento significativamente. [Resumir novo entendimento]
```

**Se o usuário confirmar suposições:**

```
Suposições validadas.
```

Continuar para offer_next.
</step>

<step name="offer_next">
Apresentar próximos passos:

```
Qual o próximo passo?
1. Discutir contexto (/gsd-discuss-phase ${PHASE}) - Deixe-me fazer perguntas para construir contexto abrangente
2. Planejar esta fase (/gsd-plan-phase ${PHASE}) - Criar planos detalhados de execução
3. Re-examinar suposições - Vou analisar novamente com suas correções
4. Pronto por agora
```

Aguardar seleção do usuário.

Se "Discutir contexto": Anotar que CONTEXT.md incorporará quaisquer correções discutidas aqui
Se "Planejar esta fase": Prosseguir sabendo que suposições são entendidas
Se "Re-examinar": Retornar para analyze_phase com entendimento atualizado
</step>

</process>

<success_criteria>
- Número da fase validado contra roteiro
- Suposições exibidas em cinco áreas: abordagem técnica, ordem de implementação, escopo, riscos, dependências
- Níveis de confiança marcados onde apropriado
- Prompt "O que você acha?" apresentado
- Feedback do usuário reconhecido
- Próximos passos claros oferecidos
</success_criteria>
