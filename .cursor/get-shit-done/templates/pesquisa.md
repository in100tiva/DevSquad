# Template de Pesquisa

Template para `.planning/phases/XX-nome/{num_fase}-RESEARCH.md` - pesquisa abrangente de ecossistema antes do planejamento.

**Propósito:** Documentar o que o Claude precisa saber para implementar bem uma fase - não apenas "qual biblioteca" mas "como especialistas constroem isso."

---

## Template do Arquivo

```markdown
# Fase [X]: [Nome] - Pesquisa

**Pesquisado:** [data]
**Domínio:** [tecnologia principal/domínio do problema]
**Confiança:** [ALTA/MÉDIA/BAIXA]

<user_constraints>
## Restrições do Usuário (do CONTEXT.md)

**CRÍTICO:** Se CONTEXT.md existe do /gsd-discuss-phase, copie decisões travadas aqui literalmente. Estas DEVEM ser respeitadas pelo planejador.

### Decisões Travadas
[Copiar da seção `## Decisões` do CONTEXT.md - estas são NÃO-NEGOCIÁVEIS]
- [Decisão 1]
- [Decisão 2]

### Critério do Claude
[Copiar do CONTEXT.md - áreas onde pesquisador/planejador pode escolher]
- [Área 1]
- [Área 2]

### Ideias Adiadas (FORA DO ESCOPO)
[Copiar do CONTEXT.md - NÃO pesquisar ou planejar estas]
- [Adiada 1]
- [Adiada 2]

**Se CONTEXT.md não existe:** Escreva "Sem restrições do usuário - todas as decisões a critério do Claude"
</user_constraints>

<research_summary>
## Resumo

[Resumo executivo de 2-3 parágrafos]
- O que foi pesquisado
- Qual é a abordagem padrão
- Recomendações chave

**Recomendação principal:** [orientação acionável de uma linha]
</research_summary>

<standard_stack>
## Stack Padrão

As bibliotecas/ferramentas estabelecidas para este domínio:

### Core
| Biblioteca | Versão | Propósito | Por que é Padrão |
|------------|--------|-----------|------------------|
| [nome] | [ver] | [o que faz] | [por que especialistas usam] |
| [nome] | [ver] | [o que faz] | [por que especialistas usam] |

### Suporte
| Biblioteca | Versão | Propósito | Quando Usar |
|------------|--------|-----------|-------------|
| [nome] | [ver] | [o que faz] | [caso de uso] |
| [nome] | [ver] | [o que faz] | [caso de uso] |

### Alternativas Consideradas
| Ao invés de | Poderia Usar | Trade-off |
|-------------|--------------|-----------|
| [padrão] | [alternativa] | [quando alternativa faz sentido] |

**Instalação:**
```bash
npm install [pacotes]
# ou
yarn add [pacotes]
```
</standard_stack>

<architecture_patterns>
## Padrões de Arquitetura

### Estrutura de Projeto Recomendada
```
src/
├── [pasta]/        # [propósito]
├── [pasta]/        # [propósito]
└── [pasta]/        # [propósito]
```

### Padrão 1: [Nome do Padrão]
**O quê:** [descrição]
**Quando usar:** [condições]
**Exemplo:**
```typescript
// [exemplo de código do Context7/docs oficiais]
```

### Padrão 2: [Nome do Padrão]
**O quê:** [descrição]
**Quando usar:** [condições]
**Exemplo:**
```typescript
// [exemplo de código]
```

### Anti-Padrões a Evitar
- **[Anti-padrão]:** [por que é ruim, o que fazer em vez]
- **[Anti-padrão]:** [por que é ruim, o que fazer em vez]
</architecture_patterns>

<dont_hand_roll>
## Não Implemente do Zero

Problemas que parecem simples mas têm soluções existentes:

| Problema | Não Construa | Use Ao Invés | Por quê |
|----------|--------------|--------------|---------|
| [problema] | [o que você construiria] | [biblioteca] | [edge cases, complexidade] |
| [problema] | [o que você construiria] | [biblioteca] | [edge cases, complexidade] |
| [problema] | [o que você construiria] | [biblioteca] | [edge cases, complexidade] |

