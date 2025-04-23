# Simple Chatbot Example

This example demonstrates how to build a conversational chatbot with memory using LangChain.

## Implementation

```javascript
// JavaScript/TypeScript implementation
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationChain } from "langchain/chains";
import { BufferMemory } from "langchain/memory";
import { ChatPromptTemplate, HumanMessagePromptTemplate, SystemMessagePromptTemplate, MessagesPlaceholder } from "langchain/prompts";

// Initialize the chat model
const chatModel = new ChatOpenAI({
  temperature: 0.7,
  modelName: "gpt-3.5-turbo",
});

// Create a prompt template with a system message
const chatPrompt = ChatPromptTemplate.fromPromptMessages([
  SystemMessagePromptTemplate.fromTemplate(
    "You are a helpful assistant that remembers previous conversations with the user."
  ),
  new MessagesPlaceholder("history"),
  HumanMessagePromptTemplate.fromTemplate("{input}"),
]);

// Set up memory to store conversation history
const memory = new BufferMemory({
  returnMessages: true,
  memoryKey: "history",
});

// Create the conversation chain
const chain = new ConversationChain({
  llm: chatModel,
  memory: memory,
  prompt: chatPrompt,
});

// Function to handle user messages
async function handleUserMessage(userInput) {
  try {
    const response = await chain.call({
      input: userInput,
    });
    return response.response;
  } catch (error) {
    console.error("Error:", error);
    return "Sorry, I encountered an error processing your request.";
  }
}

// Example usage
async function runChatExample() {
  console.log("Bot: Hello! How can I help you today?");
  
  // First user message
  const response1 = await handleUserMessage("My name is John. I'm interested in learning about artificial intelligence.");
  console.log(`Bot: ${response1}`);
  
  // Second user message - the bot should remember the user's name
  const response2 = await handleUserMessage("What topics do you recommend I start with?");
  console.log(`Bot: ${response2}`);
  
  // Third user message - testing memory further
  const response3 = await handleUserMessage("Can you remind me what my name is?");
  console.log(`Bot: ${response3}`);
}

runChatExample();
```

```python
# Python implementation
from langchain.chat_models import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
from langchain.prompts import (
    ChatPromptTemplate, 
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate, 
    MessagesPlaceholder
)

# Initialize the chat model
chat_model = ChatOpenAI(
    temperature=0.7,
    model_name="gpt-3.5-turbo"
)

# Create a prompt template with a system message
chat_prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template(
        "You are a helpful assistant that remembers previous conversations with the user."
    ),
    MessagesPlaceholder(variable_name="history"),
    HumanMessagePromptTemplate.from_template("{input}")
])

# Set up memory to store conversation history
memory = ConversationBufferMemory(
    return_messages=True,
    memory_key="history"
)

# Create the conversation chain
chain = ConversationChain(
    llm=chat_model,
    memory=memory,
    prompt=chat_prompt,
    verbose=True
)

# Function to handle user messages
def handle_user_message(user_input):
    try:
        response = chain.predict(input=user_input)
        return response
    except Exception as e:
        print(f"Error: {e}")
        return "Sorry, I encountered an error processing your request."

# Example usage
def run_chat_example():
    print("Bot: Hello! How can I help you today?")
    
    # First user message
    response1 = handle_user_message("My name is John. I'm interested in learning about artificial intelligence.")
    print(f"Bot: {response1}")
    
    # Second user message - the bot should remember the user's name
    response2 = handle_user_message("What topics do you recommend I start with?")
    print(f"Bot: {response2}")
    
    # Third user message - testing memory further
    response3 = handle_user_message("Can you remind me what my name is?")
    print(f"Bot: {response3}")

if __name__ == "__main__":
    run_chat_example()
```

## How It Works

1. **Chat Model**: The example uses ChatOpenAI, which is specifically designed for conversational interactions.

2. **Prompt Template**: We create a structured prompt with:
   - A system message that defines the assistant's behavior
   - A placeholder for conversation history
   - A template for the current human message

3. **Memory**: BufferMemory stores the full conversation history, allowing the chatbot to reference previous exchanges.

4. **Conversation Chain**: Connects the model, memory, and prompt into a single workflow.

## Customization Options

- **Personality**: Modify the system message to change the chatbot's persona
- **Memory Type**: Switch to other memory types like SummaryMemory for longer conversations
- **Model**: Use different models like GPT-4 or Claude for improved capabilities
- **Temperature**: Adjust for more deterministic (lower) or creative (higher) responses

## Extensions

- Add streaming responses
- Implement typing indicators
- Integrate with messaging platforms
- Add tool usage capabilities