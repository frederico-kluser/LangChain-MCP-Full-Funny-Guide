# Agents

Agents in LangChain are systems that use LLMs to determine which actions to take and in what order. They can use tools, maintain state, and make decisions based on previous results.

## How Agents Work

At a high level, agents:

1. Receive a user input or task
2. Analyze the task and determine the next action
3. Execute the action using available tools
4. Analyze the result and decide on further actions
5. Continue until the task is complete
6. Return the final result

## Agent Types in LangChain

### ReAct (Reasoning and Acting)
Uses a structured format of Thought, Action, Observation:

```python
# Python example
from langchain.agents import initialize_agent, AgentType
from langchain.llms import OpenAI
from langchain.tools import DuckDuckGoSearchRun

llm = OpenAI(temperature=0)
search_tool = DuckDuckGoSearchRun()

agent = initialize_agent(
    tools=[search_tool],
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

agent.run("What is the capital of France and what is its population?")
```

Output will show reasoning:
```
Thought: I need to find out the capital of France and its population.
Action: DuckDuckGoSearchRun
Action Input: "capital of France"
Observation: The capital of France is Paris.
Thought: Now I need to find the population of Paris.
...
```

### Function-calling Agents
Use structured function definitions to guide the LLM:

```python
# Python example
from langchain.agents import AgentType, initialize_agent, Tool
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

tools = [
    Tool(
        name="Weather",
        func=lambda location: f"The weather in {location} is sunny, 75 degrees.",
        description="Useful for checking weather in a specific location"
    ),
    Tool(
        name="Population",
        func=lambda country: f"The population of {country} is 10 million.",
        description="Useful for getting population statistics for a country"
    )
]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True
)

agent.run("What's the weather in Paris and the population of France?")
```

### Plan-and-Execute Agents
First create a plan of steps, then execute each step:

```python
# Python simplified example
from langchain.agents import create_plan_and_execute_agent

agent = create_plan_and_execute_agent(
    llm=ChatOpenAI(temperature=0),
    tools=tools,
    verbose=True
)

agent.run("Find the GDP of the country with the highest population in Europe")
```

## Creating Custom Agents

You can create custom agents by:

1. Defining a custom prompt template with instructions
2. Specifying an output parser for agent responses
3. Creating tools for the agent to use
4. Building the agent with these components

```python
# Python simplified example
from langchain.agents import AgentExecutor, BaseAgent
from langchain.prompts import PromptTemplate

class MyCustomAgent(BaseAgent):
    @property
    def input_keys(self):
        return ["input"]
    
    def plan(self, intermediate_steps, **kwargs):
        # Logic to determine next action
        return AgentAction(tool="some_tool", tool_input="input", log="reasoning")
    
    async def aplan(self, intermediate_steps, **kwargs):
        # Async version of plan
        return await self.plan(intermediate_steps, **kwargs)

# Create a prompt template
template = """
You are a specialized agent for task X.
...
"""

# Create the agent executor
agent_executor = AgentExecutor.from_agent_and_tools(
    agent=MyCustomAgent(),
    tools=tools,
    verbose=True
)
```

## Agent Memory and State

Agents can maintain state between actions:

```python
# Python example
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(memory_key="chat_history")

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory,
    verbose=True
)

agent.run("My name is Alice")
agent.run("What's my name?")  # Agent remembers "Alice"
```

## Best Practices

1. **Clear instructions**: Give agents specific guidelines on when to use tools
2. **Tool design**: Create tools with clear descriptions and well-defined inputs/outputs
3. **Error handling**: Implement retry mechanisms for failed actions
4. **Observation**: Use verbose mode during development to understand agent reasoning
5. **Constraints**: Set reasonable limits on iterations to prevent infinite loops
6. **Balance autonomy and control**: Consider human-in-the-loop designs for critical applications