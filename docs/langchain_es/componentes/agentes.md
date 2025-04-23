# Agentes

Los agentes en LangChain son sistemas que utilizan LLMs para determinar qué acciones tomar y en qué orden. Pueden usar herramientas, mantener estado y tomar decisiones basadas en resultados previos.

## Cómo Funcionan los Agentes

A alto nivel, los agentes:

1. Reciben una entrada o tarea del usuario
2. Analizan la tarea y determinan la siguiente acción
3. Ejecutan la acción usando herramientas disponibles
4. Analizan el resultado y deciden sobre acciones adicionales
5. Continúan hasta que la tarea esté completa
6. Devuelven el resultado final

## Tipos de Agentes en LangChain

### ReAct (Reasoning and Acting)
Utiliza un formato estructurado de Pensamiento, Acción, Observación:

```python
# Ejemplo en Python
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

agent.run("¿Cuál es la capital de Francia y cuál es su población?")
```

La salida mostrará el razonamiento:
```
Pensamiento: Necesito encontrar la capital de Francia y su población.
Acción: DuckDuckGoSearchRun
Entrada de Acción: "capital de Francia"
Observación: La capital de Francia es París.
Pensamiento: Ahora necesito encontrar la población de París.
...
```

### Agentes de Llamada a Funciones
Utilizan definiciones de funciones estructuradas para guiar al LLM:

```python
# Ejemplo en Python
from langchain.agents import AgentType, initialize_agent, Tool
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

tools = [
    Tool(
        name="Clima",
        func=lambda location: f"El clima en {location} es soleado, 24 grados.",
        description="Útil para verificar el clima en una ubicación específica"
    ),
    Tool(
        name="Población",
        func=lambda country: f"La población de {country} es de 10 millones.",
        description="Útil para obtener estadísticas de población de un país"
    )
]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True
)

agent.run("¿Cómo está el clima en París y cuál es la población de Francia?")
```

### Agentes de Plan y Ejecución
Primero crean un plan de pasos, luego ejecutan cada paso:

```python
# Ejemplo simplificado en Python
from langchain.agents import create_plan_and_execute_agent

agent = create_plan_and_execute_agent(
    llm=ChatOpenAI(temperature=0),
    tools=tools,
    verbose=True
)

agent.run("Encuentra el PIB del país con mayor población en Europa")
```

## Creación de Agentes Personalizados

Puedes crear agentes personalizados mediante:

1. Definir una plantilla de prompt personalizada con instrucciones
2. Especificar un analizador de salida para respuestas del agente
3. Crear herramientas para que el agente las use
4. Construir el agente con estos componentes

```python
# Ejemplo simplificado en Python
from langchain.agents import AgentExecutor, BaseAgent
from langchain.prompts import PromptTemplate

class MyCustomAgent(BaseAgent):
    @property
    def input_keys(self):
        return ["input"]
    
    def plan(self, intermediate_steps, **kwargs):
        # Lógica para determinar la siguiente acción
        return AgentAction(tool="some_tool", tool_input="input", log="razonamiento")
    
    async def aplan(self, intermediate_steps, **kwargs):
        # Versión asíncrona de plan
        return await self.plan(intermediate_steps, **kwargs)

# Crear una plantilla de prompt
template = """
Eres un agente especializado para la tarea X.
...
"""

# Crear el ejecutor del agente
agent_executor = AgentExecutor.from_agent_and_tools(
    agent=MyCustomAgent(),
    tools=tools,
    verbose=True
)
```

## Memoria y Estado del Agente

Los agentes pueden mantener estado entre acciones:

```python
# Ejemplo en Python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(memory_key="chat_history")

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory,
    verbose=True
)

agent.run("Mi nombre es Alice")
agent.run("¿Cómo me llamo?")  # El agente recuerda "Alice"
```

## Mejores Prácticas

1. **Instrucciones claras**: Da a los agentes pautas específicas sobre cuándo usar herramientas
2. **Diseño de herramientas**: Crea herramientas con descripciones claras y entradas/salidas bien definidas
3. **Manejo de errores**: Implementa mecanismos de reintento para acciones fallidas
4. **Observación**: Usa modo detallado durante el desarrollo para entender el razonamiento del agente
5. **Restricciones**: Establece límites razonables en iteraciones para prevenir bucles infinitos
6. **Balance entre autonomía y control**: Considera diseños con humano en el bucle para aplicaciones críticas