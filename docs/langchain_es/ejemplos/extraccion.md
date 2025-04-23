# Ejemplo de Extracción de Información

Este ejemplo demuestra cómo extraer información estructurada de texto no estructurado utilizando LangChain.

## Implementación

```javascript
// Implementación en JavaScript/TypeScript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { StructuredOutputParser } from "langchain/output_parsers";
import { PromptTemplate } from "langchain/prompts";
import { z } from "zod";

async function extractInformation(text, schema) {
  // 1. Inicializar el modelo de lenguaje
  const model = new ChatOpenAI({
    temperature: 0,
    modelName: "gpt-4", // Usando GPT-4 para mejor precisión en extracción
  });
  
  // 2. Crear un analizador basado en el esquema proporcionado
  const parser = StructuredOutputParser.fromZodSchema(schema);
  
  // 3. Crear una plantilla de prompt
  const formatInstructions = parser.getFormatInstructions();
  
  const prompt = new PromptTemplate({
    template: `Extrae la siguiente información del texto a continuación. 
    
{format_instructions}

Texto: {text}`,
    inputVariables: ["text"],
    partialVariables: { format_instructions: formatInstructions },
  });
  
  // 4. Formatear el prompt con el texto de entrada
  const input = await prompt.format({ text });
  
  // 5. Llamar al modelo y analizar la respuesta
  const response = await model.call(input);
  return parser.parse(response);
}

// Ejemplo de uso
async function runExtractionExample() {
  // Definir esquemas para diferentes tareas de extracción
  
  // 1. Esquema de Información de Contacto
  const contactSchema = z.object({
    name: z.string().describe("El nombre completo de la persona"),
    email: z.string().optional().describe("La dirección de correo electrónico de la persona, si está disponible"),
    phone: z.string().optional().describe("El número de teléfono de la persona, si está disponible"),
    address: z.string().optional().describe("La dirección física de la persona, si está disponible"),
  });
  
  // 2. Esquema de Información de Evento
  const eventSchema = z.object({
    eventName: z.string().describe("El nombre del evento"),
    date: z.string().describe("La fecha en que ocurre el evento"),
    time: z.string().optional().describe("La hora a la que comienza el evento"),
    location: z.string().describe("Dónde tiene lugar el evento"),
    description: z.string().describe("Una breve descripción del evento"),
  });
  
  // 3. Esquema de Reseña de Producto
  const reviewSchema = z.object({
    productName: z.string().describe("El nombre del producto que se está reseñando"),
    rating: z.number().min(1).max(5).describe("La calificación numérica (1-5)"),
    pros: z.array(z.string()).describe("Lista de aspectos positivos"),
    cons: z.array(z.string()).describe("Lista de aspectos negativos"),
    summary: z.string().describe("Un resumen de una frase de la reseña"),
  });
  
  // Textos de ejemplo
  const contactText = `
    Juan Pérez es nuestro nuevo representante de ventas. Puedes contactarlo en
    juan.perez@ejemplo.com o llamar a su oficina al (555) 123-4567.
    Su oficina está ubicada en Av. Negocio 123, Suite 456, Ciudad de México, CDMX 01000.
  `;
  
  const eventText = `
    Únete a nosotros para la Conferencia Anual de Tecnología 2023 el 15 de septiembre a las 9:00 AM.
    El evento se llevará a cabo en el Gran Centro de Convenciones, Blvd. Convención 789,
    San Francisco. El tema de este año se centra en los avances de IA y sus
    aplicaciones en diversas industrias. Establece contactos con líderes de la industria y
    obtén información sobre tecnologías de vanguardia.
  `;
  
  const reviewText = `
    Recientemente compré la laptop UltraBook Pro X1 y la he estado usando durante dos semanas.
    En general, le daría 4 de 5 estrellas.
    
    Lo que me encanta: La duración de la batería es excepcional, durando más de 10 horas. La pantalla es
    nítida con colores vibrantes. También es sorprendentemente ligera para una laptop con estas especificaciones.
    
    Los aspectos negativos: El teclado no tiene una gran retroalimentación táctil. Se calienta cuando se usan
    aplicaciones que consumen muchos recursos. El precio también está en el lado más alto en comparación con modelos similares.
    
    En resumen, la UltraBook Pro X1 es una laptop de alta calidad con gran duración de batería y pantalla,
    pero tiene pequeños problemas con el teclado y la gestión térmica.
  `;
  
  // Extraer información usando cada esquema
  console.log("Extrayendo información de contacto...");
  const contactInfo = await extractInformation(contactText, contactSchema);
  console.log(JSON.stringify(contactInfo, null, 2));
  
  console.log("\nExtrayendo información de evento...");
  const eventInfo = await extractInformation(eventText, eventSchema);
  console.log(JSON.stringify(eventInfo, null, 2));
  
  console.log("\nExtrayendo información de reseña de producto...");
  const reviewInfo = await extractInformation(reviewText, reviewSchema);
  console.log(JSON.stringify(reviewInfo, null, 2));
}

runExtractionExample();
```

