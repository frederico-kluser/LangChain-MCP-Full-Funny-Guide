# Information Extraction Example

This example demonstrates how to extract structured information from unstructured text using LangChain.

## Implementation

```javascript
// JavaScript/TypeScript implementation
import { ChatOpenAI } from "langchain/chat_models/openai";
import { StructuredOutputParser } from "langchain/output_parsers";
import { PromptTemplate } from "langchain/prompts";
import { z } from "zod";

async function extractInformation(text, schema) {
  // 1. Initialize the language model
  const model = new ChatOpenAI({
    temperature: 0,
    modelName: "gpt-4", // Using GPT-4 for better extraction accuracy
  });
  
  // 2. Create a parser based on the provided schema
  const parser = StructuredOutputParser.fromZodSchema(schema);
  
  // 3. Create a prompt template
  const formatInstructions = parser.getFormatInstructions();
  
  const prompt = new PromptTemplate({
    template: `Extract the following information from the text below. 
    
{format_instructions}

Text: {text}`,
    inputVariables: ["text"],
    partialVariables: { format_instructions: formatInstructions },
  });
  
  // 4. Format the prompt with the input text
  const input = await prompt.format({ text });
  
  // 5. Call the model and parse the response
  const response = await model.call(input);
  return parser.parse(response);
}

// Example usage
async function runExtractionExample() {
  // Define schemas for different extraction tasks
  
  // 1. Contact Information Schema
  const contactSchema = z.object({
    name: z.string().describe("The person's full name"),
    email: z.string().optional().describe("The person's email address, if available"),
    phone: z.string().optional().describe("The person's phone number, if available"),
    address: z.string().optional().describe("The person's physical address, if available"),
  });
  
  // 2. Event Information Schema
  const eventSchema = z.object({
    eventName: z.string().describe("The name of the event"),
    date: z.string().describe("The date when the event occurs"),
    time: z.string().optional().describe("The time when the event starts"),
    location: z.string().describe("Where the event takes place"),
    description: z.string().describe("A brief description of the event"),
  });
  
  // 3. Product Review Schema
  const reviewSchema = z.object({
    productName: z.string().describe("The name of the product being reviewed"),
    rating: z.number().min(1).max(5).describe("The numerical rating (1-5)"),
    pros: z.array(z.string()).describe("List of positive aspects"),
    cons: z.array(z.string()).describe("List of negative aspects"),
    summary: z.string().describe("A one-sentence summary of the review"),
  });
  
  // Example texts
  const contactText = `
    John Smith is our new sales representative. You can reach him at 
    john.smith@example.com or call his office at (555) 123-4567. 
    His office is located at 123 Business Ave, Suite 456, Chicago, IL 60601.
  `;
  
  const eventText = `
    Join us for the Annual Tech Conference 2023 on September 15th at 9:00 AM.
    The event will be held at the Grand Convention Center, 789 Convention Blvd, 
    San Francisco. This year's theme focuses on AI advancements and their
    applications across various industries. Network with industry leaders and
    gain insights into cutting-edge technologies.
  `;
  
  const reviewText = `
    I recently purchased the UltraBook Pro X1 laptop and have been using it for two weeks.
    Overall, I'd rate it 4 out of 5 stars. 
    
    What I love: The battery life is exceptional, lasting over 10 hours. The display is 
    crisp with vibrant colors. It's also surprisingly lightweight for a laptop with these specs.
    
    The downsides: The keyboard doesn't have great tactile feedback. It runs hot when using 
    resource-intensive applications. The price is also on the higher side compared to similar models.
    
    In summary, the UltraBook Pro X1 is a high-quality laptop with great battery life and display, 
    but has minor issues with the keyboard and thermal management.
  `;
  
  // Extract information using each schema
  console.log("Extracting contact information...");
  const contactInfo = await extractInformation(contactText, contactSchema);
  console.log(JSON.stringify(contactInfo, null, 2));
  
  console.log("\nExtracting event information...");
  const eventInfo = await extractInformation(eventText, eventSchema);
  console.log(JSON.stringify(eventInfo, null, 2));
  
  console.log("\nExtracting product review information...");
  const reviewInfo = await extractInformation(reviewText, reviewSchema);
  console.log(JSON.stringify(reviewInfo, null, 2));
}

runExtractionExample();
```

