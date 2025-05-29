# Exemplo de Implementação: Servidor de Memória com Grafo de Conhecimento

## Visão Geral

Este documento apresenta um exemplo prático de implementação de um servidor de memória baseado em grafo de conhecimento em JavaScript, incluindo código completo e instruções para integração com IA generativa.

## Arquitetura do Sistema

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   IA Cliente    │◄──►│ Servidor Memória │◄──►│ Grafo Conhec.   │
│   (Claude/GPT)  │    │   (Node.js)      │    │   (JSON/Neo4j)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## Implementação Base

### 1. Estrutura de Dados do Grafo

```javascript
// models/KnowledgeGraph.js
class KnowledgeGraph {
  constructor() {
    this.entities = new Map(); // id -> entity
    this.relations = new Map(); // id -> relation
    this.observations = new Map(); // entityId -> observations
  }
  
  // Adicionar nova entidade
  addEntity(id, type, properties = {}) {
    const entity = {
      id,
      type,
      properties,
      createdAt: new Date(),
      updatedAt: new Date()
    };
    
    this.entities.set(id, entity);
    return entity;
  }
  
  // Adicionar relação entre entidades
  addRelation(fromId, toId, type, properties = {}) {
    const relationId = `${fromId}-${type}-${toId}`;
    const relation = {
      id: relationId,
      from: fromId,
      to: toId,
      type,
      properties,
      createdAt: new Date()
    };
    
    this.relations.set(relationId, relation);
    return relation;
  }
  
  // Adicionar observação sobre entidade
  addObservation(entityId, key, value, confidence = 1.0) {
    if (!this.observations.has(entityId)) {
      this.observations.set(entityId, []);
    }
    
    const observation = {
      key,
      value,
      confidence,
      timestamp: new Date()
    };
    
    this.observations.get(entityId).push(observation);
    return observation;
  }
  
  // Buscar entidades por tipo
  getEntitiesByType(type) {
    return Array.from(this.entities.values())
      .filter(entity => entity.type === type);
  }
  
  // Buscar relações de uma entidade
  getRelationsFor(entityId, direction = 'both') {
    const relations = Array.from(this.relations.values());
    
    switch(direction) {
      case 'outgoing':
        return relations.filter(rel => rel.from === entityId);
      case 'incoming':
        return relations.filter(rel => rel.to === entityId);
      default:
        return relations.filter(rel => 
          rel.from === entityId || rel.to === entityId
        );
    }
  }
  
  // Buscar contexto relevante
  getContextFor(entityId, maxDepth = 2) {
    const context = {
      entity: this.entities.get(entityId),
      observations: this.observations.get(entityId) || [],
      connections: []
    };
    
    const visited = new Set();
    const queue = [{id: entityId, depth: 0}];
    
    while (queue.length > 0) {
      const {id, depth} = queue.shift();
      
      if (visited.has(id) || depth >= maxDepth) continue;
      visited.add(id);
      
      const relations = this.getRelationsFor(id);
      
      relations.forEach(relation => {
        const connectedId = relation.from === id ? relation.to : relation.from;
        const connectedEntity = this.entities.get(connectedId);
        
        if (connectedEntity) {
          context.connections.push({
            relation,
            entity: connectedEntity,
            observations: this.observations.get(connectedId) || []
          });
          
          if (depth < maxDepth - 1) {
            queue.push({id: connectedId, depth: depth + 1});
          }
        }
      });
    }
    
    return context;
  }
}

module.exports = KnowledgeGraph;
```

### 2. Servidor de Memória

