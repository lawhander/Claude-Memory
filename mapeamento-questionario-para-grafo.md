# Mapeamento do Questionário para Grafo de Conhecimento

## Visão Geral

Este documento demonstra como as respostas do questionário de perfil cognitivo são convertidas em entidades, relações e observações no grafo de conhecimento, criando uma representação estruturada do perfil do usuário.

## Estrutura de Mapeamento

### Entidades Principais

```
USUÁRIO (user_123)
├── PERFIL_COGNITIVO (cognitive_profile_user_123)
├── PADRÕES_PENSAMENTO (thinking_patterns_user_123)
├── CRIATIVIDADE (creativity_user_123)
├── ASPECTOS_EMOCIONAIS (emotional_aspects_user_123)
├── ESTRATÉGIAS_COPING (coping_strategies_user_123)
└── CONTEXTO_PESSOAL (personal_context_user_123)
```

## Exemplo Prático de Conversão

### Resposta Exemplo
Usuário responde:
- Questão 1: "Tenho dificuldade em tomar decisões simples" = **4** (Frequentemente)
- Questão 9: "Tenho muitas ideias simultaneamente" = **5** (Sempre)
- Questão 14: "Fico ansioso quando não encontro solução perfeita" = **4** (Frequentemente)
- Questão 19: Usa meditação (Eficácia: 3), exercícios (Eficácia: 4)
- Questão 23: Área profissional = "Desenvolvimento de Software"

### Conversão para Grafo

```javascript
// Entidades criadas
const entidades = [
  {
    id: "user_123",
    type: "usuario",
    properties: {
      area_profissional: "Desenvolvimento de Software",
      perfil_identificado: "overthinking_moderado_alto",
      data_avaliacao: "2024-05-29"
    }
  },
  {
    id: "decision_making_user_123",
    type: "padrao_comportamental",
    properties: {
      categoria: "tomada_decisao",
      intensidade: 4,
      frequencia: "frequentemente"
    }
  },
  {
    id: "high_creativity_user_123",
    type: "caracteristica_cognitiva",
    properties: {
      categoria: "criatividade",
      intensidade: 5,
      tipo: "geracao_multiplas_ideias"
    }
  },
  {
    id: "perfectionism_anxiety_user_123",
    type: "padrao_emocional",
    properties: {
      categoria: "ansiedade",
      trigger: "solucao_imperfeita",
      intensidade: 4
    }
  },
  {
    id: "meditation_strategy_user_123",
    type: "estrategia_coping",
    properties: {
      tipo: "meditacao",
      eficacia_percebida: 3,
      uso_ativo: true
    }
  },
  {
    id: "exercise_strategy_user_123",
    type: "estrategia_coping",
    properties: {
      tipo: "exercicio_fisico",
      eficacia_percebida: 4,
      uso_ativo: true
    }
  }
];

// Relações estabelecidas
const relacoes = [
  {
    from: "user_123",
    to: "decision_making_user_123",
    type: "apresenta_padrao",
    properties: { confidence: 0.9 }
  },
  {
    from: "user_123",
    to: "high_creativity_user_123",
    type: "possui_caracteristica",
    properties: { confidence: 1.0 }
  },
  {
    from: "user_123",
    to: "perfectionism_anxiety_user_123",
    type: "experiencia_emocional",
    properties: { confidence: 0.9 }
  },
  {
    from: "user_123",
    to: "meditation_strategy_user_123",
    type: "utiliza_estrategia",
    properties: { eficacia: "moderada" }
  },
  {
    from: "user_123",
    to: "exercise_strategy_user_123",
    type: "utiliza_estrategia",
    properties: { eficacia: "alta" }
  },
  {
    from: "high_creativity_user_123",
    to: "decision_making_user_123",
    type: "contribui_para",
    properties: { tipo: "sobrecarga_opcoes" }
  },
  {
    from: "decision_making_user_123",
    to: "perfectionism_anxiety_user_123",
    type: "causa",
    properties: { intensidade: "alta" }
  }
];

// Observações específicas
const observacoes = [
  {
    entityId: "user_123",
    key: "score_overthinking",
    value: 72,
    confidence: 0.95
  },
  {
    entityId: "user_123",
    key: "categoria_perfil",
    value: "overthinking_moderado_alto",
    confidence: 0.9
  },
  {
    entityId: "user_123",
    key: "maior_desafio",
    value: "paralisia_decisoria",
    confidence: 0.85
  },
  {
    entityId: "user_123",
    key: "principal_forca",
    value: "criatividade_alta",
    confidence: 1.0
  },
  {
    entityId: "user_123",
    key: "estrategia_mais_eficaz",
    value: "exercicio_fisico",
    confidence: 0.8
  }
];
```

