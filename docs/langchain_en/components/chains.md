# Chains

Chains are sequences of operations that combine LLMs with other components to create more complex applications. They allow for multi-step workflows that can process, transform, and generate information.

## Basic Concept

A chain typically consists of:

1. **Input**: The starting data for the chain
2. **Processing steps**: A sequence of operations to transform the input
3. **Output**: The final result after all processing

## Common Chain Types in LangChain

### LLMChain
The most basic chain that combines a prompt template with an LLM:

```javascript
// JavaScript example
import { OpenAI } from "langchain/llms/openai";
import { PromptTemplate } from "langchain/prompts";
import { LLMChain } from "langchain/chains";

const llm = new OpenAI({ temperature: 0.7 });
const template = "What is the capital of {country}?";
const prompt = PromptTemplate.fromTemplate(template);

const chain = new LLMChain({ llm, prompt });
const result = await chain.run({ country: "France" });
console.log(result); // Paris
```

```python
# Python example
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

llm = OpenAI(temperature=0.7)
template = "What is the capital of {country}?"
prompt = PromptTemplate(template=template, input_variables=["country"])

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run(country="France")
print(result) # Paris
```

### SequentialChain
Combines multiple chains where the output of one chain becomes the input for the next:

```python
# Python example
from langchain.chains import SimpleSequentialChain

# Chain 1: Generate a topic
template1 = "Generate a random historical topic"
prompt1 = PromptTemplate(template=template1, input_variables=[])
chain1 = LLMChain(llm=llm, prompt=prompt1)

# Chain 2: Write a summary about that topic
template2 = "Write a short summary about: {topic}"
prompt2 = PromptTemplate(template=template2, input_variables=["topic"])
chain2 = LLMChain(llm=llm, prompt=prompt2)

# Combine chains
overall_chain = SimpleSequentialChain(chains=[chain1, chain2])
result = overall_chain.run()
```

### RouterChain
Dynamically selects which chain to use based on the input:

```python
# Python simplified example
from langchain.chains.router import RouterChain

# Define multiple chains for different tasks
summary_chain = LLMChain(llm=llm, prompt=summary_prompt)
translation_chain = LLMChain(llm=llm, prompt=translation_prompt)
question_chain = LLMChain(llm=llm, prompt=question_prompt)

# Router determines which chain to use
router_chain = RouterChain(
    chains={"summary": summary_chain, 
            "translation": translation_chain, 
            "question_answering": question_chain},
    llm=llm
)

result = router_chain.run(user_input)
```

## Building Custom Chains

LangChain allows developers to create custom chains by:

1. Creating a new class that extends `BaseChain`
2. Implementing the required methods like `_call` and `run`
3. Defining input/output variables

## Advanced Chain Features

- **Memory integration**: Chains can incorporate memory to maintain context
- **Tool usage**: Chains can use tools to interact with external systems
- **Callbacks**: Monitor chain execution and performance
- **Streaming**: Process outputs incrementally as they're generated