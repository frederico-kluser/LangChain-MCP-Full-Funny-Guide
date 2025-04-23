# Ejemplo de Resumen de Texto

Este ejemplo demuestra cómo crear un sistema de resumen de documentos utilizando LangChain.

## Implementación

```javascript
// Implementación en JavaScript/TypeScript
import { OpenAI } from "langchain/llms/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { loadSummarizationChain } from "langchain/chains";
import { Document } from "langchain/document";
import * as fs from "fs";

async function summarizeText(text, type = "map_reduce") {
  // 1. Inicializar el modelo de lenguaje
  const model = new OpenAI({
    temperature: 0,
    modelName: "gpt-3.5-turbo-instruct", // Usando modelo Instruct para resumen
  });
  
  // 2. Dividir texto en chunks si es largo
  const textSplitter = new RecursiveCharacterTextSplitter({
    chunkSize: 3000,
    chunkOverlap: 200,
  });
  
  const docs = await textSplitter.createDocuments([text]);
  
  // 3. Crear la cadena de resumen
  const chain = loadSummarizationChain(model, { 
    type: type, // map_reduce, stuff, o refine
  });
  
  // 4. Ejecutar la cadena
  const result = await chain.call({
    input_documents: docs,
  });
  
  return result.text;
}

// Función para cargar texto desde un archivo
function loadTextFromFile(filePath) {
  return fs.readFileSync(filePath, "utf8");
}

// Ejemplo de uso
async function runSummarizationExample() {
  // Texto de ejemplo - artículo sobre cambio climático (sustituir con tu archivo)
  const sampleText = `
    El cambio climático es la alteración a largo plazo de la temperatura y los patrones climáticos típicos en un lugar.
    El cambio climático podría referirse a un lugar en particular o al planeta en su conjunto.
    El cambio climático puede hacer que los patrones climáticos sean menos predecibles.
    Estos patrones climáticos inesperados pueden dificultar el mantenimiento y cultivo en regiones que dependen de la agricultura.
    El cambio climático también se ha relacionado con otros eventos climáticos dañinos como huracanes, inundaciones, aguaceros y tormentas invernales más frecuentes e intensos.
    
    En las regiones polares, las temperaturas globales en aumento asociadas con el cambio climático han significado que las capas de hielo y los glaciares se están derritiendo a un ritmo acelerado.
    Esto contribuye al aumento del nivel del mar en diferentes regiones del planeta.
    Junto con la expansión de las aguas oceánicas debido al aumento de las temperaturas, el resultante aumento del nivel del mar ha creado desafíos para las personas que viven en regiones costeras.
    
    Muchos científicos del clima están de acuerdo en que las actividades humanas han acelerado el calentamiento global.
    La quema de combustibles fósiles, como el carbón y el petróleo, ha aumentado la cantidad de dióxido de carbono en la atmósfera.
    Esto ha mejorado el efecto invernadero, que calienta naturalmente la superficie de la Tierra.
    La deforestación también ha contribuido al aumento del dióxido de carbono atmosférico.
    Procesos agrícolas como el cultivo de arroz y la cría de ganado han aumentado los niveles de metano, que es otro gas de efecto invernadero.
    
    Diferentes regiones del mundo tienen diversos enfoques para abordar los desafíos del cambio climático.
    Muchos países se han comprometido a reducir las emisiones de carbono e invertir en fuentes de energía renovable como la solar y eólica.
    Los esfuerzos de conservación, como la protección de bosques y el desarrollo de prácticas agrícolas más sostenibles, también son estrategias para mitigar los efectos del cambio climático.
  `;
  
  console.log("Resumiendo texto...");
  
  // Enfoque map-reduce (bueno para documentos más largos)
  const mapReduceSummary = await summarizeText(sampleText, "map_reduce");
  console.log("\nResumen Map-Reduce:");
  console.log(mapReduceSummary);
  
  // Enfoque stuff (bueno para documentos más cortos)
  const stuffSummary = await summarizeText(sampleText, "stuff");
  console.log("\nResumen Stuff:");
  console.log(stuffSummary);
  
  // Enfoque refine (refinamiento secuencial)
  const refineSummary = await summarizeText(sampleText, "refine");
  console.log("\nResumen Refine:");
  console.log(refineSummary);
}

runSummarizationExample();
```