**Insight chave:** [por que soluções customizadas são piores neste domínio]
</dont_hand_roll>

<common_pitfalls>
## Armadilhas Comuns

### Armadilha 1: [Nome]
**O que dá errado:** [descrição]
**Por que acontece:** [causa raiz]
**Como evitar:** [estratégia de prevenção]
**Sinais de alerta:** [como detectar cedo]

### Armadilha 2: [Nome]
**O que dá errado:** [descrição]
**Por que acontece:** [causa raiz]
**Como evitar:** [estratégia de prevenção]
**Sinais de alerta:** [como detectar cedo]

### Armadilha 3: [Nome]
**O que dá errado:** [descrição]
**Por que acontece:** [causa raiz]
**Como evitar:** [estratégia de prevenção]
**Sinais de alerta:** [como detectar cedo]
</common_pitfalls>

<code_examples>
## Exemplos de Código

Padrões verificados de fontes oficiais:

### [Operação Comum 1]
```typescript
// Fonte: [URL do Context7/docs oficiais]
[código]
```

### [Operação Comum 2]
```typescript
// Fonte: [URL do Context7/docs oficiais]
[código]
```

### [Operação Comum 3]
```typescript
// Fonte: [URL do Context7/docs oficiais]
[código]
```
</code_examples>

<sota_updates>
## Estado da Arte (2024-2025)

O que mudou recentemente:

| Abordagem Antiga | Abordagem Atual | Quando Mudou | Impacto |
|-------------------|-----------------|--------------|---------|
| [antigo] | [novo] | [data/versão] | [o que significa para implementação] |

**Novas ferramentas/padrões a considerar:**
- [Ferramenta/Padrão]: [o que habilita, quando usar]
- [Ferramenta/Padrão]: [o que habilita, quando usar]

**Depreciados/desatualizados:**
- [Coisa]: [por que está desatualizado, o que substituiu]
</sota_updates>

<open_questions>
## Questões em Aberto

Coisas que não puderam ser totalmente resolvidas:

1. **[Questão]**
   - O que sabemos: [info parcial]
   - O que não está claro: [a lacuna]
   - Recomendação: [como lidar durante planejamento/execução]

2. **[Questão]**
   - O que sabemos: [info parcial]
   - O que não está claro: [a lacuna]
   - Recomendação: [como lidar]
</open_questions>

<sources>
## Fontes

### Primárias (confiança ALTA)
- [ID da biblioteca Context7] - [tópicos buscados]
- [URL docs oficiais] - [o que foi verificado]

### Secundárias (confiança MÉDIA)
- [WebSearch verificado com fonte oficial] - [descoberta + verificação]

### Terciárias (confiança BAIXA - precisa validação)
- [Apenas WebSearch] - [descoberta, marcada para validação durante implementação]
</sources>

<metadata>
## Metadados

**Escopo da pesquisa:**
- Tecnologia core: [o quê]
- Ecossistema: [bibliotecas exploradas]
- Padrões: [padrões pesquisados]
- Armadilhas: [áreas verificadas]

**Detalhamento de confiança:**
- Stack padrão: [ALTA/MÉDIA/BAIXA] - [motivo]
- Arquitetura: [ALTA/MÉDIA/BAIXA] - [motivo]
- Armadilhas: [ALTA/MÉDIA/BAIXA] - [motivo]
- Exemplos de código: [ALTA/MÉDIA/BAIXA] - [motivo]

**Data da pesquisa:** [data]
**Válido até:** [estimativa - 30 dias para tech estável, 7 dias para tech em rápida evolução]
</metadata>

---

*Fase: XX-nome*
*Pesquisa concluída: [data]*
*Pronto para planejamento: [sim/não]*
```

---

## Bom Exemplo

```markdown
# Fase 3: Direção 3D na Cidade - Pesquisa

