# Indexes and Retrieval (RAG)

Indexes in LangChain enable efficient storage and retrieval of information, forming the foundation for Retrieval Augmented Generation (RAG) systems.

## Core Concepts

### Embeddings
Vector representations of text that capture semantic meaning:

```javascript
// JavaScript example
import { OpenAIEmbeddings } from "langchain/embeddings/openai";

const embeddings = new OpenAIEmbeddings();
const documentEmbedding = await embeddings.embedDocuments(["Hello world"]);
const queryEmbedding = await embeddings.embedQuery("Hi there");
```

```python
# Python example
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
document_embedding = embeddings.embed_documents(["Hello world"])
query_embedding = embeddings.embed_query("Hi there")
```

### Document Loaders
Components that extract text from various sources:

```python
# Python example
from langchain.document_loaders import PyPDFLoader, TextLoader, WebBaseLoader

# Load PDF
pdf_loader = PyPDFLoader("document.pdf")
pdf_docs = pdf_loader.load()

# Load text file
text_loader = TextLoader("data.txt")
text_docs = text_loader.load()

# Load web page
web_loader = WebBaseLoader("https://www.example.com")
web_docs = web_loader.load()
```

### Text Splitters
Break documents into manageable chunks:

```python
# Python example
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

documents = text_loader.load()
chunks = text_splitter.split_documents(documents)
```

### Vector Stores
Databases optimized for storing and retrieving vector embeddings:

```python
# Python example
from langchain.vectorstores import Chroma, FAISS

# Using Chroma
db = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Using FAISS (in-memory)
faiss_db = FAISS.from_documents(
    documents=chunks,
    embedding=embeddings
)

# Saving and loading FAISS
faiss_db.save_local("faiss_index")
loaded_db = FAISS.load_local("faiss_index", embeddings)
```

## Retrieval Augmented Generation (RAG)

RAG combines retrieval of relevant information with LLM generation:

```python
# Python example
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# Create retriever
retriever = db.as_retriever(search_kwargs={"k": 3})

# Create QA chain
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",  # Other options: "map_reduce", "refine"
    retriever=retriever,
    return_source_documents=True
)

# Query the system
result = qa_chain({"query": "What is the capital of France?"})
print(result["result"])  # Answer
print(result["source_documents"])  # Source documents
```

## Advanced Retrieval Techniques

### Metadata Filtering
Limit retrieval to documents matching specific metadata:

```python
# Python example
filtered_retriever = db.as_retriever(
    search_kwargs={
        "k": 5,
        "filter": {"category": "finance", "year": "2023"}
    }
)
```

### Hybrid Search
Combine keyword and semantic search:

```python
# Python example
from langchain.retrievers import BM25Retriever, EnsembleRetriever

# Keyword-based retriever
bm25_retriever = BM25Retriever.from_documents(documents)
bm25_retriever.k = 5

# Vector-based retriever
vector_retriever = db.as_retriever(search_kwargs={"k": 5})

# Combine both
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, vector_retriever],
    weights=[0.3, 0.7]
)
```

### Self-Query Retriever
Allows the LLM to generate structured queries:

```python
# Python simplified example
from langchain.retrievers.self_query.base import SelfQueryRetriever

retriever = SelfQueryRetriever.from_llm(
    llm,
    db,
    document_contents="Scientific papers about climate change",
    metadata_field_info=[
        {"name": "year", "type": "integer"},
        {"name": "author", "type": "string"}
    ]
)

# The LLM will parse this into a structured query
result = retriever.get_relevant_documents(
    "Find papers about sea levels by James Hansen after 2010"
)
```

## Best Practices

1. **Choose appropriate chunk sizes**: Balance context and relevance
2. **Optimize for recall vs. precision**: Adjust k and similarity thresholds
3. **Use metadata effectively**: Add rich metadata to enable filtering
4. **Consider multiple retrieval methods**: Different approaches work better for different content
5. **Evaluate retrieval quality**: Test with known queries and expected results