```python
# Implementación en Python
from langchain.chat_models import ChatOpenAI
from langchain.output_parsers import PydanticOutputParser
from langchain.prompts import PromptTemplate
from pydantic import BaseModel, Field
from typing import List, Optional

class ContactInfo(BaseModel):
    name: str = Field(description="El nombre completo de la persona")
    email: Optional[str] = Field(None, description="La dirección de correo electrónico de la persona, si está disponible")
    phone: Optional[str] = Field(None, description="El número de teléfono de la persona, si está disponible")
    address: Optional[str] = Field(None, description="La dirección física de la persona, si está disponible")

class EventInfo(BaseModel):
    event_name: str = Field(description="El nombre del evento")
    date: str = Field(description="La fecha en que ocurre el evento")
    time: Optional[str] = Field(None, description="La hora a la que comienza el evento")
    location: str = Field(description="Dónde tiene lugar el evento")
    description: str = Field(description="Una breve descripción del evento")

class ProductReview(BaseModel):
    product_name: str = Field(description="El nombre del producto que se está reseñando")
    rating: int = Field(description="La calificación numérica (1-5)")
    pros: List[str] = Field(description="Lista de aspectos positivos")
    cons: List[str] = Field(description="Lista de aspectos negativos")
    summary: str = Field(description="Un resumen de una frase de la reseña")

def extract_information(text, schema_class):
    # 1. Inicializar el modelo de lenguaje
    model = ChatOpenAI(
        temperature=0,
        model_name="gpt-4"  # Usando GPT-4 para mejor precisión en extracción
    )
    
    # 2. Crear un analizador basado en el esquema proporcionado
    parser = PydanticOutputParser(pydantic_object=schema_class)
    
    # 3. Crear una plantilla de prompt
    format_instructions = parser.get_format_instructions()
    
    prompt = PromptTemplate(
        template="Extrae la siguiente información del texto a continuación.\n\n{format_instructions}\n\nTexto: {text}",
        input_variables=["text"],
        partial_variables={"format_instructions": format_instructions}
    )
    
    # 4. Formatear el prompt con el texto de entrada
    input_text = prompt.format(text=text)
    
    # 5. Llamar al modelo y analizar la respuesta
    response = model.predict(input_text)
    return parser.parse(response)

# Ejemplo de uso
def run_extraction_example():
    # Textos de ejemplo
    contact_text = """
    Juan Pérez es nuestro nuevo representante de ventas. Puedes contactarlo en
    juan.perez@ejemplo.com o llamar a su oficina al (555) 123-4567.
    Su oficina está ubicada en Av. Negocio 123, Suite 456, Ciudad de México, CDMX 01000.
    """
    
    event_text = """
    Únete a nosotros para la Conferencia Anual de Tecnología 2023 el 15 de septiembre a las 9:00 AM.
    El evento se llevará a cabo en el Gran Centro de Convenciones, Blvd. Convención 789,
    San Francisco. El tema de este año se centra en los avances de IA y sus
    aplicaciones en diversas industrias. Establece contactos con líderes de la industria y
    obtén información sobre tecnologías de vanguardia.
    """
    
    review_text = """
    Recientemente compré la laptop UltraBook Pro X1 y la he estado usando durante dos semanas.
    En general, le daría 4 de 5 estrellas.
    
    Lo que me encanta: La duración de la batería es excepcional, durando más de 10 horas. La pantalla es
    nítida con colores vibrantes. También es sorprendentemente ligera para una laptop con estas especificaciones.
    
    Los aspectos negativos: El teclado no tiene una gran retroalimentación táctil. Se calienta cuando se usan
    aplicaciones que consumen muchos recursos. El precio también está en el lado más alto en comparación con modelos similares.
    
    En resumen, la UltraBook Pro X1 es una laptop de alta calidad con gran duración de batería y pantalla,
    pero tiene pequeños problemas con el teclado y la gestión térmica.
    """
    
    # Extraer información usando cada esquema
    print("Extrayendo información de contacto...")
    contact_info = extract_information(contact_text, ContactInfo)
    print(contact_info.json(indent=2))
    
    print("\nExtrayendo información de evento...")
    event_info = extract_information(event_text, EventInfo)
    print(event_info.json(indent=2))
    
    print("\nExtrayendo información de reseña de producto...")
    review_info = extract_information(review_text, ProductReview)
    print(review_info.json(indent=2))

if __name__ == "__main__":
    run_extraction_example()
```