```python
# Implementación en Python
from langchain.llms import OpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains.summarize import load_summarize_chain
from langchain.docstore.document import Document

def summarize_text(text, type="map_reduce"):
    # 1. Inicializar el modelo de lenguaje
    model = OpenAI(
        temperature=0,
        model_name="gpt-3.5-turbo-instruct"  # Usando modelo Instruct para resumen
    )
    
    # 2. Dividir texto en chunks si es largo
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=3000,
        chunk_overlap=200
    )
    
    docs = text_splitter.create_documents([text])
    
    # 3. Crear la cadena de resumen
    chain = load_summarize_chain(
        model, 
        chain_type=type  # map_reduce, stuff, o refine
    )
    
    # 4. Ejecutar la cadena
    result = chain.run(docs)
    
    return result

# Función para cargar texto desde un archivo
def load_text_from_file(file_path):
    with open(file_path, "r") as file:
        return file.read()

# Ejemplo de uso
def run_summarization_example():
    # Texto de ejemplo - artículo sobre cambio climático (sustituir con tu archivo)
    sample_text = """
    El cambio climático es la alteración a largo plazo de la temperatura y los patrones climáticos típicos en un lugar.
    El cambio climático podría referirse a un lugar en particular o al planeta en su conjunto.
    El cambio climático puede hacer que los patrones climáticos sean menos predecibles.
    Estos patrones climáticos inesperados pueden dificultar el mantenimiento y cultivo en regiones que dependen de la agricultura.
    El cambio climático también se ha relacionado con otros eventos climáticos dañinos como huracanes, inundaciones, aguaceros y tormentas invernales más frecuentes e intensos.
    
    En las regiones polares, las temperaturas globales en aumento asociadas con el cambio climático han significado que las capas de hielo y los glaciares se están derritiendo a un ritmo acelerado.
    Esto contribuye al aumento del nivel del mar en diferentes regiones del planeta.
    Junto con la expansión de las aguas oceánicas debido al aumento de las temperaturas, el resultante aumento del nivel del mar ha creado desafíos para las personas que viven en regiones costeras.
    
    Muchos científicos del clima están de acuerdo en que las actividades humanas han acelerado el calentamiento global.
    La quema de combustibles fósiles, como el carbón y el petróleo, ha aumentado la cantidad de dióxido de carbono en la atmósfera.
    Esto ha mejorado el efecto invernadero, que calienta naturalmente la superficie de la Tierra.
    La deforestación también ha contribuido al aumento del dióxido de carbono atmosférico.
    Procesos agrícolas como el cultivo de arroz y la cría de ganado han aumentado los niveles de metano, que es otro gas de efecto invernadero.
    
    Diferentes regiones del mundo tienen diversos enfoques para abordar los desafíos del cambio climático.
    Muchos países se han comprometido a reducir las emisiones de carbono e invertir en fuentes de energía renovable como la solar y eólica.
    Los esfuerzos de conservación, como la protección de bosques y el desarrollo de prácticas agrícolas más sostenibles, también son estrategias para mitigar los efectos del cambio climático.
    """
    
    print("Resumiendo texto...")
    
    # Enfoque map-reduce (bueno para documentos más largos)
    map_reduce_summary = summarize_text(sample_text, "map_reduce")
    print("\nResumen Map-Reduce:")
    print(map_reduce_summary)
    
    # Enfoque stuff (bueno para documentos más cortos)
    stuff_summary = summarize_text(sample_text, "stuff")
    print("\nResumen Stuff:")
    print(stuff_summary)
    
    # Enfoque refine (refinamiento secuencial)
    refine_summary = summarize_text(sample_text, "refine")
    print("\nResumen Refine:")
    print(refine_summary)

if __name__ == "__main__":
    run_summarization_example()
```

## Cómo Funciona

1. **Preparación del Texto**: El ejemplo comienza con un texto de muestra sobre cambio climático, que podría ser reemplazado con contenido de un archivo.

2. **División del Texto**: Para documentos más largos, el texto se divide en fragmentos manejables que caben dentro de la ventana de contexto del modelo.

3. **Selección de Chain**: Se demuestran tres enfoques diferentes de resumen:

   - **Map-Reduce**: 
     - Cada fragmento se resume independientemente (map)
     - Los resúmenes se combinan en un resumen final (reduce)
     - Mejor para documentos más largos con muchos fragmentos

   - **Stuff**: 
     - Todo el texto se "mete" en un solo prompt
     - Simple pero limitado por el tamaño de la ventana de contexto
     - Mejor para documentos más cortos

   - **Refine**: 
     - Procesa fragmentos secuencialmente
     - Cada paso refina el resumen del paso anterior
     - Puede producir resultados más coherentes para documentos complejos

4. **Procesamiento del Modelo**: El modelo de lenguaje genera el resumen basado en el tipo de chain y la entrada de texto.

## Opciones de Personalización

- **Selección de Modelo**: Cambiar a modelos más potentes como GPT-4 para resúmenes de mayor calidad
- **Parámetros de Chain**: Personalizar prompts para cada tipo de chain para mejorar resultados
- **Tipos de Documento**: Extender para manejar diferentes formatos de documentos (PDFs, páginas web, etc.)
- **Formato de Salida**: Modificar prompts para obtener puntos, longitudes específicas o resúmenes enfocados

## Casos de Uso

- Resumir artículos de investigación largos
- Crear resúmenes de artículos
- Condensar transcripciones de reuniones
- Generar resúmenes ejecutivos de informes
- Simplificar documentación técnica compleja