## Mapeamento Detalhado por Seção

### Seção 1: Padrões de Pensamento

| Questão | Entidade Criada | Tipo | Relação com Usuário |
|---------|----------------|------|---------------------|
| 1-3 (Análise/Decisão) | `decision_patterns_[userID]` | `padrao_cognitivo` | `apresenta_padrao` |
| 4-6 (Processamento) | `information_processing_[userID]` | `estilo_processamento` | `utiliza_estilo` |
| 7-8 (Preocupações) | `worry_patterns_[userID]` | `padrao_emocional` | `experiencia` |

#### Exemplo de Processamento:
```javascript
// Questão 1: Score 4 - "Dificuldade em decisões simples"
function processarQuestao1(score, exemplo) {
  return {
    entidade: {
      id: `decision_difficulty_${userId}`,
      type: "padrao_decisorio",
      properties: {
        intensidade: score,
        tipo: "decisoes_simples",
        exemplo_fornecido: exemplo
      }
    },
    observacao: {
      entityId: userId,
      key: "dificuldade_decisoria",
      value: score >= 4 ? "alta" : score >= 3 ? "moderada" : "baixa",
      confidence: 0.9
    }
  };
}
```

### Seção 2: Criatividade e Ideação

| Questão | Entidade Criada | Observação Gerada |
|---------|----------------|-------------------|
| 9 (Múltiplas ideias) | `idea_generation_[userID]` | `criatividade_nivel: alto/medio/baixo` |
| 10 (Soluções criativas) | `creative_problem_solving_[userID]` | `capacidade_solucao: alta/media/baixa` |
| 12 (Implementação) | `implementation_patterns_[userID]` | `dificuldade_implementacao: sim/nao` |

### Seção 3: Aspectos Emocionais

```javascript
// Mapeamento automático de ansiedade
function mapearAnsiedade(questoes14_16) {
  const nivelAnsiedade = (questoes14_16.reduce((a, b) => a + b) / 3);
  
  return {
    entidade: {
      id: `anxiety_profile_${userId}`,
      type: "perfil_emocional",
      properties: {
        nivel_ansiedade: nivelAnsiedade,
        triggers: extrairTriggers(questoes14_16),
        areas_impacto: identificarAreas(questoes14_16)
      }
    },
    relacoes: [
      {
        from: userId,
        to: `anxiety_profile_${userId}`,
        type: "possui_caracteristica",
        properties: { intensidade: nivelAnsiedade }
      }
    ]
  };
}
```

### Seção 4: Estratégias e Coping

```javascript
// Mapeamento de estratégias
function mapearEstrategias(respostasSecao4) {
  const estrategias = [];
  const relacoes = [];
  
  Object.entries(respostasSecao4.questao19).forEach(([estrategia, dados]) => {
    if (dados.usa) {
      const estrategiaId = `strategy_${estrategia}_${userId}`;
      
      estrategias.push({
        id: estrategiaId,
        type: "estrategia_coping",
        properties: {
          tipo: estrategia,
          eficacia_percebida: dados.eficacia,
          uso_regular: dados.eficacia >= 3
        }
      });
      
      relacoes.push({
        from: userId,
        to: estrategiaId,
        type: "utiliza_estrategia",
        properties: {
          eficacia: dados.eficacia >= 4 ? "alta" : "moderada",
          recomendacao: dados.eficacia >= 4 ? "continuar" : "otimizar"
        }
      });
    }
  });
  
  return { estrategias, relacoes };
}
```

## Algoritmo de Scoring e Categorização

```javascript
function calcularPerfilOverthinking(respostas) {
  // Pesos das seções
  const pesos = {
    padroes_pensamento: 0.35,
    criatividade: 0.20,
    aspectos_emocionais: 0.30,
    estrategias: 0.15
  };
  
  // Cálculo por seção
  const scores = {
    padroes: calcularSecao1(respostas.secao1),
    criatividade: calcularSecao2(respostas.secao2),
    emocional: calcularSecao3(respostas.secao3),
    coping: calcularSecao4(respostas.secao4)
  };
  
  // Score final
  const scoreFinal = 
    (scores.padroes * pesos.padroes_pensamento) +
    (scores.criatividade * pesos.criatividade) +
    (scores.emocional * pesos.aspectos_emocionais) +
    (scores.coping * pesos.estrategias);
  
  // Categorização
  let categoria;
  if (scoreFinal >= 80) categoria = "overthinking_severo";
  else if (scoreFinal >= 60) categoria = "overthinking_moderado_alto";
  else if (scoreFinal >= 40) categoria = "overthinking_moderado";
  else categoria = "pensamento_equilibrado";
  
  return {
    score: scoreFinal,
    categoria,
    detalhes: scores
  };
}
```

