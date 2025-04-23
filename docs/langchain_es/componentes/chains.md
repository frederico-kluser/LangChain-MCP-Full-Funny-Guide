# Chains

Las chains (cadenas) son secuencias de operaciones que combinan LLMs con otros componentes para crear aplicaciones más complejas. Permiten flujos de trabajo multi-paso que pueden procesar, transformar y generar información.

## Concepto Básico

Una chain típicamente consiste en:

1. **Entrada**: Los datos iniciales para la chain
2. **Pasos de procesamiento**: Una secuencia de operaciones para transformar la entrada
3. **Salida**: El resultado final después de todo el procesamiento

## Tipos Comunes de Chains en LangChain

### LLMChain
La chain más básica que combina una plantilla de prompt con un LLM:

```javascript
// Ejemplo en JavaScript
import { OpenAI } from "langchain/llms/openai";
import { PromptTemplate } from "langchain/prompts";
import { LLMChain } from "langchain/chains";

const llm = new OpenAI({ temperature: 0.7 });
const template = "¿Cuál es la capital de {country}?";
const prompt = PromptTemplate.fromTemplate(template);

const chain = new LLMChain({ llm, prompt });
const result = await chain.run({ country: "Francia" });
console.log(result); // París
```

```python
# Ejemplo en Python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

llm = OpenAI(temperature=0.7)
template = "¿Cuál es la capital de {country}?"
prompt = PromptTemplate(template=template, input_variables=["country"])

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run(country="Francia")
print(result) # París
```

### SequentialChain
Combina múltiples chains donde la salida de una chain se convierte en la entrada para la siguiente:

```python
# Ejemplo en Python
from langchain.chains import SimpleSequentialChain

# Chain 1: Generar un tema
template1 = "Genera un tema histórico aleatorio"
prompt1 = PromptTemplate(template=template1, input_variables=[])
chain1 = LLMChain(llm=llm, prompt=prompt1)

# Chain 2: Escribir un resumen sobre ese tema
template2 = "Escribe un breve resumen sobre: {topic}"
prompt2 = PromptTemplate(template=template2, input_variables=["topic"])
chain2 = LLMChain(llm=llm, prompt=prompt2)

# Combinar chains
overall_chain = SimpleSequentialChain(chains=[chain1, chain2])
result = overall_chain.run()
```

### RouterChain
Selecciona dinámicamente qué chain usar basándose en la entrada:

```python
# Ejemplo simplificado en Python
from langchain.chains.router import RouterChain

# Definir múltiples chains para diferentes tareas
summary_chain = LLMChain(llm=llm, prompt=summary_prompt)
translation_chain = LLMChain(llm=llm, prompt=translation_prompt)
question_chain = LLMChain(llm=llm, prompt=question_prompt)

# El router determina qué chain usar
router_chain = RouterChain(
    chains={"summary": summary_chain, 
            "translation": translation_chain, 
            "question_answering": question_chain},
    llm=llm
)

result = router_chain.run(user_input)
```

## Construcción de Chains Personalizadas

LangChain permite a los desarrolladores crear chains personalizadas:

1. Creando una nueva clase que extienda `BaseChain`
2. Implementando los métodos requeridos como `_call` y `run`
3. Definiendo variables de entrada/salida

## Características Avanzadas de Chains

- **Integración de memoria**: Las chains pueden incorporar memoria para mantener contexto
- **Uso de herramientas**: Las chains pueden usar herramientas para interactuar con sistemas externos
- **Callbacks**: Monitorear la ejecución y el rendimiento de la chain
- **Streaming**: Procesar salidas incrementalmente a medida que se generan