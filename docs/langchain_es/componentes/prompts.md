# Prompts

Los prompts son las instrucciones o consultas proporcionadas a los modelos de lenguaje. Un prompteo efectivo es fundamental para obtener resultados de alta calidad de los LLMs.

## Componentes del Prompt

Un prompt bien estructurado típicamente incluye:

1. **Instrucciones**: Direcciones claras sobre lo que el modelo debe hacer
2. **Contexto**: Información de fondo necesaria para la tarea
3. **Datos de entrada**: Información específica para procesar
4. **Formato de salida**: Estructura esperada de la respuesta

## Plantillas de Prompts en LangChain

LangChain proporciona plantillas de prompts para hacer que el trabajo con prompts sea más estructurado:

```javascript
// Ejemplo en JavaScript
import { PromptTemplate } from "langchain/prompts";

const template = "Resume el siguiente texto en {word_count} palabras: {text}";
const prompt = PromptTemplate.fromTemplate(template);

const formattedPrompt = await prompt.format({
  word_count: "100",
  text: "Texto largo para resumir...",
});
```

```python
# Ejemplo en Python
from langchain.prompts import PromptTemplate

template = "Resume el siguiente texto en {word_count} palabras: {text}"
prompt = PromptTemplate(
    input_variables=["word_count", "text"],
    template=template,
)

formatted_prompt = prompt.format(word_count="100", text="Texto largo para resumir...")
```

## Técnicas Avanzadas de Prompts

### Ejemplos Few-shot
Proporcionar ejemplos de pares esperados de entrada/salida:

```
Clasifica el sentimiento (positivo, negativo, neutral):

Texto: La comida estaba deliciosa.
Sentimiento: positivo

Texto: El servicio fue lento.
Sentimiento: negativo

Texto: {input_text}
Sentimiento:
```

### Prompteo Chain-of-Thought
Guiar al modelo a través de un proceso de razonamiento:

```
Problema: Si Juan tiene 5 manzanas y le da 2 a María, ¿cuántas le quedan?
Proceso de pensamiento: Juan comenzó con 5 manzanas. Le dio 2 a María. 5 - 2 = 3.
Respuesta: 3 manzanas

Problema: {problem}
Proceso de pensamiento:
```

## Mejores Prácticas para Prompts

1. **Sé específico**: Define claramente lo que quieres que haga el modelo
2. **Proporciona contexto**: Da la información de fondo necesaria
3. **Usa ejemplos**: Muestra al modelo el formato esperado
4. **Desglosa tareas complejas**: Usa instrucciones paso a paso
5. **Prueba e itera**: Refina los prompts basándote en los resultados
6. **Considera role prompting**: Define un rol específico para el modelo (ej., "Actúa como un experto en...")