## Geração de Insights Automáticos

```javascript
function gerarInsights(grafo, userId) {
  const insights = [];
  
  // Insight 1: Padrão dominante
  const padroesDominantes = grafo.getRelationsFor(userId, 'outgoing')
    .filter(rel => rel.type === 'apresenta_padrao')
    .sort((a, b) => b.properties.intensidade - a.properties.intensidade);
  
  if (padroesDominantes.length > 0) {
    insights.push({
      tipo: "padrao_dominante",
      descricao: `Seu padrão mais forte é ${padroesDominantes[0].to}`,
      acao_sugerida: obterAcaoParaPadrao(padroesDominantes[0].to)
    });
  }
  
  // Insight 2: Estratégia mais eficaz
  const estrategiasMaisEficazes = grafo.getRelationsFor(userId, 'outgoing')
    .filter(rel => rel.type === 'utiliza_estrategia')
    .filter(rel => rel.properties.eficacia === 'alta');
  
  if (estrategiasMaisEficazes.length > 0) {
    insights.push({
      tipo: "estrategia_eficaz",
      descricao: `Suas estratégias mais eficazes são: ${estrategiasMaisEficazes.map(e => e.to).join(', ')}`,
      acao_sugerida: "Considere intensificar o uso dessas estratégias"
    });
  }
  
  // Insight 3: Área de melhoria
  const areasProblematicas = identificarAreasProblematicas(grafo, userId);
  if (areasProblematicas.length > 0) {
    insights.push({
      tipo: "area_melhoria",
      descricao: `Áreas que podem se beneficiar de atenção: ${areasProblematicas.join(', ')}`,
      acao_sugerida: gerarSugestoesMelhoria(areasProblematicas)
    });
  }
  
  return insights;
}
```

## Exemplo de Consulta do Grafo

```javascript
// Consulta para sessão de coaching
function prepararSessaoCoaching(userId) {
  const contexto = grafo.getContextFor(userId, 3);
  
  const resumo = {
    perfil_geral: {
      categoria: contexto.observations.find(o => o.key === 'categoria_perfil')?.value,
      score: contexto.observations.find(o => o.key === 'score_overthinking')?.value
    },
    
    desafios_principais: contexto.connections
      .filter(c => c.relation.type === 'apresenta_padrao')
      .filter(c => c.entity.properties.intensidade >= 4)
      .map(c => c.entity.type),
    
    forcas_identificadas: contexto.connections
      .filter(c => c.relation.type === 'possui_caracteristica')
      .filter(c => c.entity.properties.intensidade >= 4)
      .map(c => c.entity.type),
    
    estrategias_eficazes: contexto.connections
      .filter(c => c.relation.type === 'utiliza_estrategia')
      .filter(c => c.relation.properties.eficacia === 'alta')
      .map(c => c.entity.properties.tipo),
    
    recomendacoes: gerarRecomendacoes(contexto)
  };
  
  return resumo;
}
```

## Evolução e Atualização do Grafo

```javascript
// Sistema de atualização baseada em interações
function atualizarGrafoComInteracao(userId, interacao) {
  const novasObservacoes = [];
  
  // Detectar padrões na interação
  if (interacao.conteudo.includes('não consigo decidir')) {
    novasObservacoes.push({
      entityId: `decision_patterns_${userId}`,
      key: 'ocorrencia_recente',
      value: new Date().toISOString(),
      confidence: 0.8
    });
  }
  
  // Atualizar eficácia de estratégias
  if (interacao.feedback_estrategia) {
    const estrategiaId = `strategy_${interacao.estrategia}_${userId}`;
    const novaEficacia = calcularNovaEficacia(
      estrategiaId, 
      interacao.feedback_estrategia
    );
    
    novasObservacoes.push({
      entityId: estrategiaId,
      key: 'eficacia_atualizada',
      value: novaEficacia,
      confidence: 0.9
    });
  }
  
  return grafo.updateMemory([], [], novasObservacoes);
}
```

## Considerações de Implementação

### Performance
- Indexar entidades por tipo e usuário
- Cache para consultas frequentes de contexto
- Compressão de observações antigas menos relevantes

### Privacidade
- Criptografar dados sensíveis
- Permitir exclusão de dados específicos
- Anonimizar para análises agregadas

### Escalabilidade
- Particionar grafo por usuário
- Implementar limpeza automática de dados antigos
- Usar banco de dados especializado em grafos para grandes volumes

---

*Este mapeamento permite criar um perfil rico e nuanceado que evolui com as interações, fornecendo base sólida para suporte personalizado.*