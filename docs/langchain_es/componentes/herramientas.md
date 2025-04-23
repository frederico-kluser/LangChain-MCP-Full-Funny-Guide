# Herramientas

Las herramientas en LangChain son interfaces que permiten a los modelos de lenguaje interactuar con sistemas externos y realizar funciones específicas. Extienden las capacidades de los LLMs más allá de la generación de texto.

## Entendiendo las Herramientas

Una herramienta típicamente tiene:

1. **Nombre**: Un identificador único
2. **Descripción**: Texto que explica qué hace la herramienta y cómo usarla
3. **Función**: La implementación real que realiza la acción
4. **Esquema de entrada**: El formato esperado de las entradas
5. **Procesador de salida**: Maneja los resultados devueltos por la función

## Herramientas Incorporadas

LangChain proporciona muchas herramientas pre-construidas:

### Búsqueda y Recuperación
- Búsqueda web (Google, Bing, DuckDuckGo)
- Acceso a Wikipedia
- Recuperación de documentos

### Cálculo
- Calculadora
- Evaluadores matemáticos
- Integración con Wolfram Alpha

### Ejecución de Código
- REPL de Python
- Comandos de shell
- Consultas SQL

### APIs
- Información meteorológica
- Datos bursátiles
- Wrappers de API externos

## Creación de Herramientas Personalizadas

Puedes crear herramientas personalizadas para conectar LLMs a cualquier sistema externo:

```javascript
// Ejemplo en JavaScript
import { Tool } from "langchain/tools";

const customTool = new Tool({
  name: "weather-lookup",
  description: "Obtener el clima actual en una ubicación",
  func: async ({ location }) => {
    // Implementación que llama a una API del clima
    const response = await fetch(`https://api.weather.com?location=${location}`);
    const data = await response.json();
    return data.temperature;
  },
});
```

```python
# Ejemplo en Python
from langchain.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Optional, Type

class WeatherInput(BaseModel):
    location: str = Field(description="La ciudad y estado, ej. San Francisco, CA")

class WeatherTool(BaseTool):
    name = "weather-lookup"
    description = "Obtener el clima actual en una ubicación"
    args_schema: Type[BaseModel] = WeatherInput
    
    def _run(self, location: str) -> str:
        # Implementación que llama a una API del clima
        # Devolver la información del clima
        return f"22°C y soleado en {location}"
        
    def _arun(self, location: str) -> str:
        # Implementación asíncrona
        return self._run(location)
```

## Uso de Herramientas con Agentes

Las herramientas se vuelven especialmente poderosas cuando se usan con agentes, que pueden decidir qué herramientas usar y cuándo:

```python
# Ejemplo en Python
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

agent.run("¿Cuál es la capital de Francia y cuál es su población?")
```

## Mejores Prácticas para Herramientas

1. **Descripciones claras**: Proporciona instrucciones detalladas de uso en la descripción
2. **Validación de entradas**: Valida las entradas para prevenir errores
3. **Manejo de errores**: Maneja los fallos con elegancia y devuelve mensajes informativos
4. **Observabilidad**: Añade registro para rastrear el uso y rendimiento de la herramienta
5. **Permisos**: Considera las implicaciones de seguridad de herramientas que modifican sistemas