**Pesquisado:** 2025-01-20
**Domínio:** Jogo web 3D com Three.js e mecânicas de direção
**Confiança:** ALTA

<research_summary>
## Resumo

Pesquisou o ecossistema Three.js para construir um jogo de direção 3D na cidade. A abordagem padrão usa Three.js com React Three Fiber para arquitetura de componentes, Rapier para física e drei para helpers comuns.

Descoberta chave: Não implemente física ou detecção de colisão do zero. Rapier (via @react-three/rapier) lida com física de veículos, colisão com terreno e interações com objetos da cidade eficientemente. Código de física customizado leva a bugs e problemas de performance.

**Recomendação principal:** Use stack R3F + Rapier + drei. Comece com controlador de veículo do drei, adicione física de veículo Rapier, construa cidade com instanced meshes para performance.
</research_summary>

<standard_stack>
## Stack Padrão

### Core
| Biblioteca | Versão | Propósito | Por que é Padrão |
|------------|--------|-----------|------------------|
| three | 0.160.0 | Renderização 3D | O padrão para 3D web |
| @react-three/fiber | 8.15.0 | Renderer React para Three.js | 3D declarativo, melhor DX |
| @react-three/drei | 9.92.0 | Helpers e abstrações | Resolve problemas comuns |
| @react-three/rapier | 1.2.1 | Bindings de engine de física | Melhor física para R3F |

### Suporte
| Biblioteca | Versão | Propósito | Quando Usar |
|------------|--------|-----------|-------------|
| @react-three/postprocessing | 2.16.0 | Efeitos visuais | Bloom, DOF, motion blur |
| leva | 0.9.35 | UI de debug | Ajustar parâmetros |
| zustand | 4.4.7 | Gerenciamento de estado | Estado do jogo, estado da UI |
| use-sound | 4.0.1 | Áudio | Sons do motor, ambiente |

### Alternativas Consideradas
| Ao invés de | Poderia Usar | Trade-off |
|-------------|--------------|-----------|
| Rapier | Cannon.js | Cannon mais simples mas menos performante para veículos |
| R3F | Vanilla Three | Vanilla se sem React, mas DX do R3F é muito melhor |
| drei | Helpers customizados | drei é testado em batalha, não reinvente |

**Instalação:**
```bash
npm install three @react-three/fiber @react-three/drei @react-three/rapier zustand
```
</standard_stack>

<architecture_patterns>
## Padrões de Arquitetura

### Estrutura de Projeto Recomendada
```
src/
├── components/
│   ├── Vehicle/          # Carro do jogador com física
│   ├── City/             # Geração de cidade e prédios
│   ├── Road/             # Rede de estradas
│   └── Environment/      # Céu, iluminação, névoa
├── hooks/
│   ├── useVehicleControls.ts
│   └── useGameState.ts
├── stores/
│   └── gameStore.ts      # Estado Zustand
└── utils/
    └── cityGenerator.ts  # Helpers de geração procedural
```

### Padrão 1: Veículo com Física Rapier
**O quê:** Usar RigidBody com configurações específicas de veículo, não física customizada
**Quando usar:** Qualquer veículo terrestre
**Exemplo:**
```typescript
// Fonte: docs do @react-three/rapier
import { RigidBody, useRapier } from '@react-three/rapier'

function Vehicle() {
  const rigidBody = useRef()

  return (
    <RigidBody
      ref={rigidBody}
      type="dynamic"
      colliders="hull"
      mass={1500}
      linearDamping={0.5}
      angularDamping={0.5}
    >
      <mesh>
        <boxGeometry args={[2, 1, 4]} />
        <meshStandardMaterial />
      </mesh>
    </RigidBody>
  )
}
```

