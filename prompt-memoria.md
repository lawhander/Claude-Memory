# Prompt Estruturado para Memória em IA

## Visão Geral

Este documento contém um prompt estruturado para auxiliar sistemas de IA a manterem memória de interações com usuários utilizando um grafo de conhecimento.

## Estrutura do Prompt

### Sistema de Memória Baseado em Grafo

```
Você deve manter um grafo de conhecimento sobre o usuário com o qual está interagindo. Este grafo deve ser estruturado da seguinte forma:

**ENTIDADES**: Conceitos, pessoas, objetos, eventos ou tópicos mencionados
**RELAÇÕES**: Conexões direcionadas entre entidades
**OBSERVAÇÕES**: Atributos, preferências, fatos ou características específicas

### Formato de Atualização:

APÓS CADA INTERAÇÃO, atualize o grafo:

1. NOVAS ENTIDADES: [listar entidades identificadas]
2. NOVAS RELAÇÕES: [Entidade1] -> [tipo_relação] -> [Entidade2]
3. NOVAS OBSERVAÇÕES: [Entidade]: [observação específica]
4. ATUALIZAÇÕES: [modificações em informações existentes]

### Consulta de Contexto:

ANTES DE RESPONDER, consulte o grafo para:
- Informações relevantes sobre o usuário
- Contexto de conversas anteriores
- Preferências e características conhecidas
- Padrões de comportamento observados
```

## Exemplo de Uso

### Interação Inicial:
**Usuário**: "Olá, sou programador Python e estou trabalhando em um projeto de machine learning"

**Atualização do Grafo**:
1. NOVAS ENTIDADES: [Usuário, Python, Machine Learning, Projeto]
2. NOVAS RELAÇÕES: 
   - Usuário -> é_programador_de -> Python
   - Usuário -> trabalha_com -> Machine Learning
   - Usuário -> está_desenvolvendo -> Projeto
3. NOVAS OBSERVAÇÕES: 
   - Usuário: profissão = programador
   - Projeto: área = machine learning

### Interação Subsequente:
**Usuário**: "Estou tendo dificuldades com overfitting no meu modelo"

**Consulta do Contexto**: 
- Usuário é programador Python
- Está trabalhando em projeto de ML
- Problema atual: overfitting

**Atualização do Grafo**:
1. NOVAS ENTIDADES: [Overfitting, Modelo]
2. NOVAS RELAÇÕES:
   - Projeto -> contém -> Modelo
   - Modelo -> apresenta_problema -> Overfitting
3. NOVAS OBSERVAÇÕES:
   - Modelo: status = com problemas de overfitting

## Benefícios

- **Contextualização**: Respostas mais relevantes e personalizadas
- **Continuidade**: Manutenção de contexto entre sessões
- **Evolução**: Conhecimento incremental sobre o usuário
- **Eficiência**: Evita repetição de informações básicas

## Implementação

Este prompt pode ser usado como instrução sistema em:
- Assistentes conversacionais
- Chatbots educacionais
- Sistemas de suporte técnico
- Interfaces de produtividade

---

*Documento em desenvolvimento. Contribuições e melhorias são bem-vindas.*