```python
# Python implementation
from langchain.chat_models import ChatOpenAI
from langchain.output_parsers import PydanticOutputParser
from langchain.prompts import PromptTemplate
from pydantic import BaseModel, Field
from typing import List, Optional

class ContactInfo(BaseModel):
    name: str = Field(description="The person's full name")
    email: Optional[str] = Field(None, description="The person's email address, if available")
    phone: Optional[str] = Field(None, description="The person's phone number, if available")
    address: Optional[str] = Field(None, description="The person's physical address, if available")

class EventInfo(BaseModel):
    event_name: str = Field(description="The name of the event")
    date: str = Field(description="The date when the event occurs")
    time: Optional[str] = Field(None, description="The time when the event starts")
    location: str = Field(description="Where the event takes place")
    description: str = Field(description="A brief description of the event")

class ProductReview(BaseModel):
    product_name: str = Field(description="The name of the product being reviewed")
    rating: int = Field(description="The numerical rating (1-5)")
    pros: List[str] = Field(description="List of positive aspects")
    cons: List[str] = Field(description="List of negative aspects")
    summary: str = Field(description="A one-sentence summary of the review")

def extract_information(text, schema_class):
    # 1. Initialize the language model
    model = ChatOpenAI(
        temperature=0,
        model_name="gpt-4"  # Using GPT-4 for better extraction accuracy
    )
    
    # 2. Create a parser based on the provided schema
    parser = PydanticOutputParser(pydantic_object=schema_class)
    
    # 3. Create a prompt template
    format_instructions = parser.get_format_instructions()
    
    prompt = PromptTemplate(
        template="Extract the following information from the text below.\n\n{format_instructions}\n\nText: {text}",
        input_variables=["text"],
        partial_variables={"format_instructions": format_instructions}
    )
    
    # 4. Format the prompt with the input text
    input_text = prompt.format(text=text)
    
    # 5. Call the model and parse the response
    response = model.predict(input_text)
    return parser.parse(response)

# Example usage
def run_extraction_example():
    # Example texts
    contact_text = """
    John Smith is our new sales representative. You can reach him at 
    john.smith@example.com or call his office at (555) 123-4567. 
    His office is located at 123 Business Ave, Suite 456, Chicago, IL 60601.
    """
    
    event_text = """
    Join us for the Annual Tech Conference 2023 on September 15th at 9:00 AM.
    The event will be held at the Grand Convention Center, 789 Convention Blvd, 
    San Francisco. This year's theme focuses on AI advancements and their
    applications across various industries. Network with industry leaders and
    gain insights into cutting-edge technologies.
    """
    
    review_text = """
    I recently purchased the UltraBook Pro X1 laptop and have been using it for two weeks.
    Overall, I'd rate it 4 out of 5 stars. 
    
    What I love: The battery life is exceptional, lasting over 10 hours. The display is 
    crisp with vibrant colors. It's also surprisingly lightweight for a laptop with these specs.
    
    The downsides: The keyboard doesn't have great tactile feedback. It runs hot when using 
    resource-intensive applications. The price is also on the higher side compared to similar models.
    
    In summary, the UltraBook Pro X1 is a high-quality laptop with great battery life and display, 
    but has minor issues with the keyboard and thermal management.
    """
    
    # Extract information using each schema
    print("Extracting contact information...")
    contact_info = extract_information(contact_text, ContactInfo)
    print(contact_info.json(indent=2))
    
    print("\nExtracting event information...")
    event_info = extract_information(event_text, EventInfo)
    print(event_info.json(indent=2))
    
    print("\nExtracting product review information...")
    review_info = extract_information(review_text, ProductReview)
    print(review_info.json(indent=2))

if __name__ == "__main__":
    run_extraction_example()
```

## How It Works

1. **Schema Definition**: The example defines structured schemas for different types of information:
   - Contact information (name, email, phone, address)
   - Event details (name, date, time, location, description)
   - Product reviews (product name, rating, pros, cons, summary)

2. **Parser Creation**: A structured output parser is created from each schema to:
   - Generate formatting instructions for the LLM
   - Parse the LLM's output into the desired structure

3. **Prompt Engineering**: A prompt is created that includes:
   - Clear instructions for the extraction task
   - Formatting requirements from the parser
   - The text to analyze

4. **Model Invocation**: A language model (GPT-4) processes the prompt and returns structured output.

5. **Response Parsing**: The parser converts the text response into the structured format defined by the schema.

## Applications

- **Document Processing**: Extract key information from legal documents, reports, or forms
- **Email Analysis**: Pull appointments, contact information, or action items from emails
- **Review Mining**: Analyze product or service reviews for specific attributes
- **Resume Parsing**: Extract candidate information, skills, and experience from resumes
- **News Analysis**: Pull entities, facts, and relationships from news articles

## Customization Options

- **Schema Complexity**: Add nested structures or more detailed fields
- **Validation Rules**: Include field validation (e.g., email format, numeric ranges)
- **Multiple Extractions**: Process entire datasets of documents
- **Custom Instructions**: Refine prompts for specific extraction needs

## Extension Ideas

- Add pre-processing for noisy text
- Implement batch processing for multiple documents
- Integrate with document OCR for physical document extraction
- Create a feedback loop to improve extraction accuracy over time