## Cómo Funciona

1. **Definición de Esquema**: El ejemplo define esquemas estructurados para diferentes tipos de información:
   - Información de contacto (nombre, correo electrónico, teléfono, dirección)
   - Detalles de eventos (nombre, fecha, hora, ubicación, descripción)
   - Reseñas de productos (nombre del producto, calificación, pros, contras, resumen)

2. **Creación del Analizador**: Se crea un analizador de salida estructurado a partir de cada esquema para:
   - Generar instrucciones de formato para el LLM
   - Analizar la salida del LLM en la estructura deseada

3. **Ingeniería de Prompt**: Se crea un prompt que incluye:
   - Instrucciones claras para la tarea de extracción
   - Requisitos de formato del analizador
   - El texto a analizar

4. **Invocación del Modelo**: Un modelo de lenguaje (GPT-4) procesa el prompt y devuelve una respuesta estructurada.

5. **Análisis de Respuesta**: El analizador convierte la respuesta de texto en la estructura definida por el esquema.

## Aplicaciones

- **Procesamiento de Documentos**: Extraer información clave de documentos legales, informes o formularios
- **Análisis de Correo Electrónico**: Extraer citas, acciones, contactos o información de reuniones de correos electrónicos
- **Minería de Reseñas**: Analizar reseñas de productos para atributos específicos
- **Análisis de Currículums**: Extraer información de candidatos, habilidades y experiencia de currículums
- **Análisis de Noticias**: Extraer entidades, hechos y relaciones de artículos de noticias

## Opciones de Personalización

- **Complejidad de Esquema**: Añadir estructuras anidadas o campos más detallados
- **Reglas de Validación**: Incluir validación de campo (ej. formato de correo electrónico, rangos numéricos)
- **Extracciones Múltiples**: Procesar conjuntos completos de documentos
- **Instrucciones Personalizadas**: Refinar prompts para necesidades de extracción específicas

## Ideas de Extensión

- Añadir preprocesamiento para texto ruidoso
- Implementar procesamiento por lotes para múltiples documentos
- Integrar con OCR para extracción de documentos físicos
- Crear un bucle de retroalimentación para mejorar la precisión de extracción con el tiempo