```javascript
// server.js
const express = require('express');
const cors = require('cors');
const KnowledgeGraph = require('./models/KnowledgeGraph');

const app = express();
const port = 3000;

// Instância global do grafo (em produção, usar database)
const graph = new KnowledgeGraph();

app.use(cors());
app.use(express.json());

// Middleware para logging
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Endpoint para adicionar informações ao grafo
app.post('/memory/update', (req, res) => {
  try {
    const { entities, relations, observations } = req.body;
    const results = { added: { entities: 0, relations: 0, observations: 0 } };
    
    // Adicionar entidades
    if (entities) {
      entities.forEach(entity => {
        graph.addEntity(entity.id, entity.type, entity.properties);
        results.added.entities++;
      });
    }
    
    // Adicionar relações
    if (relations) {
      relations.forEach(relation => {
        graph.addRelation(
          relation.from, 
          relation.to, 
          relation.type, 
          relation.properties
        );
        results.added.relations++;
      });
    }
    
    // Adicionar observações
    if (observations) {
      observations.forEach(obs => {
        graph.addObservation(
          obs.entityId, 
          obs.key, 
          obs.value, 
          obs.confidence
        );
        results.added.observations++;
      });
    }
    
    res.json({ success: true, results });
  } catch (error) {
    console.error('Erro ao atualizar memória:', error);
    res.status(500).json({ success: false, error: error.message });
  }
});

// Endpoint para buscar contexto
app.get('/memory/context/:entityId', (req, res) => {
  try {
    const { entityId } = req.params;
    const { maxDepth = 2 } = req.query;
    
    const context = graph.getContextFor(entityId, parseInt(maxDepth));
    
    if (!context.entity) {
      return res.status(404).json({ 
        success: false, 
        error: 'Entidade não encontrada' 
      });
    }
    
    res.json({ success: true, context });
  } catch (error) {
    console.error('Erro ao buscar contexto:', error);
    res.status(500).json({ success: false, error: error.message });
  }
});

// Endpoint para buscar entidades por tipo
app.get('/memory/entities/:type', (req, res) => {
  try {
    const { type } = req.params;
    const entities = graph.getEntitiesByType(type);
    
    res.json({ success: true, entities });
  } catch (error) {
    console.error('Erro ao buscar entidades:', error);
    res.status(500).json({ success: false, error: error.message });
  }
});

// Endpoint para buscar informações gerais sobre o grafo
app.get('/memory/stats', (req, res) => {
  try {
    const stats = {
      entities: graph.entities.size,
      relations: graph.relations.size,
      observations: Array.from(graph.observations.values())
        .reduce((total, obs) => total + obs.length, 0)
    };
    
    res.json({ success: true, stats });
  } catch (error) {
    console.error('Erro ao obter estatísticas:', error);
    res.status(500).json({ success: false, error: error.message });
  }
});

app.listen(port, () => {
  console.log(`Servidor de memória rodando em http://localhost:${port}`);
});
```

### 3. Cliente para Integração com IA

```javascript
// client/MemoryClient.js
class MemoryClient {
  constructor(baseUrl = 'http://localhost:3000') {
    this.baseUrl = baseUrl;
  }
  
  async updateMemory(entities, relations, observations) {
    try {
      const response = await fetch(`${this.baseUrl}/memory/update`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ entities, relations, observations })
      });
      
      return await response.json();
    } catch (error) {
      console.error('Erro ao atualizar memória:', error);
      throw error;
    }
  }
  
  async getContext(entityId, maxDepth = 2) {
    try {
      const response = await fetch(
        `${this.baseUrl}/memory/context/${entityId}?maxDepth=${maxDepth}`
      );
      
      return await response.json();
    } catch (error) {
      console.error('Erro ao buscar contexto:', error);
      throw error;
    }
  }
  
  async getEntitiesByType(type) {
    try {
      const response = await fetch(`${this.baseUrl}/memory/entities/${type}`);
      return await response.json();
    } catch (error) {
      console.error('Erro ao buscar entidades:', error);
      throw error;
    }
  }
}

module.exports = MemoryClient;
```

## Exemplo de Uso com IA

### Integração com Claude/GPT

```javascript
// example/ai-integration.js
const MemoryClient = require('../client/MemoryClient');

class AIWithMemory {
  constructor() {
    this.memory = new MemoryClient();
    this.userId = 'user-123'; // ID único do usuário
  }
  
  async processUserMessage(message, userId = this.userId) {
    // 1. Buscar contexto do usuário
    const userContext = await this.getUserContext(userId);
    
    // 2. Processar mensagem com IA (simulado)
    const aiResponse = await this.generateResponse(message, userContext);
    
    // 3. Extrair e salvar nova informação
    await this.extractAndSaveInformation(message, aiResponse, userId);
    
    return aiResponse;
  }
  
