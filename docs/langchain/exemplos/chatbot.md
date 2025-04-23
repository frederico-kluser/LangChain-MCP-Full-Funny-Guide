# Exemplo: Chatbot Simples

Este exemplo demonstra como criar um chatbot simples com memória de conversa usando LangChain.

## Código (JavaScript)

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationChain } from "langchain/chains";
import { BufferMemory } from "langchain/memory";

// Inicialização do modelo de chat
const chatModel = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY,
  modelName: "gpt-3.5-turbo",
  temperature: 0.7,
});

// Configuração da memória do chatbot
const memory = new BufferMemory();

// Criar a cadeia de conversa
const conversationChain = new ConversationChain({
  llm: chatModel,
  memory: memory,
});

// Função para interagir com o chatbot
async function chatWithBot(userInput) {
  const response = await conversationChain.call({
    input: userInput,
  });
  
  console.log(`Usuário: ${userInput}`);
  console.log(`Chatbot: ${response.response}`);
  return response.response;
}

// Exemplo de conversa
async function runConversation() {
  await chatWithBot("Olá! Meu nome é Ana.");
  await chatWithBot("Qual é o meu nome?");
  await chatWithBot("Você pode me explicar o que é aprendizado de máquina?");
}

runConversation();
```

## Explicado passo a passo

1. **Importação dos módulos necessários**:
   - `ChatOpenAI`: Wrapper para o modelo de chat da OpenAI
   - `ConversationChain`: Chain que facilita a conversação
   - `BufferMemory`: Sistema de memória que armazena interações anteriores

2. **Inicialização do modelo**:
   - Configuramos o modelo de chat com a API key
   - Definimos o modelo específico e temperatura (controla aleatoriedade)

3. **Configuração da memória**:
   - `BufferMemory` armazena o histórico de conversa

4. **Criação da chain de conversa**:
   - Combinamos o modelo e a memória

5. **Função para interação**:
   - Envia entrada do usuário para a chain
   - Recebe e exibe a resposta

6. **Exemplo de conversa**:
   - Demonstra perguntas sequenciais onde o chatbot lembra informações anteriores

## Resultado esperado

```
Usuário: Olá! Meu nome é Ana.
Chatbot: Olá, Ana! É um prazer conhecer você. Como posso ajudar você hoje?

Usuário: Qual é o meu nome?
Chatbot: Seu nome é Ana, você mencionou isso em sua mensagem anterior.

Usuário: Você pode me explicar o que é aprendizado de máquina?
Chatbot: Claro, Ana! Aprendizado de máquina é um ramo da inteligência artificial que se concentra no desenvolvimento de algoritmos e modelos que permitem que computadores aprendam a partir de dados sem serem explicitamente programados para cada tarefa específica.
```

## Customizações possíveis

1. **Mudar o tipo de memória**:
   ```javascript
   // Memória com resumo da conversa - útil para conversas longas
   import { ConversationSummaryMemory } from "langchain/memory";
   const memory = new ConversationSummaryMemory({
     llm: chatModel,
     maxTokenLimit: 100
   });
   ```

2. **Adicionar prompts personalizados**:
   ```javascript
   import { ChatPromptTemplate } from "langchain/prompts";
   
   const chatPrompt = ChatPromptTemplate.fromMessages([
     ["system", "Você é um assistente amigável que sempre responde em tom informal. Você se chama TechBot."],
     ["human", "{input}"],
   ]);
   ```

3. **Mudar o modelo**:
   ```javascript
   // Usar modelo da Anthropic
   import { ChatAnthropic } from "langchain/chat_models/anthropic";
   
   const chatModel = new ChatAnthropic({
     anthropicApiKey: process.env.ANTHROPIC_API_KEY,
     modelName: "claude-2",
   });
   ```