# Document Search (RAG) Example

This example demonstrates how to implement a Retrieval Augmented Generation (RAG) system that allows querying documents using natural language.

## Implementation

```javascript
// JavaScript/TypeScript implementation
import { OpenAI } from "langchain/llms/openai";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { RetrievalQAChain } from "langchain/chains";
import { CheerioWebBaseLoader } from "langchain/document_loaders/web/cheerio";

async function createRAGSystem() {
  // 1. Load data from a website
  const loader = new CheerioWebBaseLoader(
    "https://docs.langchain.com/docs/components/chains/"
  );
  const docs = await loader.load();
  
  // 2. Split text into chunks
  const textSplitter = new RecursiveCharacterTextSplitter({
    chunkSize: 1000,
    chunkOverlap: 200,
  });
  const splitDocs = await textSplitter.splitDocuments(docs);
  
  // 3. Create embeddings for each chunk
  const embeddings = new OpenAIEmbeddings();
  
  // 4. Store document chunks and their embeddings
  const vectorStore = await MemoryVectorStore.fromDocuments(splitDocs, embeddings);
  
  // 5. Create a retriever
  const retriever = vectorStore.asRetriever({
    searchType: "similarity",
    k: 4, // Number of documents to retrieve
  });
  
  // 6. Create the language model
  const model = new OpenAI({ temperature: 0 });
  
  // 7. Create the RAG chain
  const chain = RetrievalQAChain.fromLLM(model, retriever);
  
  return chain;
}

// Function to handle user queries
async function queryDocuments(chain, query) {
  try {
    const response = await chain.call({
      query: query,
    });
    return response.text;
  } catch (error) {
    console.error("Error:", error);
    return "An error occurred while processing your query.";
  }
}

// Example usage
async function runRAGExample() {
  console.log("Creating RAG system...");
  const chain = await createRAGSystem();
  
  const queries = [
    "What are chains in LangChain?",
    "How can I create a sequential chain?",
    "What's the difference between map_reduce and refine chains?",
  ];
  
  for (const query of queries) {
    console.log(`\nQuery: ${query}`);
    const result = await queryDocuments(chain, query);
    console.log(`Result: ${result}`);
  }
}

runRAGExample();
```

```python
# Python implementation
from langchain.llms import OpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import FAISS
from langchain.chains import RetrievalQA
from langchain.document_loaders import WebBaseLoader

def create_rag_system():
    # 1. Load data from a website
    loader = WebBaseLoader("https://docs.langchain.com/docs/components/chains/")
    docs = loader.load()
    
    # 2. Split text into chunks
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,
        chunk_overlap=200
    )
    split_docs = text_splitter.split_documents(docs)
    
    # 3. Create embeddings for each chunk
    embeddings = OpenAIEmbeddings()
    
    # 4. Store document chunks and their embeddings
    vector_store = FAISS.from_documents(split_docs, embeddings)
    
    # 5. Create a retriever
    retriever = vector_store.as_retriever(
        search_type="similarity",
        search_kwargs={"k": 4}  # Number of documents to retrieve
    )
    
    # 6. Create the language model
    model = OpenAI(temperature=0)
    
    # 7. Create the RAG chain
    chain = RetrievalQA.from_chain_type(
        llm=model,
        chain_type="stuff",  # Other options: "map_reduce", "refine"
        retriever=retriever,
        return_source_documents=True
    )
    
    return chain

# Function to handle user queries
def query_documents(chain, query):
    try:
        response = chain({"query": query})
        return response["result"], response["source_documents"]
    except Exception as e:
        print(f"Error: {e}")
        return "An error occurred while processing your query.", []

# Example usage
def run_rag_example():
    print("Creating RAG system...")
    chain = create_rag_system()
    
    queries = [
        "What are chains in LangChain?",
        "How can I create a sequential chain?",
        "What's the difference between map_reduce and refine chains?",
    ]
    
    for query in queries:
        print(f"\nQuery: {query}")
        result, sources = query_documents(chain, query)
        print(f"Result: {result}")
        print("\nSources:")
        for i, doc in enumerate(sources[:2]):  # Show first 2 sources only
            print(f"Source {i+1}: {doc.page_content[:150]}...")

if __name__ == "__main__":
    run_rag_example()
```

## How It Works

1. **Document Loading**: The system begins by loading content from a webpage about LangChain chains.

2. **Text Splitting**: The document is divided into smaller chunks that fit within the context window of the language model.

3. **Embedding Creation**: Each text chunk is converted into a vector embedding that captures its semantic meaning.

4. **Vector Storage**: These embeddings are stored in a vector database (in-memory for JavaScript, FAISS for Python).

5. **Retrieval**: When a query is received, the system:
   - Converts the query to an embedding
   - Finds the most similar document chunks
   - Retrieves these chunks

6. **Generation**: The LLM generates an answer based on:
   - The original query
   - The retrieved document chunks

## Customization Options

- **Data Sources**: Replace WebBaseLoader with other loaders for PDFs, CSVs, databases, etc.
- **Splitting Strategy**: Adjust chunk size and overlap based on your content
- **Retrieval Method**: Change from similarity search to hybrid search or MMR
- **Chain Type**: Use different chain types like "map_reduce" for large documents or "refine" for incremental processing

## Extensions

- Add metadata filtering for more precise retrieval
- Implement conversational memory to handle follow-up questions
- Enhance with sources citation and explanation
- Optimize performance with caching and batching