  async getUserContext(userId) {
    try {
      const result = await this.memory.getContext(userId);
      return result.success ? result.context : null;
    } catch (error) {
      console.error('Erro ao buscar contexto do usuário:', error);
      return null;
    }
  }
  
  async generateResponse(message, context) {
    // Simular chamada para IA com contexto
    const prompt = this.buildPromptWithContext(message, context);
    
    // Aqui você faria a chamada real para Claude/GPT
    // const response = await claude.generate(prompt);
    
    // Simulação para exemplo
    return `Resposta da IA considerando contexto: ${message}`;
  }
  
  buildPromptWithContext(message, context) {
    let prompt = `Mensagem do usuário: ${message}\n\n`;
    
    if (context) {
      prompt += `Contexto do usuário:\n`;
      
      if (context.observations && context.observations.length > 0) {
        prompt += `Observações:\n`;
        context.observations.forEach(obs => {
          prompt += `- ${obs.key}: ${obs.value}\n`;
        });
      }
      
      if (context.connections && context.connections.length > 0) {
        prompt += `\nConexões relevantes:\n`;
        context.connections.forEach(conn => {
          prompt += `- ${conn.relation.type}: ${conn.entity.type}\n`;
        });
      }
    }
    
    return prompt;
  }
  
  async extractAndSaveInformation(userMessage, aiResponse, userId) {
    // Lógica para extrair informações da conversa
    // Este é um exemplo simplificado
    
    const entities = [];
    const relations = [];
    const observations = [];
    
    // Garantir que o usuário existe como entidade
    entities.push({
      id: userId,
      type: 'user',
      properties: { lastInteraction: new Date() }
    });
    
    // Exemplo: detectar se o usuário mencionou um projeto
    if (userMessage.toLowerCase().includes('projeto')) {
      const projectId = `project-${Date.now()}`;
      
      entities.push({
        id: projectId,
        type: 'project',
        properties: { mentioned: new Date() }
      });
      
      relations.push({
        from: userId,
        to: projectId,
        type: 'trabalha_em',
        properties: { confidence: 0.8 }
      });
    }
    
    // Salvar informações extraídas
    if (entities.length > 0 || relations.length > 0 || observations.length > 0) {
      await this.memory.updateMemory(entities, relations, observations);
    }
  }
}

// Exemplo de uso
async function exemplo() {
  const ai = new AIWithMemory();
  
  // Simulação de conversa
  const response1 = await ai.processUserMessage(
    "Olá, estou trabalhando em um projeto de machine learning"
  );
  console.log('Resposta 1:', response1);
  
  const response2 = await ai.processUserMessage(
    "Preciso de ajuda com overfitting no meu modelo"
  );
  console.log('Resposta 2:', response2);
}

// exemplo();
```

## Instruções de Instalação

### 1. Dependências

```bash
npm init -y
npm install express cors
```

### 2. Estrutura de Arquivos

```
project/
├── package.json
├── server.js
├── models/
│   └── KnowledgeGraph.js
├── client/
│   └── MemoryClient.js
└── example/
    └── ai-integration.js
```

### 3. Executar

```bash
node server.js
```

## Próximos Passos

1. **Persistência**: Integrar com banco de dados real (Neo4j, MongoDB)
2. **Segurança**: Adicionar autenticação e autorização
3. **Performance**: Implementar cache e otimizações
4. **Interface**: Criar dashboard para visualização do grafo
5. **IA**: Integrar com APIs reais de IA (Claude, OpenAI)

## Considerações de Produção

- **Escalabilidade**: Para muitos usuários, considerar sharding
- **Backup**: Implementar backup automático regular
- **Monitoramento**: Adicionar logs e métricas
- **Privacidade**: Criptografar dados sensíveis
- **Rate Limiting**: Proteger contra uso excessivo

---

*Este é um exemplo base que pode ser expandido conforme necessidades específicas.*