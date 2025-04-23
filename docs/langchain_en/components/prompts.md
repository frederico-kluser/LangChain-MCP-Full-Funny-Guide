# Prompts

Prompts are the instructions or queries provided to language models. Effective prompting is critical for getting high-quality outputs from LLMs.

## Prompt Components

A well-structured prompt typically includes:

1. **Instructions**: Clear directions for what the model should do
2. **Context**: Background information needed for the task
3. **Input data**: Specific information to process
4. **Output format**: Expected structure of the response

## Prompt Templates in LangChain

LangChain provides prompt templates to make working with prompts more structured:

```javascript
// JavaScript example
import { PromptTemplate } from "langchain/prompts";

const template = "Summarize the following text in {word_count} words: {text}";
const prompt = PromptTemplate.fromTemplate(template);

const formattedPrompt = await prompt.format({
  word_count: "100",
  text: "Long text to summarize...",
});
```

```python
# Python example
from langchain.prompts import PromptTemplate

template = "Summarize the following text in {word_count} words: {text}"
prompt = PromptTemplate(
    input_variables=["word_count", "text"],
    template=template,
)

formatted_prompt = prompt.format(word_count="100", text="Long text to summarize...")
```

## Advanced Prompt Techniques

### Few-shot Examples
Providing examples of expected input/output pairs:

```
Classify the sentiment (positive, negative, neutral):

Text: The food was delicious.
Sentiment: positive

Text: The service was slow.
Sentiment: negative

Text: {input_text}
Sentiment:
```

### Chain-of-Thought Prompting
Guiding the model through a reasoning process:

```
Problem: If John has 5 apples and gives 2 to Mary, how many does he have left?
Thought process: John started with 5 apples. He gave 2 to Mary. 5 - 2 = 3.
Answer: 3 apples

Problem: {problem}
Thought process:
```

## Prompt Best Practices

1. **Be specific**: Clearly define what you want the model to do
2. **Provide context**: Give necessary background information
3. **Use examples**: Show the model the expected format
4. **Break down complex tasks**: Use step-by-step instructions
5. **Test and iterate**: Refine prompts based on outputs
6. **Consider role prompting**: Define a specific role for the model (e.g., "Act as an expert in...")