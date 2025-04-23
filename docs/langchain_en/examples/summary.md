# Text Summarization Example

This example demonstrates how to create a document summarization system using LangChain.

## Implementation

```javascript
// JavaScript/TypeScript implementation
import { OpenAI } from "langchain/llms/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { loadSummarizationChain } from "langchain/chains";
import { Document } from "langchain/document";
import * as fs from "fs";

async function summarizeText(text, type = "map_reduce") {
  // 1. Initialize the language model
  const model = new OpenAI({
    temperature: 0,
    modelName: "gpt-3.5-turbo-instruct", // Using Instruct model for summarization
  });
  
  // 2. Split text into chunks if it's long
  const textSplitter = new RecursiveCharacterTextSplitter({
    chunkSize: 3000,
    chunkOverlap: 200,
  });
  
  const docs = await textSplitter.createDocuments([text]);
  
  // 3. Create the summarization chain
  const chain = loadSummarizationChain(model, { 
    type: type, // map_reduce, stuff, or refine
  });
  
  // 4. Run the chain
  const result = await chain.call({
    input_documents: docs,
  });
  
  return result.text;
}

// Function to load text from a file
function loadTextFromFile(filePath) {
  return fs.readFileSync(filePath, "utf8");
}

// Example usage
async function runSummarizationExample() {
  // Example text - article on climate change (substitute with your file)
  const sampleText = `
    Climate change is the long-term alteration of temperature and typical weather patterns in a place. 
    Climate change could refer to a particular location or the planet as a whole. 
    Climate change may cause weather patterns to be less predictable. 
    These unexpected weather patterns can make it difficult to maintain and grow crops in regions that rely on farming. 
    Climate change has also been connected with other damaging weather events such as more frequent and more intense hurricanes, floods, downpours, and winter storms.
    
    In polar regions, the warming global temperatures associated with climate change have meant ice sheets and glaciers are melting at an accelerated rate. 
    This contributes to sea levels rising in different regions of the planet. 
    Together with expanding ocean waters due to rising temperatures, the resulting rise in sea level has created challenges for people living in coastal regions.
    
    Many climate scientists agree that human activities have accelerated global warming. 
    The burning of fossil fuels, such as coal and oil, has increased the amount of carbon dioxide in the atmosphere. 
    This has enhanced the greenhouse effect, which naturally warms the Earth's surface. 
    Deforestation has also contributed to increased atmospheric carbon dioxide. 
    Agricultural processes such as rice farming and raising livestock have increased levels of methane, which is another greenhouse gas.
    
    Different regions around the world have varying approaches to addressing climate change challenges. 
    Many countries have committed to reducing carbon emissions and investing in renewable energy sources like solar and wind power.
    Conservation efforts, such as protecting forests and developing more sustainable agricultural practices, are also strategies for mitigating the effects of climate change.
  `;
  
  console.log("Summarizing text...");
  
  // Map-reduce approach (good for longer documents)
  const mapReduceSummary = await summarizeText(sampleText, "map_reduce");
  console.log("\nMap-Reduce Summary:");
  console.log(mapReduceSummary);
  
  // Stuff approach (good for shorter documents)
  const stuffSummary = await summarizeText(sampleText, "stuff");
  console.log("\nStuff Summary:");
  console.log(stuffSummary);
  
  // Refine approach (sequential refinement)
  const refineSummary = await summarizeText(sampleText, "refine");
  console.log("\nRefine Summary:");
  console.log(refineSummary);
}

runSummarizationExample();
```

