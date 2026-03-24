# Template de Descoberta

Template para `.planning/phases/XX-nome/DISCOVERY.md` - pesquisa rasa para decisões de biblioteca/opção.

**Propósito:** Responder perguntas "qual biblioteca/opção devemos usar" durante descoberta obrigatória no plan-phase.

Para pesquisa profunda de ecossistema ("como especialistas constroem isso"), use `/gsd-research-phase` que produz RESEARCH.md.

---

## Template do Arquivo

```markdown
---
phase: XX-nome
type: discovery
topic: [tópico-de-descoberta]
---

<session_initialization>
Antes de iniciar a descoberta, verifique a data de hoje:
!`date +%Y-%m-%d`

Use esta data ao pesquisar informação "atual" ou "mais recente".
Exemplo: Se hoje é 2025-11-22, pesquise por "2025" não "2024".
</session_initialization>

<discovery_objective>
Descobrir [tópico] para informar implementação da [nome da fase].

Purpose: [Qual decisão/implementação isto habilita]
Scope: [Limites]
Output: DISCOVERY.md com recomendação
</discovery_objective>

<discovery_scope>
<include>
- [Pergunta a responder]
- [Área a investigar]
- [Comparação específica se necessário]
</include>

<exclude>
- [Fora do escopo desta descoberta]
- [Adiar para fase de implementação]
</exclude>
</discovery_scope>

<discovery_protocol>

**Prioridade de Fontes:**
1. **Context7 MCP** - Para documentação de biblioteca/framework (atual, autoritativa)
2. **Docs Oficiais** - Para bibliotecas específicas de plataforma ou não indexadas
3. **WebSearch** - Para comparações, tendências, padrões da comunidade (verificar todas as descobertas)

**Checklist de Qualidade:**
Antes de completar a descoberta, verifique:
- [ ] Todas as alegações têm fontes autoritativas (Context7 ou docs oficiais)
- [ ] Alegações negativas ("X não é possível") verificadas com documentação oficial
- [ ] Sintaxe/configuração de API do Context7 ou docs oficiais (nunca apenas WebSearch)
- [ ] Descobertas do WebSearch verificadas cruzadamente com fontes autoritativas
- [ ] Atualizações/changelogs recentes verificados para breaking changes
- [ ] Abordagens alternativas consideradas (não apenas primeira solução encontrada)

**Níveis de Confiança:**
- ALTA: Context7 ou docs oficiais confirmam
- MÉDIA: WebSearch + Context7/docs oficiais confirmam
- BAIXA: Apenas WebSearch ou apenas conhecimento do treinamento (marcar para validação)

</discovery_protocol>


<output_structure>
Crie `.planning/phases/XX-nome/DISCOVERY.md`:

```markdown
# Descoberta: [Tópico]

## Resumo
[Resumo executivo de 2-3 parágrafos - o que foi pesquisado, o que foi encontrado, o que é recomendado]

## Recomendação Principal
[O que fazer e por quê - ser específico e acionável]

## Alternativas Consideradas
[O que mais foi avaliado e por que não foi escolhido]

## Descobertas Chave

### [Categoria 1]
- [Descoberta com URL da fonte e relevância para nosso caso]

### [Categoria 2]
- [Descoberta com URL da fonte e relevância]

## Exemplos de Código
[Padrões de implementação relevantes, se aplicável]

## Metadados

<metadata>
<confidence level="high|medium|low">
[Por que este nível de confiança - baseado na qualidade da fonte e verificação]
</confidence>

<sources>
- [Fontes autoritativas primárias usadas]
</sources>

<open_questions>
[O que não pôde ser determinado ou precisa de validação durante implementação]
</open_questions>

<validation_checkpoints>
[Se confiança é BAIXA ou MÉDIA, liste coisas específicas para verificar durante implementação]
</validation_checkpoints>
</metadata>
```
</output_structure>

<success_criteria>
- Todas as perguntas do escopo respondidas com fontes autoritativas
- Itens do checklist de qualidade completados
- Recomendação principal clara
- Descobertas de baixa confiança marcadas com checkpoints de validação
- Pronto para informar criação do PLAN.md
</success_criteria>

<guidelines>
**Quando usar descoberta:**
- Escolha de tecnologia incerta (biblioteca A vs B)
- Melhores práticas necessárias para integração desconhecida
- Investigação de API/biblioteca necessária
- Decisão única pendente

**Quando NÃO usar:**
- Padrões estabelecidos (CRUD, auth com biblioteca conhecida)
- Detalhes de implementação (adiar para execução)
- Perguntas respondíveis pelo contexto existente do projeto

**Quando usar RESEARCH.md ao invés:**
- Domínios nicho/complexos (3D, jogos, áudio, shaders)
- Precisa de conhecimento do ecossistema, não apenas escolha de biblioteca
- Perguntas "como especialistas constroem isso"
- Use `/gsd-research-phase` para estes
</guidelines>
