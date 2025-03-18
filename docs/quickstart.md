# Quickstart

This guide will help you get started with Synalinks quickly. We'll cover installation, creating your first program, and running it.

## Install

Install Synalinks using pip:

```bash
pip install synalinks
```

For better dependency management, we recommend using [uv](https://github.com/astral-sh/uv):

```bash
uv pip install synalinks
```

## Creating Your First Program

Let's create a simple question-answering program using Synalinks. First, we'll define our data models, then create the program, and finally run it.

### Set Up Your Environment

Create a new file called `simple_qa.py`:

```python
import asyncio
import synalinks
from synalinks import DataModel, Field, Program
from typing import Optional

# Set up your API keys (or use environment variables)
# os.environ["OPENAI_API_KEY"] = "your-api-key"
```

### Define Data Models

Define the input and output data models:

```python
class Question(DataModel):
    """Input model for the question answering system."""
    
    question: str = Field(
        description="The question to be answered"
    )
    context: Optional[str] = Field(
        None, 
        description="Optional context information"
    )

class Answer(DataModel):
    """Output model for the question answering system."""
    
    answer: str = Field(
        description="The direct answer to the question"
    )
    confidence: float = Field(
        description="Confidence score between 0 and 1"
    )
```

### Create a Program

Now let's create a simple program using the functional API:

```python
@synalinks.program
async def simple_qa(question: Question) -> Answer:
    """A simple question answering program."""
    
    # Create a language model instance
    language_model = synalinks.language_models.OpenAILanguageModel(
        model="gpt-3.5-turbo",
        temperature=0.3
    )
    
    # Build the prompt
    prompt = f"""
    Question: {question.question}
    """
    
    if question.context:
        prompt += f"\nContext: {question.context}"
    
    prompt += "\nPlease provide a direct answer to this question."
    
    # Generate response
    response = await language_model.generate(prompt)
    
    # Analyze confidence (in a real app, you'd have a more sophisticated approach)
    confidence = 0.8 if len(response) > 50 else 0.6
    
    # Return structured output
    return Answer(
        answer=response.strip(),
        confidence=confidence
    )
```

### Run the Program

Add code to run the program:

```python
async def main():
    # Create an instance of our input model
    question = Question(
        question="What is the capital of France?",
        context="France is a country in Western Europe."
    )
    
    # Run the program
    result = await simple_qa(question)
    
    # Print the result
    print(f"Question: {question.question}")
    print(f"Answer: {result.answer}")
    print(f"Confidence: {result.confidence}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Run the Script

```bash
python simple_qa.py
```

## Using the Class-based API

For more complex programs, you might prefer the class-based API. Here's how to rewrite the same program:

```python
class QuestionAnswerer(Program):
    """A class-based question answering program."""
    
    def __init__(self, model_name="gpt-3.5-turbo", temperature=0.3):
        """Initialize the question answerer."""
        super().__init__()
        
        self.language_model = synalinks.language_models.OpenAILanguageModel(
            model=model_name,
            temperature=temperature
        )
    
    async def __call__(self, question: Question) -> Answer:
        """Process a question and generate an answer."""
        
        # Build the prompt
        prompt = f"""
        Question: {question.question}
        """
        
        if question.context:
            prompt += f"\nContext: {question.context}"
        
        prompt += "\nPlease provide a direct answer to this question."
        
        # Generate response
        response = await self.language_model.generate(prompt)
        
        # Analyze confidence
        confidence = 0.8 if len(response) > 50 else 0.6
        
        # Return structured output
        return Answer(
            answer=response.strip(),
            confidence=confidence
        )
```

Then use it like this:

```python
async def main():
    # Create the program
    qa_program = QuestionAnswerer()
    
    # Create an instance of our input model
    question = Question(
        question="What is the capital of France?",
        context="France is a country in Western Europe."
    )
    
    # Run the program
    result = await qa_program(question)
    
    # Print the result
    print(f"Question: {question.question}")
    print(f"Answer: {result.answer}")
    print(f"Confidence: {result.confidence}")
```

## Using the Sequential API

For pipeline-based programs, the Sequential API is perfect:

```python
from synalinks import Sequential, Generator, Input

async def main():
    # Define a language model
    language_model = synalinks.language_models.OpenAILanguageModel(
        model="gpt-3.5-turbo",
        temperature=0.3
    )
    
    # Create a sequential program
    qa_pipeline = Sequential([
        Input(data_model=Question),
        Generator(
            data_model=Answer,
            language_model=language_model,
            system_message="You are a helpful assistant that answers questions directly and concisely."
        )
    ])
    
    # Create an instance of our input model
    question = Question(
        question="What is the capital of France?",
        context="France is a country in Western Europe."
    )
    
    # Run the program
    result = await qa_pipeline(question)
    
    # Print the result
    print(f"Question: {question.question}")
    print(f"Answer: {result.answer}")
    print(f"Confidence: {result.confidence}")
```

## Saving and Loading Programs

You can save and load your Synalinks programs:

```python
# Save the program
await qa_program.save("qa_program.json")

# Load the program later
loaded_program = await synalinks.Program.load("qa_program.json")
```

## Next Steps

Now that you've created your first Synalinks program, you can:

1. Explore more complex program structures
2. Try different language models
3. Add evaluation metrics
4. Implement training for your programs
5. Deploy your program as an API

Check out the other documentation sections for more advanced features and examples. 