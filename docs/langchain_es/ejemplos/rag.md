# Ejemplo de Búsqueda en Documentos (RAG)

Este ejemplo demuestra cómo implementar un sistema de Recuperación Aumentada de Generación (RAG) que permite consultar documentos usando lenguaje natural.

## Implementación

```javascript
// Implementación en JavaScript/TypeScript
import { OpenAI } from "langchain/llms/openai";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { RetrievalQAChain } from "langchain/chains";
import { CheerioWebBaseLoader } from "langchain/document_loaders/web/cheerio";

async function createRAGSystem() {
  // 1. Cargar datos de un sitio web
  const loader = new CheerioWebBaseLoader(
    "https://docs.langchain.com/docs/components/chains/"
  );
  const docs = await loader.load();
  
  // 2. Dividir texto en chunks
  const textSplitter = new RecursiveCharacterTextSplitter({
    chunkSize: 1000,
    chunkOverlap: 200,
  });
  const splitDocs = await textSplitter.splitDocuments(docs);
  
  // 3. Crear embeddings para cada chunk
  const embeddings = new OpenAIEmbeddings();
  
  // 4. Almacenar chunks de documentos y sus embeddings
  const vectorStore = await MemoryVectorStore.fromDocuments(splitDocs, embeddings);
  
  // 5. Crear un recuperador
  const retriever = vectorStore.asRetriever({
    searchType: "similarity",
    k: 4, // Número de documentos a recuperar
  });
  
  // 6. Crear el modelo de lenguaje
  const model = new OpenAI({ temperature: 0 });
  
  // 7. Crear la cadena RAG
  const chain = RetrievalQAChain.fromLLM(model, retriever);
  
  return chain;
}

// Función para manejar consultas de usuario
async function queryDocuments(chain, query) {
  try {
    const response = await chain.call({
      query: query,
    });
    return response.text;
  } catch (error) {
    console.error("Error:", error);
    return "Ocurrió un error al procesar tu consulta.";
  }
}

// Ejemplo de uso
async function runRAGExample() {
  console.log("Creando sistema RAG...");
  const chain = await createRAGSystem();
  
  const queries = [
    "¿Qué son las chains en LangChain?",
    "¿Cómo puedo crear una sequential chain?",
    "¿Cuál es la diferencia entre map_reduce y refine chains?",
  ];
  
  for (const query of queries) {
    console.log(`\nConsulta: ${query}`);
    const result = await queryDocuments(chain, query);
    console.log(`Resultado: ${result}`);
  }
}

runRAGExample();
```

```python
# Implementación en Python
from langchain.llms import OpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import FAISS
from langchain.chains import RetrievalQA
from langchain.document_loaders import WebBaseLoader

def create_rag_system():
    # 1. Cargar datos de un sitio web
    loader = WebBaseLoader("https://docs.langchain.com/docs/components/chains/")
    docs = loader.load()
    
    # 2. Dividir texto en chunks
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,
        chunk_overlap=200
    )
    split_docs = text_splitter.split_documents(docs)
    
    # 3. Crear embeddings para cada chunk
    embeddings = OpenAIEmbeddings()
    
    # 4. Almacenar chunks de documentos y sus embeddings
    vector_store = FAISS.from_documents(split_docs, embeddings)
    
    # 5. Crear un recuperador
    retriever = vector_store.as_retriever(
        search_type="similarity",
        search_kwargs={"k": 4}  # Número de documentos a recuperar
    )
    
    # 6. Crear el modelo de lenguaje
    model = OpenAI(temperature=0)
    
    # 7. Crear la cadena RAG
    chain = RetrievalQA.from_chain_type(
        llm=model,
        chain_type="stuff",  # Otras opciones: "map_reduce", "refine"
        retriever=retriever,
        return_source_documents=True
    )
    
    return chain

# Función para manejar consultas de usuario
def query_documents(chain, query):
    try:
        response = chain({"query": query})
        return response["result"], response["source_documents"]
    except Exception as e:
        print(f"Error: {e}")
        return "Ocurrió un error al procesar tu consulta.", []

# Ejemplo de uso
def run_rag_example():
    print("Creando sistema RAG...")
    chain = create_rag_system()
    
    queries = [
        "¿Qué son las chains en LangChain?",
        "¿Cómo puedo crear una sequential chain?",
        "¿Cuál es la diferencia entre map_reduce y refine chains?",
    ]
    
    for query in queries:
        print(f"\nConsulta: {query}")
        result, sources = query_documents(chain, query)
        print(f"Resultado: {result}")
        print("\nFuentes:")
        for i, doc in enumerate(sources[:2]):  # Mostrar solo las primeras 2 fuentes
            print(f"Fuente {i+1}: {doc.page_content[:150]}...")

if __name__ == "__main__":
    run_rag_example()
```

## Cómo Funciona

1. **Carga de Documentos**: El sistema comienza cargando contenido de una página web sobre las chains de LangChain.

2. **División de Texto**: El documento se divide en fragmentos más pequeños que caben dentro de la ventana de contexto del modelo de lenguaje.

3. **Creación de Embeddings**: Cada fragmento de texto se convierte en un embedding vectorial que captura su significado semántico.

4. **Almacenamiento Vectorial**: Estos embeddings se almacenan en una base de datos vectorial (en memoria para JavaScript, FAISS para Python).

5. **Recuperación**: Cuando se recibe una consulta, el sistema:
   - Convierte la consulta a un embedding
   - Encuentra los fragmentos de documento más similares
   - Recupera estos fragmentos

6. **Generación**: El LLM genera una respuesta basada en:
   - La consulta original
   - Los fragmentos de documento recuperados

## Opciones de Personalización

- **Fuentes de Datos**: Reemplaza WebBaseLoader con otros cargadores para PDFs, CSVs, bases de datos, etc.
- **Estrategia de División**: Ajusta el tamaño del chunk y el solapamiento según tu contenido
- **Método de Recuperación**: Cambia de búsqueda por similitud a búsqueda híbrida o MMR
- **Tipo de Chain**: Usa diferentes tipos de chain como "map_reduce" para documentos grandes o "refine" para procesamiento incremental

## Extensiones

- Añadir filtrado por metadatos para una recuperación más precisa
- Implementar memoria conversacional para manejar preguntas de seguimiento
- Mejorar con citación de fuentes y explicación
- Optimizar el rendimiento con caché y procesamiento por lotes