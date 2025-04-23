# LangSmith

LangSmith é uma plataforma de desenvolvimento projetada para depurar, testar, avaliar e monitorar aplicações baseadas em LLMs.

## O que é LangSmith?

LangSmith funciona como uma plataforma de observabilidade e desenvolvimento para aplicações LangChain. Ele oferece:

- **Rastreamento e registro** - Visualize e depure todas as execuções de suas aplicações
- **Avaliação** - Compare diferentes modelos, prompts e cadeias
- **Anotações** - Marque e classifique resultados para avaliação humana
- **Datasets** - Crie conjuntos de dados para testes e avaliações
- **Monitoramento** - Acompanhe o desempenho em produção

## Como Usar o LangSmith

### 1. Configuração

```javascript
// Configuração com variáveis de ambiente
process.env.LANGCHAIN_TRACING_V2 = "true";
process.env.LANGCHAIN_API_KEY = "seu-api-key";
process.env.LANGCHAIN_PROJECT = "meu-projeto";

// OU configuração programática
import { Client } from "langsmith";

const client = new Client({
  apiKey: "seu-api-key",
  projectName: "meu-projeto",
});
```

### 2. Rastreamento de Execuções

Com o LangSmith configurado, todas as execuções de LangChain serão automaticamente rastreadas:

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationChain } from "langchain/chains";

const llm = new ChatOpenAI();
const chain = new ConversationChain({ llm });

// Esta execução será rastreada automaticamente no LangSmith
const result = await chain.call({
  input: "Olá, LangChain!",
});
```

O rastreamento inclui:
- Todas as entradas e saídas
- Tempos de execução
- Tokens usados
- Estrutura hierárquica da aplicação

### 3. Criação de Datasets para Testes

Você pode criar conjuntos de dados para avaliar sistematicamente sua aplicação:

```javascript
import { Dataset, RunEvaluator } from "langsmith";

// Criar ou carregar um dataset
const dataset = await client.createDataset("perguntas-teste");

// Adicionar exemplos
await client.createExample(
  { input: "Qual a capital do Brasil?" },
  { output: "Brasília" },
  { datasetId: dataset.id }
);

// Executar testes contra o dataset
const results = await client.runTests({
  datasetId: dataset.id,
  llmOrChain: chain,
});
```

### 4. Avaliação Automática

Configure avaliadores automatizados para medir a qualidade das respostas:

```javascript
import { StringEvaluator } from "langsmith/evaluators";

// Criar um avaliador de exatidão
const evaluator = new StringEvaluator({
  name: "exatidao_resposta",
});

// Avaliar as execuções com base no dataset
const evaluations = await client.evaluateRuns({
  evaluators: [evaluator],
  datasetId: dataset.id,
  deploymentId: deployment.id,
});
```

### 5. Monitoramento em Produção

Configure alertas e monitore métricas em produção:

```javascript
import { LangChainTracer } from "langchain/callbacks";

// Configurar tracer específico para produção
const tracer = new LangChainTracer({
  projectName: "aplicacao-producao",
  client,
});

// Adicionar callbacks para monitoramento
const chain = new ConversationChain({
  llm,
  callbacks: [tracer],
});
```

## Interface Web

O LangSmith fornece uma interface web para:

1. **Visualizar rastreamentos** - Explore a árvore de execução completa
2. **Analisar erros** - Identifique e corrija problemas rapidamente
3. **Comparar modelos** - Testes A/B entre diferentes configurações
4. **Anotar resultados** - Classificação e feedback manual
5. **Criar dashboards** - Acompanhe métricas importantes

## Benefícios para Desenvolvimento

- **Ciclos de desenvolvimento mais rápidos** - Encontre e corrija erros rapidamente
- **Melhoria iterativa** - Teste mudanças em prompts e configurações
- **Confiança para produção** - Validação robusta antes de deploy
- **Observabilidade** - Acompanhe desempenho e comportamento em produção

## Limitações

- Requer uma API key e conta no LangSmith
- Pode afetar ligeiramente a performance em aplicações de alta escala
- Algumas funcionalidades avançadas são pagas

## Quando Usar LangSmith

LangSmith é essencial quando:

- Você está construindo aplicações LangChain complexas
- Precisa de observabilidade e debugging avançado
- Tem necessidade de avaliação sistemática de modelos e prompts
- Quer monitorar aplicações em produção
- Trabalha em equipe e precisa compartilhar resultados