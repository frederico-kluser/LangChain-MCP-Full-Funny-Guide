# Tools

Tools in LangChain are interfaces that allow language models to interact with external systems and perform specific functions. They extend the capabilities of LLMs beyond text generation.

## Understanding Tools

A tool typically has:

1. **Name**: A unique identifier
2. **Description**: Text that explains what the tool does and how to use it
3. **Function**: The actual implementation that performs the action
4. **Input schema**: The expected format of inputs
5. **Output processor**: Handles the results returned by the function

## Built-in Tools

LangChain provides many pre-built tools:

### Search and Retrieval
- Web search (Google, Bing, DuckDuckGo)
- Wikipedia access
- Document retrieval

### Calculation
- Calculator
- Math evaluators
- Wolfram Alpha integration

### Code Execution
- Python REPL
- Shell commands
- SQL queries

### APIs
- Weather information
- Stock data
- External API wrappers

## Creating Custom Tools

You can create custom tools to connect LLMs to any external system:

```javascript
// JavaScript example
import { Tool } from "langchain/tools";

const customTool = new Tool({
  name: "weather-lookup",
  description: "Get the current weather in a location",
  func: async ({ location }) => {
    // Implementation that calls a weather API
    const response = await fetch(`https://api.weather.com?location=${location}`);
    const data = await response.json();
    return data.temperature;
  },
});
```

```python
# Python example
from langchain.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Optional, Type

class WeatherInput(BaseModel):
    location: str = Field(description="The city and state, e.g. San Francisco, CA")

class WeatherTool(BaseTool):
    name = "weather-lookup"
    description = "Get the current weather in a location"
    args_schema: Type[BaseModel] = WeatherInput
    
    def _run(self, location: str) -> str:
        # Implementation that calls a weather API
        # Return the weather information
        return f"72°F and sunny in {location}"
        
    def _arun(self, location: str) -> str:
        # Async implementation
        return self._run(location)
```

## Using Tools with Agents

Tools become especially powerful when used with agents, which can decide which tools to use and when:

```python
# Python example
from langchain.agents import initialize_agent, AgentType
from langchain.tools import DuckDuckGoSearchRun, WikipediaQueryRun

llm = OpenAI(temperature=0)
search_tool = DuckDuckGoSearchRun()
wiki_tool = WikipediaQueryRun()

tools = [search_tool, wiki_tool]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

agent.run("What is the capital of France and what is its population?")
```

## Tool Best Practices

1. **Clear descriptions**: Provide detailed usage instructions in the description
2. **Input validation**: Validate inputs to prevent errors
3. **Error handling**: Gracefully handle failures and return informative messages
4. **Observability**: Add logging to track tool usage and performance
5. **Permissions**: Consider security implications of tools that modify systems