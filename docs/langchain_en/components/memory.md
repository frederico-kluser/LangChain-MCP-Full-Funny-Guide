# Memory

Memory components in LangChain allow applications to retain and utilize information from previous interactions. This is essential for creating conversational experiences that feel natural and contextual.

## Types of Memory

### Conversation Memory
Stores the history of interactions between the user and the system:

```javascript
// JavaScript example
import { ConversationChain } from "langchain/chains";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationMemory } from "langchain/memory";

const llm = new ChatOpenAI();
const memory = new ConversationMemory();

const chain = new ConversationChain({ llm, memory });

// First interaction
const response1 = await chain.call({ input: "Hi, my name is John." });
console.log(response1); // "Hello John, nice to meet you."

// Second interaction - the model remembers the name
const response2 = await chain.call({ input: "What's my name?" });
console.log(response2); // "Your name is John."
```

```python
# Python example
from langchain.chains import ConversationChain
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI()
memory = ConversationBufferMemory()

chain = ConversationChain(llm=llm, memory=memory)

# First interaction
response1 = chain.predict(input="Hi, my name is John.")
print(response1) # "Hello John, nice to meet you."

# Second interaction - the model remembers the name
response2 = chain.predict(input="What's my name?")
print(response2) # "Your name is John."
```

### Buffer Memory
The simplest form that stores all previous interactions:

- **ConversationBufferMemory**: Stores all messages
- **ConversationBufferWindowMemory**: Stores the last N messages

### Summary Memory
Creates summaries of past interactions to save context length:

- **ConversationSummaryMemory**: Periodically summarizes the conversation
- **ConversationSummaryBufferMemory**: Combines summary and buffer approaches

### Entity Memory
Tracks specific entities mentioned in conversations:

```python
# Python example
from langchain.memory import ConversationEntityMemory
from langchain.llms import OpenAI

llm = OpenAI()
memory = ConversationEntityMemory(llm=llm)

memory.save_context(
    {"input": "John loves tennis"},
    {"output": "That's great to hear about John's interest."}
)

memory.save_context(
    {"input": "Where does Sarah work?"},
    {"output": "I don't know where Sarah works."}
)

print(memory.entity_store.get("John")) # Contains information about John
print(memory.entity_store.get("Sarah")) # Contains information about Sarah
```

## Memory Storage Options

Memory can be stored in different backends:

- **In-memory**: Default, not persistent between restarts
- **Redis**: For distributed and persistent memory
- **DynamoDB/MongoDB**: For structured and scalable storage
- **SQLite/PostgreSQL**: For relational database storage

```python
# Example with Redis storage
from langchain.memory import ConversationBufferMemory
from langchain.memory.chat_message_histories import RedisChatMessageHistory

message_history = RedisChatMessageHistory(
    url="redis://localhost:6379/0", 
    ttl=600,  # Time to live in seconds
    session_id="user-123"
)

memory = ConversationBufferMemory(
    chat_memory=message_history
)
```

## Best Practices

1. **Choose appropriate memory type**: Consider the nature of your application
2. **Manage context length**: For long conversations, use windowing or summarization
3. **Consider privacy**: Be mindful of storing sensitive user information
4. **Persistence strategy**: Implement proper storage for production applications
5. **Clear memory when appropriate**: Some conversations should start fresh