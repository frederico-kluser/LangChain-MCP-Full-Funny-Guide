# Exemplo: Agente com Ferramentas

Este exemplo demonstra como criar um agente que pode usar ferramentas externas para resolver problemas.

## Código (JavaScript)

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { initializeAgentExecutorWithOptions } from "langchain/agents";
import { Calculator } from "langchain/tools/calculator";
import { WebBrowser } from "langchain/tools/webbrowser";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";

// Configuração do modelo
const model = new ChatOpenAI({
  temperature: 0,
  modelName: "gpt-3.5-turbo",
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Configuração dos embeddings
const embeddings = new OpenAIEmbeddings({
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Configuração das ferramentas
const calculator = new Calculator();
const browser = new WebBrowser({ model, embeddings });
const tools = [calculator, browser];

// Inicialização do agente
const executor = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: "chat-conversational-react-description",
  verbose: true,
  maxIterations: 5,
});

// Função para interagir com o agente
async function perguntarAgente(pergunta) {
  console.log(`\nPergunta: ${pergunta}`);
  console.log("Pensando...");
  
  const resultado = await executor.call({ input: pergunta });
  
  console.log(`Resposta: ${resultado.output}`);
  return resultado.output;
}

// Executar exemplos de consultas
async function executarExemplos() {
  await perguntarAgente("Quanto é 354 * 236?");
  await perguntarAgente("O que está acontecendo no Brasil hoje? Faça um resumo das principais notícias.");
  await perguntarAgente("Se eu tenho 500 reais e quero investir em ações, quantas ações da empresa X posso comprar se cada ação custa 25 reais?");
}

executarExemplos();
```

## Explicado passo a passo

1. **Importação dos módulos necessários**:
   - Componentes para o modelo de chat, agente, e ferramentas

2. **Configuração do modelo e embeddings**:
   - Inicialização do LLM que será usado como cérebro do agente
   - Configuração dos embeddings para a ferramenta de navegação

3. **Configuração das ferramentas**:
   - `Calculator`: Ferramenta para realizar cálculos matemáticos
   - `WebBrowser`: Ferramenta para buscar e extrair informações da web

4. **Inicialização do agente**:
   - Usando o tipo `chat-conversational-react-description`
   - Configuração para exibir passos intermediários e limitar iterações

5. **Função para interação**:
   - Interface simplificada para fazer perguntas ao agente
   - Exibe pergunta, pensamento e resposta final

6. **Exemplos de consultas**:
   - Demonstração de como o agente utiliza diferentes ferramentas

## Como funciona um Agente LangChain

1. **Análise da pergunta**: O agente recebe a pergunta e decide qual abordagem seguir

2. **Planejamento de ações**: O LLM determina quais ferramentas usar e em que ordem

3. **Execução e reflexão**: O agente executa ações, analisa resultados intermediários e decide próximos passos

4. **Síntese da resposta**: Após obter todas as informações necessárias, o agente formula a resposta final

## Exemplo de saída

```
Pergunta: Quanto é 354 * 236?
Pensando...
Vou usar a ferramenta de calculadora para resolver esta multiplicação.
Action: Calculator
Action Input: 354 * 236
Observation: 83544
Pensando... Ok, agora que tenho o resultado, posso responder.
Resposta: 354 multiplicado por 236 é igual a 83.544.

Pergunta: O que está acontecendo no Brasil hoje? Faça um resumo das principais notícias.
Pensando...
Preciso buscar informações atuais sobre notícias no Brasil. Vou usar o navegador para isso.
Action: WebBrowser
Action Input: https://g1.globo.com/
Observation: [Conteúdo do site com várias notícias...]
Pensando... Baseado nas informações que encontrei, vou resumir as principais notícias.
Resposta: As principais notícias do Brasil hoje incluem: [resumo das notícias encontradas]

Pergunta: Se eu tenho 500 reais e quero investir em ações, quantas ações da empresa X posso comprar se cada ação custa 25 reais?
Pensando...
Este é um problema matemático simples de divisão. Vou calcular quantas ações inteiras podem ser compradas.
Action: Calculator
Action Input: 500 / 25
Observation: 20
Pensando... Agora posso responder à pergunta.
Resposta: Com 500 reais, você pode comprar 20 ações da empresa X, considerando que cada ação custa 25 reais.
```

## Customizações possíveis

1. **Ferramentas personalizadas**:
   ```javascript
   // Criando uma ferramenta customizada
   import { Tool } from "langchain/tools";
   
   class BancoDadosTool extends Tool {
     name = "banco_dados";
     description = "Útil para consultar informações no banco de dados da empresa.";
     
     async _call(query) {
       // Implementação para conectar ao banco e executar consulta
       // Este é apenas um exemplo simplificado
       const resultado = await db.executar(query);
       return JSON.stringify(resultado);
     }
   }
   
   // Adicionar ao conjunto de ferramentas
   const bancoDados = new BancoDadosTool();
   const tools = [calculator, browser, bancoDados];
   ```

2. **Memória para conversação contínua**:
   ```javascript
   import { ConversationSummaryMemory } from "langchain/memory";
   
   // Criar memória
   const memory = new ConversationSummaryMemory({
     memoryKey: "chat_history",
     llm: model,
   });
   
   // Adicionar à inicialização do agente
   const executor = await initializeAgentExecutorWithOptions(tools, model, {
     agentType: "chat-conversational-react-description",
     memory,
     verbose: true,
   });
   ```

3. **Configurações avançadas de agente**:
   ```javascript
   // Uso de um agente customizado com prompt específico
   import { ChatPromptTemplate } from "langchain/prompts";
   import { ZeroShotAgent } from "langchain/agents";
   
   // Prompt personalizado
   const prompt = ChatPromptTemplate.fromMessages([
     ["system", "Você é um assistente especialista em finanças pessoais que ajuda a responder perguntas sobre investimentos..."],
     ["human", "{input}"],
   ]);
   
   // Criação do agente personalizado
   const agent = ZeroShotAgent.fromLLMAndTools(model, tools, {
     prefix: "Responda as perguntas sobre finanças pessoais.",
     suffix: "Comece!",
   });
   ```