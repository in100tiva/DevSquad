# Perfis de Modelo

Perfis de modelo controlam qual modelo Claude cada agente GSD usa. Isso permite equilibrar qualidade vs gasto de tokens, ou herdar o modelo de sessão atualmente selecionado.

## Definições de Perfil

| Agente | `quality` | `balanced` | `budget` | `inherit` |
|--------|-----------|------------|----------|-----------|
| gsd-planner | opus | opus | sonnet | inherit |
| gsd-roadmapper | opus | sonnet | sonnet | inherit |
| gsd-executor | opus | sonnet | sonnet | inherit |
| gsd-phase-researcher | opus | sonnet | haiku | inherit |
| gsd-project-researcher | opus | sonnet | haiku | inherit |
| gsd-research-synthesizer | sonnet | sonnet | haiku | inherit |
| gsd-debugger | opus | sonnet | sonnet | inherit |
| gsd-codebase-mapper | sonnet | haiku | haiku | inherit |
| gsd-verifier | sonnet | sonnet | haiku | inherit |
| gsd-plan-checker | sonnet | sonnet | haiku | inherit |
| gsd-integration-checker | sonnet | sonnet | haiku | inherit |
| gsd-nyquist-auditor | sonnet | sonnet | haiku | inherit |

## Filosofia dos Perfis

**quality** - Máximo poder de raciocínio
- Opus para todos os agentes de tomada de decisão
- Sonnet para verificação somente-leitura
- Usar quando: cota disponível, trabalho de arquitetura crítico

**balanced** (padrão) - Alocação inteligente
- Opus somente para planejamento (onde decisões de arquitetura acontecem)
- Sonnet para execução e pesquisa (segue instruções explícitas)
- Sonnet para verificação (precisa de raciocínio, não apenas reconhecimento de padrões)
- Usar quando: desenvolvimento normal, bom equilíbrio de qualidade e custo

**budget** - Uso mínimo de Opus
- Sonnet para qualquer coisa que escreva código
- Haiku para pesquisa e verificação
- Usar quando: conservando cota, trabalho de alto volume, fases menos críticas

**inherit** - Seguir o modelo da sessão atual
- Todos os agentes resolvem para `inherit`
- Melhor quando você troca modelos interativamente (por exemplo OpenCode `/model`)
- **Obrigatório quando usando provedores não-Anthropic** (OpenRouter, modelos locais, etc.) — caso contrário GSD pode chamar modelos Anthropic diretamente, gerando custos inesperados
- Usar quando: você quer que GSD siga seu modelo de runtime atualmente selecionado

## Usando Runtimes Não-Claude (Codex, OpenCode, Gemini CLI)

Quando instalado para um runtime não-Claude, o instalador GSD define `resolve_model_ids: "omit"` em `~/.gsd/defaults.json`. Isso retorna um parâmetro de modelo vazio para todos os agentes, então cada agente usa o modelo padrão do runtime. Nenhuma configuração manual é necessária.

Para atribuir modelos diferentes a agentes diferentes, adicione `model_overrides` com IDs de modelo que seu runtime reconhece:

```json
{
  "resolve_model_ids": "omit",
  "model_overrides": {
    "gsd-planner": "o3",
    "gsd-executor": "o4-mini",
    "gsd-debugger": "o3",
    "gsd-codebase-mapper": "o4-mini"
  }
}
```

A mesma lógica de camadas se aplica: modelos mais fortes para planejamento e debugging, modelos mais baratos para execução e mapeamento.

## Usando Cursor com Provedores Não-Anthropic (OpenRouter, Local)

Se você está usando Cursor com OpenRouter, um modelo local, ou qualquer provedor não-Anthropic, defina o perfil `inherit` para evitar que GSD chame modelos Anthropic para subagentes:

```bash
# Via comando de configurações
/gsd-settings
# → Selecione "Inherit" para perfil de modelo

# Ou manualmente em .planning/config.json
{
  "model_profile": "inherit"
}
```

Sem `inherit`, o perfil padrão `balanced` do GSD cria modelos Anthropic específicos (`opus`, `sonnet`, `haiku`) para cada tipo de agente, o que pode resultar em custos adicionais de API através do seu provedor não-Anthropic.

## Lógica de Resolução

Orquestradores resolvem modelo antes de criar:

```
1. Ler .planning/config.json
2. Verificar model_overrides para override específico do agente
3. Se sem override, consultar agente na tabela de perfil
4. Passar parâmetro de modelo para chamada Task
```

## Overrides Por Agente

Substituir agentes específicos sem mudar o perfil inteiro:

```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "gsd-executor": "opus",
    "gsd-planner": "haiku"
  }
}
```

Overrides têm precedência sobre o perfil. Valores válidos: `opus`, `sonnet`, `haiku`, `inherit`, ou qualquer ID de modelo totalmente qualificado (ex: `"o3"`, `"openai/o3"`, `"google/gemini-2.5-pro"`).

## Trocando Perfis

Runtime: `/gsd-set-profile <perfil>`

Padrão por projeto: Definir em `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Justificativa de Design

**Por que Opus para gsd-planner?**
Planejamento envolve decisões de arquitetura, decomposição de objetivos e design de tarefas. É onde a qualidade do modelo tem maior impacto.

**Por que Sonnet para gsd-executor?**
Executores seguem instruções explícitas do PLAN.md. O plano já contém o raciocínio; execução é implementação.

**Por que Sonnet (não Haiku) para verificadores no balanced?**
Verificação requer raciocínio de trás para frente - verificar se código *entrega* o que a fase prometeu, não apenas reconhecimento de padrões. Sonnet lida bem com isso; Haiku pode perder lacunas sutis.

**Por que Haiku para gsd-codebase-mapper?**
Exploração somente-leitura e extração de padrões. Sem raciocínio necessário, apenas saída estruturada do conteúdo de arquivos.

**Por que `inherit` em vez de passar `opus` diretamente?**
O alias `"opus"` do Cursor mapeia para uma versão específica de modelo. Organizações podem bloquear versões mais antigas do opus enquanto permitem mais novas. GSD retorna `"inherit"` para agentes de nível opus, fazendo-os usar qualquer versão de opus que o usuário tenha configurado em sua sessão. Isso evita conflitos de versão e fallbacks silenciosos para Sonnet.

**Por que perfil `inherit`?**
Alguns runtimes (incluindo OpenCode) permitem que usuários troquem modelos em runtime (`/model`). O perfil `inherit` mantém todos os subagentes GSD alinhados a essa seleção ao vivo.
