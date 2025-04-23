# Deployment de Aplicações LangChain

Esta seção aborda estratégias e melhores práticas para implantar aplicações LangChain em ambientes de produção.

## Considerações Iniciais

Antes de fazer deploy, considere:

1. **Requisitos de performance** - Latência, throughput, concorrência
2. **Custos** - Modelos, armazenamento, infraestrutura
3. **Segurança** - Filtragem de entrada/saída, autenticação, criptografia
4. **Monitoramento** - Observabilidade, logs, alertas
5. **Escalabilidade** - Carga variável, resiliência

## Arquiteturas de Deployment

### Serverless Functions

Ideal para cargas de trabalho intermitentes e aplicações simples:

```javascript
// Exemplo de função serverless (AWS Lambda)
exports.handler = async (event) => {
  // Inicialização sob demanda
  const { ChatOpenAI } = await import("langchain/chat_models/openai");
  const { ConversationChain } = await import("langchain/chains");
  
  // Processar a requisição
  const llm = new ChatOpenAI({
    modelName: "gpt-3.5-turbo",
    temperature: 0,
    cache: true,
  });
  
  const chain = new ConversationChain({ llm });
  const response = await chain.call({
    input: event.body.query,
  });
  
  return {
    statusCode: 200,
    body: JSON.stringify(response),
  };
};
```

**Prós**: Escalabilidade automática, sem gerenciamento de servidor, pay-per-use
**Contras**: Cold starts, tempos limite, limitações de memória

### API Server

Para aplicações mais complexas e com maior controle:

```javascript
// Exemplo Express.js
import express from "express";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationChain } from "langchain/chains";
import { Redis } from "langchain/cache";

// Configuração de cache e otimização
const redis = new Redis({
  url: process.env.REDIS_URL
});

// Inicialização ao startup
const llm = new ChatOpenAI({
  modelName: "gpt-3.5-turbo",
  temperature: 0,
  cache: redis,
});

const chain = new ConversationChain({ llm });

const app = express();
app.use(express.json());

app.post("/chat", async (req, res) => {
  try {
    const response = await chain.call({
      input: req.body.query,
    });
    res.json(response);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

**Prós**: Maior controle, tempos limite configuráveis, melhor desempenho
**Contras**: Exige gerenciamento de infraestrutura, escalabilidade manual

### Contêineres

Para portabilidade e isolamento:

```dockerfile
# Exemplo de Dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

# Configurações como variáveis de ambiente
ENV NODE_ENV production
ENV OPENAI_API_KEY your-api-key-reference

# Exposição da porta
EXPOSE 3000

CMD ["node", "server.js"]
```

**Prós**: Isolamento, portabilidade, controle de dependências
**Contras**: Overhead de contêiner, complexidade de orquestração

## Otimizações de Performance

### Caching

Implemente cache para reduzir chamadas de API e melhorar latência:

```javascript
import { Redis } from "langchain/cache";

// Configuração de cache
const cache = new Redis({
  url: process.env.REDIS_URL,
  ttl: 3600, // 1 hora
});

const llm = new ChatOpenAI({
  cache: cache,
  modelName: "gpt-3.5-turbo",
});
```

### Agendamento de Tarefas

Para operações de longa duração, utilize sistemas de filas:

```javascript
// Exemplo com Bull Queue
import Queue from "bull";

const processingQueue = new Queue("langchain-processing", {
  redis: process.env.REDIS_URL,
});

// Adicionar trabalho à fila
app.post("/process-document", async (req, res) => {
  const job = await processingQueue.add({
    documentId: req.body.documentId,
    userId: req.user.id,
  });
  
  res.json({ jobId: job.id });
});

