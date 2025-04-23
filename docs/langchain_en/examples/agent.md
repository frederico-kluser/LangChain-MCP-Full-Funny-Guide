# Agent with Tools Example

This example demonstrates how to create an agent that can use multiple tools to complete complex tasks.

## Implementation

```javascript
// JavaScript/TypeScript implementation
import { ChatOpenAI } from "langchain/chat_models/openai";
import { Calculator } from "langchain/tools/calculator";
import { SerpAPI } from "langchain/tools";
import { WebBrowser } from "langchain/tools/webbrowser";
import { initializeAgentExecutorWithOptions } from "langchain/agents";

async function createToolAgent() {
  // 1. Initialize the language model
  const model = new ChatOpenAI({
    temperature: 0,
    modelName: "gpt-4", // Using GPT-4 for better reasoning
  });
  
  // 2. Define the tools the agent can use
  const tools = [
    new Calculator(), // For mathematical calculations
    new SerpAPI(process.env.SERPAPI_API_KEY, {
      location: "Austin,Texas,United States",
      hl: "en",
      gl: "us",
    }), // For web search
    new WebBrowser({ model, embeddings: new OpenAIEmbeddings() }), // For browsing websites
  ];
  
  // 3. Create the agent
  const executor = await initializeAgentExecutorWithOptions(tools, model, {
    agentType: "chat-conversational-react-description",
    verbose: true,
    maxIterations: 5, // Prevent infinite loops
  });
  
  return executor;
}

// Function to handle user requests
async function runAgentTask(agent, task) {
  try {
    const result = await agent.call({ input: task });
    return result.output;
  } catch (error) {
    console.error("Error:", error);
    return "An error occurred while processing your request.";
  }
}

// Example usage
async function runAgentExample() {
  console.log("Creating agent with tools...");
  const agent = await createToolAgent();
  
  const tasks = [
    "What is the current price of Apple stock?",
    "Calculate the square root of 1225",
    "What's the weather like in New York today? And what's 15% of 85?",
  ];
  
  for (const task of tasks) {
    console.log(`\nTask: ${task}`);
    const result = await runAgentTask(agent, task);
    console.log(`Result: ${result}`);
  }
}

runAgentExample();
```

```python
# Python implementation
from langchain.chat_models import ChatOpenAI
from langchain.agents import initialize_agent, AgentType
from langchain.tools import Calculator, DuckDuckGoSearchRun
from langchain.agents.agent_toolkits import create_python_agent
from langchain.tools.python.tool import PythonREPLTool
from langchain.embeddings import OpenAIEmbeddings
from langchain.tools.playwright.utils import create_sync_playwright_browser

def create_tool_agent():
    # 1. Initialize the language model
    model = ChatOpenAI(
        temperature=0,
        model_name="gpt-4"  # Using GPT-4 for better reasoning
    )
    
    # 2. Define the tools the agent can use
    tools = [
        Calculator(),  # For mathematical calculations
        DuckDuckGoSearchRun(),  # For web search
    ]
    
    # Create a Python REPL agent for code execution
    # Note: This should be used with caution
    python_agent = create_python_agent(
        llm=model,
        tool=PythonREPLTool(),
        verbose=True
    )
    
    # 3. Create the main agent
    agent = initialize_agent(
        tools=tools,
        llm=model,
        agent=AgentType.CHAT_CONVERSATIONAL_REACT_DESCRIPTION,
        verbose=True,
        max_iterations=5,  # Prevent infinite loops
        handle_parsing_errors=True
    )
    
    return agent

# Function to handle user requests
def run_agent_task(agent, task):
    try:
        result = agent({"input": task})
        return result["output"]
    except Exception as e:
        print(f"Error: {e}")
        return "An error occurred while processing your request."

# Example usage
def run_agent_example():
    print("Creating agent with tools...")
    agent = create_tool_agent()
    
    tasks = [
        "What is the current price of Apple stock?",
        "Calculate the square root of 1225",
        "What's the weather like in New York today? And what's 15% of 85?",
    ]
    
    for task in tasks:
        print(f"\nTask: {task}")
        result = run_agent_task(agent, task)
        print(f"Result: {result}")

if __name__ == "__main__":
    run_agent_example()
```

## How It Works

1. **Model Selection**: The example uses ChatOpenAI (GPT-4) for its advanced reasoning capabilities.

2. **Tool Definition**: The agent has access to several tools:
   - Calculator: For performing mathematical operations
   - Search (SerpAPI/DuckDuckGo): For retrieving real-time information
   - Web Browser (JavaScript example): For browsing and analyzing web content
   - Python REPL (Python example): For executing Python code

3. **Agent Creation**: The agent is initialized with:
   - The specified tools
   - The language model
   - Agent type configuration
   - Iteration limits for safety

4. **Execution Process**: When given a task, the agent:
   - Analyzes what needs to be done
   - Selects the appropriate tool(s)
   - Uses the tools to gather information
   - Synthesizes a final answer

## Agent Reasoning Workflow

The conversational agent follows this process:

1. **Task Analysis**: The agent parses the task to understand what's being asked
2. **Tool Selection**: The agent decides which tool is best for the specific part of the task
3. **Tool Execution**: The agent uses the selected tool with appropriate parameters
4. **Result Evaluation**: The agent reviews the tool's output
5. **Next Step Decision**: The agent decides if the task is complete or if more tools need to be used
6. **Response Generation**: The agent compiles all gathered information into a final response

## Customization Options

- **Tools**: Add custom tools specific to your application needs
- **Agent Type**: Change the agent type based on the use case (e.g., structured-chat-zero-shot-react-description for more structured outputs)
- **Memory**: Add memory to allow the agent to remember previous interactions
- **Prompt Engineering**: Customize the agent's system message for different personalities or specialized knowledge

## Safety Considerations

- **Rate Limiting**: Implement rate limiting for tools that access external APIs
- **Input Validation**: Validate user inputs before passing to the agent
- **Output Filtering**: Consider filtering sensitive information from outputs
- **Execution Limits**: Set appropriate max_iterations to prevent infinite loops