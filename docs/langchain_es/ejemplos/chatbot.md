# Ejemplo de Chatbot Simple

Este ejemplo demuestra cómo construir un chatbot conversacional con memoria utilizando LangChain.

## Implementación

```javascript
// Implementación en JavaScript/TypeScript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationChain } from "langchain/chains";
import { BufferMemory } from "langchain/memory";
import { ChatPromptTemplate, HumanMessagePromptTemplate, SystemMessagePromptTemplate, MessagesPlaceholder } from "langchain/prompts";

// Inicializar el modelo de chat
const chatModel = new ChatOpenAI({
  temperature: 0.7,
  modelName: "gpt-3.5-turbo",
});

// Crear una plantilla de prompt con un mensaje del sistema
const chatPrompt = ChatPromptTemplate.fromPromptMessages([
  SystemMessagePromptTemplate.fromTemplate(
    "Eres un asistente útil que recuerda conversaciones previas con el usuario."
  ),
  new MessagesPlaceholder("history"),
  HumanMessagePromptTemplate.fromTemplate("{input}"),
]);

// Configurar la memoria para almacenar el historial de conversación
const memory = new BufferMemory({
  returnMessages: true,
  memoryKey: "history",
});

// Crear la cadena de conversación
const chain = new ConversationChain({
  llm: chatModel,
  memory: memory,
  prompt: chatPrompt,
});

// Función para manejar mensajes del usuario
async function handleUserMessage(userInput) {
  try {
    const response = await chain.call({
      input: userInput,
    });
    return response.response;
  } catch (error) {
    console.error("Error:", error);
    return "Lo siento, encontré un error al procesar tu solicitud.";
  }
}

// Ejemplo de uso
async function runChatExample() {
  console.log("Bot: ¡Hola! ¿En qué puedo ayudarte hoy?");
  
  // Primer mensaje del usuario
  const response1 = await handleUserMessage("Mi nombre es Juan. Estoy interesado en aprender sobre inteligencia artificial.");
  console.log(`Bot: ${response1}`);
  
  // Segundo mensaje del usuario - el bot debería recordar el nombre del usuario
  const response2 = await handleUserMessage("¿Qué temas me recomiendas comenzar?");
  console.log(`Bot: ${response2}`);
  
  // Tercer mensaje del usuario - probando más la memoria
  const response3 = await handleUserMessage("¿Puedes recordarme cuál es mi nombre?");
  console.log(`Bot: ${response3}`);
}

runChatExample();
```

```python
# Implementación en Python
from langchain.chat_models import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
from langchain.prompts import (
    ChatPromptTemplate, 
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate, 
    MessagesPlaceholder
)

# Inicializar el modelo de chat
chat_model = ChatOpenAI(
    temperature=0.7,
    model_name="gpt-3.5-turbo"
)

# Crear una plantilla de prompt con un mensaje del sistema
chat_prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template(
        "Eres un asistente útil que recuerda conversaciones previas con el usuario."
    ),
    MessagesPlaceholder(variable_name="history"),
    HumanMessagePromptTemplate.from_template("{input}")
])

# Configurar la memoria para almacenar el historial de conversación
memory = ConversationBufferMemory(
    return_messages=True,
    memory_key="history"
)

# Crear la cadena de conversación
chain = ConversationChain(
    llm=chat_model,
    memory=memory,
    prompt=chat_prompt,
    verbose=True
)

# Función para manejar mensajes del usuario
def handle_user_message(user_input):
    try:
        response = chain.predict(input=user_input)
        return response
    except Exception as e:
        print(f"Error: {e}")
        return "Lo siento, encontré un error al procesar tu solicitud."

# Ejemplo de uso
def run_chat_example():
    print("Bot: ¡Hola! ¿En qué puedo ayudarte hoy?")
    
    # Primer mensaje del usuario
    response1 = handle_user_message("Mi nombre es Juan. Estoy interesado en aprender sobre inteligencia artificial.")
    print(f"Bot: {response1}")
    
    # Segundo mensaje del usuario - el bot debería recordar el nombre del usuario
    response2 = handle_user_message("¿Qué temas me recomiendas comenzar?")
    print(f"Bot: {response2}")
    
    # Tercer mensaje del usuario - probando más la memoria
    response3 = handle_user_message("¿Puedes recordarme cuál es mi nombre?")
    print(f"Bot: {response3}")

if __name__ == "__main__":
    run_chat_example()
```

## Cómo Funciona

1. **Modelo de Chat**: El ejemplo utiliza ChatOpenAI, que está específicamente diseñado para interacciones conversacionales.

2. **Plantilla de Prompt**: Creamos un prompt estructurado con:
   - Un mensaje del sistema que define el comportamiento del asistente
   - Un marcador de posición para el historial de conversación
   - Una plantilla para el mensaje humano actual

3. **Memoria**: BufferMemory almacena el historial completo de la conversación, permitiendo al chatbot referenciar intercambios previos.

4. **Cadena de Conversación**: Conecta el modelo, la memoria y el prompt en un único flujo de trabajo.

## Opciones de Personalización

- **Personalidad**: Modifica el mensaje del sistema para cambiar la personalidad del chatbot
- **Tipo de Memoria**: Cambia a otros tipos de memoria como SummaryMemory para conversaciones más largas
- **Modelo**: Usa diferentes modelos como GPT-4 o Claude para mejorar las capacidades
- **Temperatura**: Ajusta para respuestas más deterministas (menor) o creativas (mayor)

## Extensiones

- Añadir respuestas en streaming
- Implementar indicadores de escritura
- Integrar con plataformas de mensajería
- Añadir capacidades de uso de herramientas