### Padrão 2: Instanced Meshes para Cidade
**O quê:** Usar InstancedMesh para objetos repetidos (prédios, árvores, props)
**Quando usar:** >100 objetos similares
**Exemplo:**
```typescript
// Fonte: docs do drei
import { Instances, Instance } from '@react-three/drei'

function Buildings({ positions }) {
  return (
    <Instances limit={1000}>
      <boxGeometry />
      <meshStandardMaterial />
      {positions.map((pos, i) => (
        <Instance key={i} position={pos} scale={[1, Math.random() * 5 + 1, 1]} />
      ))}
    </Instances>
  )
}
```

### Anti-Padrões a Evitar
- **Criar meshes no render loop:** Crie uma vez, atualize apenas transforms
- **Não usar InstancedMesh:** Meshes individuais para prédios mata a performance
- **Matemática de física customizada:** Rapier lida melhor, sempre
</architecture_patterns>

<dont_hand_roll>
## Não Implemente do Zero

| Problema | Não Construa | Use Ao Invés | Por quê |
|----------|--------------|--------------|---------|
| Física de veículo | Velocidade/aceleração customizada | Rapier RigidBody | Fricção de rodas, suspensão, colisões são complexos |
| Detecção de colisão | Raycasting de tudo | Rapier colliders | Performance, edge cases, tunneling |
| Camera follow | Lerp manual | drei CameraControls ou customizado com useFrame | Interpolação suave, limites |
| Geração de cidade | Posicionamento puramente aleatório | Baseado em grid com noise para variação | Aleatório parece errado, grid é previsível |
| LOD | Verificações manuais de distância | drei <Detailed> | Lida com transições, histerese |

**Insight chave:** Desenvolvimento de jogos 3D tem 40+ anos de problemas resolvidos. Rapier implementa simulação de física adequada. drei implementa helpers 3D adequados. Lutar contra estes leva a bugs que parecem problemas de "game feel" mas na verdade são edge cases de física.
</dont_hand_roll>

<common_pitfalls>
## Armadilhas Comuns

### Armadilha 1: Tunneling de Física
**O que dá errado:** Objetos rápidos passam através de paredes
**Por que acontece:** Passo de física padrão muito grande para a velocidade
**Como evitar:** Usar CCD (Continuous Collision Detection) no Rapier
**Sinais de alerta:** Objetos aparecendo aleatoriamente fora de prédios

### Armadilha 2: Morte por Draw Calls na Performance
**O que dá errado:** Jogo trava com muitos prédios
**Por que acontece:** Cada mesh = 1 draw call, centenas de prédios = centenas de calls
**Como evitar:** InstancedMesh para objetos similares, mesclar geometria estática
**Sinais de alerta:** GPU limitada, FPS baixo apesar de cena simples

### Armadilha 3: Veículo com Sensação "Flutuante"
**O que dá errado:** Carro não parece estar no chão
**Por que acontece:** Simulação de roda/suspensão inadequada
**Como evitar:** Usar controlador de veículo Rapier ou ajustar massa/damping cuidadosamente
**Sinais de alerta:** Carro quica estranhamente, não gripa nas curvas
</common_pitfalls>

<code_examples>
## Exemplos de Código

### Setup Básico R3F + Rapier
```typescript
// Fonte: getting started do @react-three/rapier
import { Canvas } from '@react-three/fiber'
import { Physics } from '@react-three/rapier'

function Game() {
  return (
    <Canvas>
      <Physics gravity={[0, -9.81, 0]}>
        <Vehicle />
        <City />
        <Ground />
      </Physics>
    </Canvas>
  )
}
```

### Hook de Controles do Veículo
```typescript
// Fonte: Padrão da comunidade, verificado com docs do drei
import { useFrame } from '@react-three/fiber'
import { useKeyboardControls } from '@react-three/drei'

function useVehicleControls(rigidBodyRef) {
  const [, getKeys] = useKeyboardControls()

  useFrame(() => {
    const { forward, back, left, right } = getKeys()
    const body = rigidBodyRef.current
    if (!body) return

    const impulse = { x: 0, y: 0, z: 0 }
    if (forward) impulse.z -= 10
    if (back) impulse.z += 5

    body.applyImpulse(impulse, true)

    if (left) body.applyTorqueImpulse({ x: 0, y: 2, z: 0 }, true)
    if (right) body.applyTorqueImpulse({ x: 0, y: -2, z: 0 }, true)
  })
}
```
</code_examples>

