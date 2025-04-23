# Memoria

Los componentes de memoria en LangChain permiten a las aplicaciones retener y utilizar información de interacciones previas. Esto es esencial para crear experiencias conversacionales que se sientan naturales y contextuales.

## Tipos de Memoria

### Memoria de Conversación
Almacena el historial de interacciones entre el usuario y el sistema:

```javascript
// Ejemplo en JavaScript
import { ConversationChain } from "langchain/chains";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationMemory } from "langchain/memory";

const llm = new ChatOpenAI();
const memory = new ConversationMemory();

const chain = new ConversationChain({ llm, memory });

// Primera interacción
const response1 = await chain.call({ input: "Hola, mi nombre es Juan." });
console.log(response1); // "Hola Juan, encantado de conocerte."

// Segunda interacción - el modelo recuerda el nombre
const response2 = await chain.call({ input: "¿Cómo te llamas?" });
console.log(response2); // "Tu nombre es Juan."
```

```python
# Ejemplo en Python
from langchain.chains import ConversationChain
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI()
memory = ConversationBufferMemory()

chain = ConversationChain(llm=llm, memory=memory)

# Primera interacción
response1 = chain.predict(input="Hola, mi nombre es Juan.")
print(response1) # "Hola Juan, encantado de conocerte."

# Segunda interacción - el modelo recuerda el nombre
response2 = chain.predict(input="¿Cómo me llamo?")
print(response2) # "Tu nombre es Juan."
```

### Memoria de Buffer
La forma más simple que almacena todas las interacciones previas:

- **ConversationBufferMemory**: Almacena todos los mensajes
- **ConversationBufferWindowMemory**: Almacena los últimos N mensajes

### Memoria de Resumen
Crea resúmenes de interacciones pasadas para ahorrar longitud de contexto:

- **ConversationSummaryMemory**: Resume periódicamente la conversación
- **ConversationSummaryBufferMemory**: Combina enfoques de resumen y buffer

### Memoria de Entidades
Realiza seguimiento de entidades específicas mencionadas en conversaciones:

```python
# Ejemplo en Python
from langchain.memory import ConversationEntityMemory
from langchain.llms import OpenAI

llm = OpenAI()
memory = ConversationEntityMemory(llm=llm)

memory.save_context(
    {"input": "Juan ama el tenis"},
    {"output": "Es genial saber sobre el interés de Juan."}
)

memory.save_context(
    {"input": "¿Dónde trabaja Sara?"},
    {"output": "No sé dónde trabaja Sara."}
)

print(memory.entity_store.get("Juan")) # Contiene información sobre Juan
print(memory.entity_store.get("Sara")) # Contiene información sobre Sara
```

## Opciones de Almacenamiento de Memoria

La memoria puede almacenarse en diferentes backends:

- **En memoria**: Por defecto, no persistente entre reinicios
- **Redis**: Para memoria distribuida y persistente
- **DynamoDB/MongoDB**: Para almacenamiento estructurado y escalable
- **SQLite/PostgreSQL**: Para almacenamiento en base de datos relacional

```python
# Ejemplo con almacenamiento Redis
from langchain.memory import ConversationBufferMemory
from langchain.memory.chat_message_histories import RedisChatMessageHistory

message_history = RedisChatMessageHistory(
    url="redis://localhost:6379/0", 
    ttl=600,  # Tiempo de vida en segundos
    session_id="user-123"
)

memory = ConversationBufferMemory(
    chat_memory=message_history
)
```

## Mejores Prácticas

1. **Elegir el tipo de memoria apropiado**: Considera la naturaleza de tu aplicación
2. **Gestionar la longitud del contexto**: Para conversaciones largas, usa ventanas o resúmenes
3. **Considerar la privacidad**: Ten cuidado al almacenar información sensible del usuario
4. **Estrategia de persistencia**: Implementa almacenamiento adecuado para aplicaciones en producción
5. **Limpiar la memoria cuando sea apropiado**: Algunas conversaciones deberían comenzar frescas