# Exemplo: Resumo de Texto

Este exemplo demonstra como criar um sistema para resumir documentos longos usando LangChain.

## Código (JavaScript)

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { loadSummarizationChain } from "langchain/chains";
import * as fs from "fs";

// Carregar documento longo
const documentoTexto = fs.readFileSync("caminho/para/documento-longo.txt", "utf8");

// Configurar o modelo
const modelo = new ChatOpenAI({
  temperature: 0,
  modelName: "gpt-3.5-turbo",
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Criar divisor de texto
const textSplitter = new RecursiveCharacterTextSplitter({
  chunkSize: 2000,  // Tamanho de cada chunk em caracteres
  chunkOverlap: 200, // Sobreposição entre chunks
});

// Dividir o texto em chunks
async function resumirDocumento(texto) {
  console.log("Dividindo o documento em partes...");
  const chunks = await textSplitter.createDocuments([texto]);
  console.log(`Documento dividido em ${chunks.length} partes`);
  
  // Escolher tipo de chain de resumo
  // 'map_reduce' processa cada chunk individualmente e depois combina
  // 'stuff' combina tudo em um só prompt (limite de tokens)
  // 'refine' refina incrementalmente o resumo
  const chain = loadSummarizationChain(modelo, { type: "map_reduce" });
  
  console.log("Gerando resumo...");
  const resposta = await chain.call({
    input_documents: chunks,
  });
  
  return resposta.text;
}

// Executar o resumo
async function executar() {
  console.log("Começando o processo de resumo...");
  const resumo = await resumirDocumento(documentoTexto);
  console.log("\nResumo do documento:");
  console.log(resumo);
  
  // Opcional: salvar o resumo em um arquivo
  fs.writeFileSync("resumo-documento.txt", resumo, "utf8");
}

executar();
```

## Explicado passo a passo

1. **Importação dos módulos necessários**:
   - `ChatOpenAI`: Modelo de linguagem
   - `RecursiveCharacterTextSplitter`: Divide texto em chunks gerenciáveis
   - `loadSummarizationChain`: Chain especializada em resumo
   - `fs`: Para ler e escrever arquivos

2. **Carregamento do documento**:
   - Lendo o documento de texto do sistema de arquivos

3. **Configuração do modelo e do divisor de texto**:
   - Temperatura baixa para maior determinismo
   - Parâmetros de divisão de texto otimizados para resumo

4. **Processamento e resumo**:
   - Divisão do texto em chunks menores
   - Aplicação da chain de resumo com a estratégia map_reduce

5. **Retorno e salvar resultado**:
   - Exibição do resumo
   - Salvar em arquivo para uso posterior

## Estratégias de Resumo

O LangChain oferece diferentes estratégias para resumo de documentos longos:

### 1. Map-Reduce

```
Documento → [Chunk1, Chunk2, Chunk3, ...] → [Resumo1, Resumo2, Resumo3, ...] → Resumo Final
```

- **Vantagens**: Escalabilidade para documentos muito longos
- **Desvantagens**: Pode perder coerência global

### 2. Refine

```
Documento → [Chunk1, Chunk2, Chunk3, ...] → Resumo1 → Refinar com Chunk2 → Resumo2 → Refinar com Chunk3 → Resumo Final
```

- **Vantagens**: Mantém contexto entre chunks
- **Desvantagens**: Mais chamadas de API, mais lento

### 3. Stuff

```
Documento → Combinar todos os chunks → Resumo Final
```

- **Vantagens**: Mantém coerência global
- **Desvantagens**: Limitado pelo tamanho máximo de tokens do modelo

## Customizações possíveis

1. **Mudar a estratégia para 'refine'**:
   ```javascript
   // Usando refine para maior coerência
   const chain = loadSummarizationChain(modelo, { 
     type: "refine",
     refinementPrompt: "Refine o resumo existente com este texto adicional: {document}\n\nResumo atual: {existing_answer}"
   });
   ```

2. **Personalizar prompts de resumo**:
   ```javascript
   import { PromptTemplate } from "langchain/prompts";
   
   // Prompt customizado para resumo
   const mapPrompt = new PromptTemplate({
     template: "Faça um resumo detalhado do seguinte texto: {text}\n\nResumo:",
     inputVariables: ["text"],
   });
   
   const combinePrompt = new PromptTemplate({
     template: "Combine os resumos a seguir em um resumo coerente e abrangente:\n{text}\n\nResumo final:",
     inputVariables: ["text"],
   });
   
   const chain = loadSummarizationChain(modelo, {
     type: "map_reduce",
     mapPrompt,
     combinePrompt,
   });
   ```

3. **Adicionar metadados e instruções específicas**:
   ```javascript
   // Adicionar metadados aos documentos
   const docs = await textSplitter.createDocuments(
     [texto],
     [
       { source: "artigo-cientifico.pdf", tipo: "academico" },
     ]
   );
   
   // Prompt que usa metadados
   const mapPrompt = new PromptTemplate({
     template: "Resumo do texto {source} do tipo {tipo}: {text}\n\nResumo:",
     inputVariables: ["text", "source", "tipo"],
   });
   ```