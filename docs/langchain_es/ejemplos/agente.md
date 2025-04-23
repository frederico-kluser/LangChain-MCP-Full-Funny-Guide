# Ejemplo de Agente con Herramientas

Este ejemplo demuestra cómo crear un agente que puede usar múltiples herramientas para completar tareas complejas.

## Implementación

```javascript
// Implementación en JavaScript/TypeScript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { Calculator } from "langchain/tools/calculator";
import { SerpAPI } from "langchain/tools";
import { WebBrowser } from "langchain/tools/webbrowser";
import { initializeAgentExecutorWithOptions } from "langchain/agents";

async function createToolAgent() {
  // 1. Inicializar el modelo de lenguaje
  const model = new ChatOpenAI({
    temperature: 0,
    modelName: "gpt-4", // Usando GPT-4 para mejor razonamiento
  });
  
  // 2. Definir las herramientas que el agente puede usar
  const tools = [
    new Calculator(), // Para cálculos matemáticos
    new SerpAPI(process.env.SERPAPI_API_KEY, {
      location: "Ciudad de México,México",
      hl: "es",
      gl: "mx",
    }), // Para búsqueda web
    new WebBrowser({ model, embeddings: new OpenAIEmbeddings() }), // Para navegar sitios web
  ];
  
  // 3. Crear el agente
  const executor = await initializeAgentExecutorWithOptions(tools, model, {
    agentType: "chat-conversational-react-description",
    verbose: true,
    maxIterations: 5, // Prevenir bucles infinitos
  });
  
  return executor;
}

// Función para manejar solicitudes del usuario
async function runAgentTask(agent, task) {
  try {
    const result = await agent.call({ input: task });
    return result.output;
  } catch (error) {
    console.error("Error:", error);
    return "Ocurrió un error al procesar tu solicitud.";
  }
}

// Ejemplo de uso
async function runAgentExample() {
  console.log("Creando agente con herramientas...");
  const agent = await createToolAgent();
  
  const tasks = [
    "¿Cuál es el precio actual de las acciones de Apple?",
    "Calcula la raíz cuadrada de 1225",
    "¿Cómo está el clima en Nueva York hoy? Y ¿cuánto es el 15% de 85?",
  ];
  
  for (const task of tasks) {
    console.log(`\nTarea: ${task}`);
    const result = await runAgentTask(agent, task);
    console.log(`Resultado: ${result}`);
  }
}

runAgentExample();
```

```python
# Implementación en Python
from langchain.chat_models import ChatOpenAI
from langchain.agents import initialize_agent, AgentType
from langchain.tools import Calculator, DuckDuckGoSearchRun
from langchain.agents.agent_toolkits import create_python_agent
from langchain.tools.python.tool import PythonREPLTool
from langchain.embeddings import OpenAIEmbeddings
from langchain.tools.playwright.utils import create_sync_playwright_browser

def create_tool_agent():
    # 1. Inicializar el modelo de lenguaje
    model = ChatOpenAI(
        temperature=0,
        model_name="gpt-4"  # Usando GPT-4 para mejor razonamiento
    )
    
    # 2. Definir las herramientas que el agente puede usar
    tools = [
        Calculator(),  # Para cálculos matemáticos
        DuckDuckGoSearchRun(),  # Para búsqueda web
    ]
    
    # Crear un agente REPL de Python para ejecución de código
    # Nota: Esto debe usarse con precaución
    python_agent = create_python_agent(
        llm=model,
        tool=PythonREPLTool(),
        verbose=True
    )
    
    # 3. Crear el agente principal
    agent = initialize_agent(
        tools=tools,
        llm=model,
        agent=AgentType.CHAT_CONVERSATIONAL_REACT_DESCRIPTION,
        verbose=True,
        max_iterations=5,  # Prevenir bucles infinitos
        handle_parsing_errors=True
    )
    
    return agent

# Función para manejar solicitudes del usuario
def run_agent_task(agent, task):
    try:
        result = agent({"input": task})
        return result["output"]
    except Exception as e:
        print(f"Error: {e}")
        return "Ocurrió un error al procesar tu solicitud."

# Ejemplo de uso
def run_agent_example():
    print("Creando agente con herramientas...")
    agent = create_tool_agent()
    
    tasks = [
        "¿Cuál es el precio actual de las acciones de Apple?",
        "Calcula la raíz cuadrada de 1225",
        "¿Cómo está el clima en Nueva York hoy? Y ¿cuánto es el 15% de 85?",
    ]
    
    for task in tasks:
        print(f"\nTarea: {task}")
        result = run_agent_task(agent, task)
        print(f"Resultado: {result}")

if __name__ == "__main__":
    run_agent_example()
```

## Cómo Funciona

1. **Selección del Modelo**: El ejemplo usa ChatOpenAI (GPT-4) por sus capacidades avanzadas de razonamiento.

2. **Definición de Herramientas**: El agente tiene acceso a varias herramientas:
   - Calculadora: Para realizar operaciones matemáticas
   - Búsqueda (SerpAPI/DuckDuckGo): Para recuperar información en tiempo real
   - Navegador Web (ejemplo de JavaScript): Para navegar y analizar contenido web
   - REPL de Python (ejemplo de Python): Para ejecutar código Python

3. **Creación del Agente**: El agente se inicializa con:
   - Las herramientas especificadas
   - El modelo de lenguaje
   - Configuración del tipo de agente
   - Límites de iteración por seguridad

4. **Proceso de Ejecución**: Cuando se le da una tarea, el agente:
   - Analiza lo que se necesita hacer
   - Selecciona la(s) herramienta(s) apropiada(s)
   - Usa las herramientas para recopilar información
   - Sintetiza una respuesta final

## Flujo de Trabajo de Razonamiento del Agente

El agente conversacional sigue este proceso:

1. **Análisis de Tarea**: El agente analiza la tarea para entender qué se está preguntando
2. **Selección de Herramienta**: El agente decide qué herramienta es mejor para la parte específica de la tarea
3. **Ejecución de Herramienta**: El agente usa la herramienta seleccionada con parámetros apropiados
4. **Evaluación de Resultado**: El agente revisa la salida de la herramienta
5. **Decisión de Siguiente Paso**: El agente decide si la tarea está completa o si se necesitan usar más herramientas
6. **Generación de Respuesta**: El agente compila toda la información recopilada en una respuesta final

## Opciones de Personalización

- **Herramientas**: Añade herramientas personalizadas específicas para las necesidades de tu aplicación
- **Tipo de Agente**: Cambia el tipo de agente basado en el caso de uso (p.ej., structured-chat-zero-shot-react-description para salidas más estructuradas)
- **Memoria**: Añade memoria para permitir al agente recordar interacciones previas
- **Ingeniería de Prompts**: Personaliza el mensaje del sistema del agente para diferentes personalidades o conocimiento especializado

## Consideraciones de Seguridad

- **Limitación de Tasa**: Implementa limitación de tasa para herramientas que acceden a APIs externas
- **Validación de Entrada**: Valida entradas del usuario antes de pasarlas al agente
- **Filtrado de Salida**: Considera filtrar información sensible de las salidas
- **Límites de Ejecución**: Establece max_iterations apropiados para prevenir bucles infinitos