```python
# Python implementation
from langchain.llms import OpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains.summarize import load_summarize_chain
from langchain.docstore.document import Document

def summarize_text(text, type="map_reduce"):
    # 1. Initialize the language model
    model = OpenAI(
        temperature=0,
        model_name="gpt-3.5-turbo-instruct"  # Using Instruct model for summarization
    )
    
    # 2. Split text into chunks if it's long
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=3000,
        chunk_overlap=200
    )
    
    docs = text_splitter.create_documents([text])
    
    # 3. Create the summarization chain
    chain = load_summarize_chain(
        model, 
        chain_type=type  # map_reduce, stuff, or refine
    )
    
    # 4. Run the chain
    result = chain.run(docs)
    
    return result

# Function to load text from a file
def load_text_from_file(file_path):
    with open(file_path, "r") as file:
        return file.read()

# Example usage
def run_summarization_example():
    # Example text - article on climate change (substitute with your file)
    sample_text = """
    Climate change is the long-term alteration of temperature and typical weather patterns in a place. 
    Climate change could refer to a particular location or the planet as a whole. 
    Climate change may cause weather patterns to be less predictable. 
    These unexpected weather patterns can make it difficult to maintain and grow crops in regions that rely on farming. 
    Climate change has also been connected with other damaging weather events such as more frequent and more intense hurricanes, floods, downpours, and winter storms.
    
    In polar regions, the warming global temperatures associated with climate change have meant ice sheets and glaciers are melting at an accelerated rate. 
    This contributes to sea levels rising in different regions of the planet. 
    Together with expanding ocean waters due to rising temperatures, the resulting rise in sea level has created challenges for people living in coastal regions.
    
    Many climate scientists agree that human activities have accelerated global warming. 
    The burning of fossil fuels, such as coal and oil, has increased the amount of carbon dioxide in the atmosphere. 
    This has enhanced the greenhouse effect, which naturally warms the Earth's surface. 
    Deforestation has also contributed to increased atmospheric carbon dioxide. 
    Agricultural processes such as rice farming and raising livestock have increased levels of methane, which is another greenhouse gas.
    
    Different regions around the world have varying approaches to addressing climate change challenges. 
    Many countries have committed to reducing carbon emissions and investing in renewable energy sources like solar and wind power.
    Conservation efforts, such as protecting forests and developing more sustainable agricultural practices, are also strategies for mitigating the effects of climate change.
    """
    
    print("Summarizing text...")
    
    # Map-reduce approach (good for longer documents)
    map_reduce_summary = summarize_text(sample_text, "map_reduce")
    print("\nMap-Reduce Summary:")
    print(map_reduce_summary)
    
    # Stuff approach (good for shorter documents)
    stuff_summary = summarize_text(sample_text, "stuff")
    print("\nStuff Summary:")
    print(stuff_summary)
    
    # Refine approach (sequential refinement)
    refine_summary = summarize_text(sample_text, "refine")
    print("\nRefine Summary:")
    print(refine_summary)

if __name__ == "__main__":
    run_summarization_example()
```

## How It Works

1. **Text Preparation**: The example starts with a sample text on climate change, which could be replaced with content from a file.

2. **Text Splitting**: For longer documents, the text is divided into manageable chunks that fit within the model's context window.

3. **Chain Selection**: Three different summarization approaches are demonstrated:

   - **Map-Reduce**: 
     - Each chunk is summarized independently (map)
     - The summaries are combined into a final summary (reduce)
     - Best for longer documents with many chunks

   - **Stuff**: 
     - All text is "stuffed" into a single prompt
     - Simple but limited by context window size
     - Best for shorter documents

   - **Refine**: 
     - Processes chunks sequentially
     - Each step refines the summary from the previous step
     - Can produce more coherent results for complex documents

4. **Model Processing**: The language model generates the summary based on the chain type and text input.

## Customization Options

- **Model Selection**: Switch to more powerful models like GPT-4 for higher quality summaries
- **Chain Parameters**: Customize prompts for each chain type to improve results
- **Document Types**: Extend to handle different document formats (PDFs, webpages, etc.)
- **Output Format**: Modify prompts to get bullet points, specific lengths, or focused summaries

## Use Cases

- Summarizing long research papers
- Creating article abstracts
- Condensing meeting transcripts
- Generating executive summaries of reports
- Simplifying complex technical documentation