// Processador de fila
processingQueue.process(async (job) => {
  const { documentId, userId } = job.data;
  
  // Executar operação LangChain de longa duração
  const document = await db.getDocument(documentId);
  const result = await ragChain.call({
    query: "Analise este documento",
    context: document.content,
  });
  
  // Salvar resultado
  await db.saveResult(documentId, result);
  
  return { status: "completed" };
});
```

## Monitoramento

### Integração com LangSmith

```javascript
// Configuração básica
process.env.LANGCHAIN_TRACING_V2 = "true";
process.env.LANGCHAIN_API_KEY = process.env.LANGSMITH_API_KEY;
process.env.LANGCHAIN_PROJECT = "production-app";

// OU configuração detalhada
import { LangChainTracer } from "langchain/callbacks";

const tracer = new LangChainTracer({
  projectName: "production-app",
  tags: ["prod", "v1.2"],
});

const chain = new ConversationChain({
  llm,
  callbacks: [tracer],
});
```

### Logging Customizado

```javascript
import { ConsoleCallbackHandler } from "langchain/callbacks";
import winston from "winston";

// Logger customizado
const logger = winston.createLogger({
  level: "info",
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.File({ filename: "combined.log" }),
  ],
});

// Callback handler personalizado
class LoggingHandler extends ConsoleCallbackHandler {
  handleLLMEnd(output) {
    logger.info("LLM completed", {
      output: output.generations[0][0].text,
      tokensUsed: output.llmOutput?.tokenUsage,
    });
  }
  
  handleChainError(err) {
    logger.error("Chain error", { error: err.message, stack: err.stack });
  }
}

// Uso
const chain = new ConversationChain({
  llm,
  callbacks: [new LoggingHandler()],
});
```

## Segurança

### Validação de Entrada

```javascript
import { z } from "zod";

// Esquema de validação
const inputSchema = z.object({
  query: z.string().min(1).max(500),
  context: z.string().optional(),
});

app.post("/chat", async (req, res) => {
  try {
    // Validação
    const validatedInput = inputSchema.parse(req.body);
    
    // Processamento
    const response = await chain.call({
      input: validatedInput.query,
    });
    
    res.json(response);
  } catch (error) {
    if (error instanceof z.ZodError) {
      res.status(400).json({ error: "Invalid input", details: error.errors });
    } else {
      res.status(500).json({ error: "Internal server error" });
    }
  }
});
```

### Filtragem de Conteúdo

```javascript
import { OpenAIModerationChain } from "langchain/chains";

// Criar chain de moderação
const moderationChain = new OpenAIModerationChain({
  throwError: true,
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Uso em cadeia
app.post("/chat", async (req, res) => {
  try {
    // Verificar entrada
    await moderationChain.call({
      input: req.body.query,
    });
    
    // Processar com LLM
    const response = await chain.call({
      input: req.body.query,
    });
    
    // Verificar saída
    await moderationChain.call({
      input: response.response,
    });
    
    res.json(response);
  } catch (error) {
    if (error.message.includes("Moderation")) {
      res.status(400).json({ error: "Content violates content policy" });
    } else {
      res.status(500).json({ error: "Internal server error" });
    }
  }
});
```

## Checklist para Produção

✅ **Performance**
- [ ] Implementação de cache
- [ ] Otimização de prompts
- [ ] Gerenciamento de concorrência
- [ ] Escolha adequada de modelos

✅ **Resiliência**
- [ ] Tratamento de erros
- [ ] Retentativas com backoff
- [ ] Fallbacks para serviços críticos
- [ ] Timeout adequado

✅ **Monitoramento**
- [ ] Logging abrangente
- [ ] Métricas de desempenho
- [ ] Alertas para falhas
- [ ] Rastreamento de custos

✅ **Segurança**
- [ ] Validação de entrada
- [ ] Filtragem de saída
- [ ] Autenticação e autorização
- [ ] Proteção de dados sensíveis

✅ **Escalabilidade**
- [ ] Estratégia para picos de demanda
- [ ] Distribuição geográfica (se necessário)
- [ ] Balanceamento de carga
- [ ] Limites de taxa configurados