# Índices y Recuperación (RAG)

Los índices en LangChain permiten el almacenamiento y recuperación eficientes de información, formando la base para los sistemas de Recuperación Aumentada de Generación (RAG).

## Conceptos Principales

### Embeddings
Representaciones vectoriales de texto que capturan el significado semántico:

```javascript
// Ejemplo en JavaScript
import { OpenAIEmbeddings } from "langchain/embeddings/openai";

const embeddings = new OpenAIEmbeddings();
const documentEmbedding = await embeddings.embedDocuments(["Hola mundo"]);
const queryEmbedding = await embeddings.embedQuery("Hola a todos");
```

```python
# Ejemplo en Python
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
document_embedding = embeddings.embed_documents(["Hola mundo"])
query_embedding = embeddings.embed_query("Hola a todos")
```

### Cargadores de Documentos
Componentes que extraen texto de varias fuentes:

```python
# Ejemplo en Python
from langchain.document_loaders import PyPDFLoader, TextLoader, WebBaseLoader

# Cargar PDF
pdf_loader = PyPDFLoader("documento.pdf")
pdf_docs = pdf_loader.load()

# Cargar archivo de texto
text_loader = TextLoader("datos.txt")
text_docs = text_loader.load()

# Cargar página web
web_loader = WebBaseLoader("https://www.ejemplo.com")
web_docs = web_loader.load()
```

### Divisores de Texto
Dividen documentos en fragmentos manejables:

```python
# Ejemplo en Python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

documents = text_loader.load()
chunks = text_splitter.split_documents(documents)
```

### Almacenes Vectoriales
Bases de datos optimizadas para almacenar y recuperar embeddings vectoriales:

```python
# Ejemplo en Python
from langchain.vectorstores import Chroma, FAISS

# Usando Chroma
db = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Usando FAISS (en memoria)
faiss_db = FAISS.from_documents(
    documents=chunks,
    embedding=embeddings
)

# Guardar y cargar FAISS
faiss_db.save_local("faiss_index")
loaded_db = FAISS.load_local("faiss_index", embeddings)
```

## Recuperación Aumentada de Generación (RAG)

RAG combina la recuperación de información relevante con la generación del LLM:

```python
# Ejemplo en Python
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# Crear recuperador
retriever = db.as_retriever(search_kwargs={"k": 3})

# Crear cadena QA
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",  # Otras opciones: "map_reduce", "refine"
    retriever=retriever,
    return_source_documents=True
)

# Consultar el sistema
result = qa_chain({"query": "¿Cuál es la capital de Francia?"})
print(result["result"])  # Respuesta
print(result["source_documents"])  # Documentos fuente
```

## Técnicas Avanzadas de Recuperación

### Filtrado por Metadatos
Limitar la recuperación a documentos que coincidan con metadatos específicos:

```python
# Ejemplo en Python
filtered_retriever = db.as_retriever(
    search_kwargs={
        "k": 5,
        "filter": {"categoria": "finanzas", "año": "2023"}
    }
)
```

### Búsqueda Híbrida
Combinar búsqueda por palabras clave y semántica:

```python
# Ejemplo en Python
from langchain.retrievers import BM25Retriever, EnsembleRetriever

# Recuperador basado en palabras clave
bm25_retriever = BM25Retriever.from_documents(documents)
bm25_retriever.k = 5

# Recuperador basado en vectores
vector_retriever = db.as_retriever(search_kwargs={"k": 5})

# Combinar ambos
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, vector_retriever],
    weights=[0.3, 0.7]
)
```

### Recuperador de Auto-Consulta
Permite que el LLM genere consultas estructuradas:

```python
# Ejemplo simplificado en Python
from langchain.retrievers.self_query.base import SelfQueryRetriever

retriever = SelfQueryRetriever.from_llm(
    llm,
    db,
    document_contents="Artículos científicos sobre cambio climático",
    metadata_field_info=[
        {"name": "año", "type": "integer"},
        {"name": "autor", "type": "string"}
    ]
)

# El LLM analizará esto en una consulta estructurada
result = retriever.get_relevant_documents(
    "Encuentra artículos sobre el nivel del mar por James Hansen después de 2010"
)
```

## Mejores Prácticas

1. **Elegir tamaños de fragmento apropiados**: Equilibrar contexto y relevancia
2. **Optimizar para recall vs. precisión**: Ajustar k y umbrales de similitud
3. **Usar metadatos efectivamente**: Añadir metadatos ricos para permitir filtrado
4. **Considerar múltiples métodos de recuperación**: Diferentes enfoques funcionan mejor para diferentes contenidos
5. **Evaluar la calidad de recuperación**: Probar con consultas conocidas y resultados esperados