<sota_updates>
## Estado da Arte (2024-2025)

| Abordagem Antiga | Abordagem Atual | Quando Mudou | Impacto |
|-------------------|-----------------|--------------|---------|
| cannon-es | Rapier | 2023 | Rapier é mais rápido, melhor mantido |
| vanilla Three.js | React Three Fiber | 2020+ | R3F agora é padrão para apps React |
| InstancedMesh Manual | drei <Instances> | 2022 | API mais simples, lida com updates |

**Novas ferramentas/padrões a considerar:**
- **WebGPU:** Chegando mas não production-ready para jogos ainda (2025)
- **drei Gltf helpers:** <useGLTF.preload> para telas de carregamento

**Depreciados/desatualizados:**
- **cannon.js (original):** Use fork cannon-es ou melhor, Rapier
- **Raycasting manual para física:** Apenas use Rapier colliders
</sota_updates>

<sources>
## Fontes

### Primárias (confiança ALTA)
- /pmndrs/react-three-fiber - getting started, hooks, performance
- /pmndrs/drei - instances, controls, helpers
- /dimforge/rapier-js - setup de física, física de veículos

### Secundárias (confiança MÉDIA)
- Threads do Three.js discourse "city driving game" - padrões verificados contra docs
- Repositório de exemplos R3F - código verificado funciona

### Terciárias (confiança BAIXA - precisa validação)
- Nenhuma - todas as descobertas verificadas
</sources>

<metadata>
## Metadados

**Escopo da pesquisa:**
- Tecnologia core: Three.js + React Three Fiber
- Ecossistema: Rapier, drei, zustand
- Padrões: Física de veículos, instanciamento, geração de cidade
- Armadilhas: Performance, física, sensação

**Detalhamento de confiança:**
- Stack padrão: ALTA - verificado com Context7, amplamente usado
- Arquitetura: ALTA - de exemplos oficiais
- Armadilhas: ALTA - documentadas no discourse, verificadas nos docs
- Exemplos de código: ALTA - do Context7/fontes oficiais

**Data da pesquisa:** 2025-01-20
**Válido até:** 2025-02-20 (30 dias - ecossistema R3F estável)
</metadata>

---

*Fase: 03-direcao-na-cidade*
*Pesquisa concluída: 2025-01-20*
*Pronto para planejamento: sim*
```

---

## Diretrizes

**Quando criar:**
- Antes de planejar fases em domínios nicho/complexos
- Quando dados de treinamento do Claude provavelmente estão desatualizados ou escassos
- Quando "como especialistas fazem isso" importa mais que "qual biblioteca"

**Estrutura:**
- Use tags XML para marcadores de seção (corresponde aos templates GSD)
- Sete seções core: summary, standard_stack, architecture_patterns, dont_hand_roll, common_pitfalls, code_examples, sources
- Todas as seções obrigatórias (impulsiona pesquisa abrangente)

**Qualidade do conteúdo:**
- Stack padrão: Versões específicas, não apenas nomes
- Arquitetura: Inclua exemplos de código reais de fontes autoritativas
- Não implemente do zero: Seja explícito sobre quais problemas NÃO resolver sozinho
- Armadilhas: Inclua sinais de alerta, não apenas "não faça isso"
- Fontes: Marque níveis de confiança honestamente

**Integração com planejamento:**
- RESEARCH.md carregado como referência @context no PLAN.md
- Stack padrão informa escolhas de biblioteca
- Não implemente do zero previne soluções customizadas
- Armadilhas informam critérios de verificação
- Exemplos de código podem ser referenciados em ações de tarefas

**Após criação:**
- Arquivo fica no diretório da fase: `.planning/phases/XX-nome/{num_fase}-RESEARCH.md`
- Referenciado durante workflow de planejamento
- plan-phase carrega automaticamente quando presente
