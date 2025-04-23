# Exemplo: Pesquisa em Documentos (RAG)

Este exemplo demonstra como implementar um sistema de Recuperação Aumentada por Geração (RAG) para consultar documentos usando LangChain.

## Código (JavaScript)

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { RetrievalQAChain } from "langchain/chains";
import * as fs from "fs";

// Carregar documento de exemplo
const text = fs.readFileSync("documentos/artigo.txt", "utf8");

// Inicializar o divisor de texto
const textSplitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});

// Dividir o documento em chunks
const splitDocs = await textSplitter.createDocuments([text]);
console.log(`Documento dividido em ${splitDocs.length} chunks`);

// Inicializar o modelo de embeddings
const embeddings = new OpenAIEmbeddings({
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Criar base de conhecimento vetorial
const vectorStore = await MemoryVectorStore.fromDocuments(
  splitDocs,
  embeddings
);

// Inicializar o modelo de LLM
const model = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY,
  modelName: "gpt-3.5-turbo",
  temperature: 0.2,
});

// Criar a chain de consulta com recuperação
const chain = RetrievalQAChain.fromLLM(model, vectorStore.asRetriever());

// Função para consultar o documento
async function perguntarDocumento(pergunta) {
  const resposta = await chain.call({
    query: pergunta,
  });
  
  console.log(`Pergunta: ${pergunta}`);
  console.log(`Resposta: ${resposta.text}`);
  return resposta.text;
}

// Exemplo de consultas
async function executarConsultas() {
  await perguntarDocumento("Quais são os principais tópicos abordados neste documento?");
  await perguntarDocumento("Explique o conceito X mencionado no documento");
  await perguntarDocumento("Resuma as conclusões do artigo");
}

executarConsultas();
```

## Explicado passo a passo

1. **Importação dos módulos necessários**:
   - Componentes para processamento de texto, embeddings, armazenamento vetorial e chains

2. **Carregamento e processamento do documento**:
   - Leitura do arquivo de texto
   - Divisão em chunks menores para processamento eficiente

3. **Criação de embeddings e armazenamento vetorial**:
   - Conversão do texto em representações vetoriais usando OpenAI Embeddings
   - Armazenamento dos vetores em memória para consulta

4. **Configuração do modelo e chain de recuperação**:
   - Inicialização do modelo de chat
   - Criação da chain que combina recuperação e geração de resposta

5. **Função para consulta**:
   - Recebe uma pergunta
   - Recupera documentos relevantes
   - Gera resposta contextualizada

6. **Exemplos de consultas**:
   - Demonstração de diferentes tipos de perguntas que podem ser feitas

## Como funciona o RAG

1. **Indexação (fase de pré-processamento)**:
   - O documento é dividido em chunks menores
   - Cada chunk é convertido em um vetor de embedding
   - Os vetores são armazenados em um banco de dados vetorial

2. **Consulta (em tempo real)**:
   - A pergunta do usuário é convertida em um vetor de embedding
   - O sistema recupera os chunks mais similares à pergunta
   - O LLM recebe a pergunta e os chunks recuperados como contexto
   - O modelo gera uma resposta baseada nos documentos recuperados

## Customizações possíveis

1. **Usar um armazenamento vetorial persistente**:
   ```javascript
   import { Chroma } from "langchain/vectorstores/chroma";
   
   // Criar um armazenamento persistente
   const vectorStore = await Chroma.fromDocuments(
     splitDocs,
     embeddings,
     { collectionName: "artigos" }
   );
   ```

2. **Personalizar a recuperação**:
   ```javascript
   // Configurar o retriever para buscar mais documentos
   const retriever = vectorStore.asRetriever({
     k: 5,  // Número de documentos a recuperar
     searchType: "similarity",  // Tipo de busca
   });
   ```

3. **Usar prompt personalizado**:
   ```javascript
   import { PromptTemplate } from "langchain/prompts";
   
   const template = `Use as seguintes partes do documento para responder à pergunta. 
   Se não souber a resposta, diga que não encontrou a informação no documento, não tente inventar.
   
   Contexto: {context}
   
   Pergunta: {question}`;
   
   const promptTemplate = new PromptTemplate({
     template,
     inputVariables: ["context", "question"],
   });
   ```