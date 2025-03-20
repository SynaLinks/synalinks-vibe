# Synalinks API Reference

This section provides a comprehensive reference for the Synalinks API, allowing you to build applications that leverage the Synalinks framework for AI-driven programs.

## Core API Modules

The Synalinks API consists of several core modules:

- **Data Models** - Define structured input and output data for your AI programs
- **Program** - Core program classes for defining AI-driven logic
- **Language Models** - Interfaces to various LLM providers
- **Evaluation** - Tools for evaluating program performance
- **Training** - APIs for fine-tuning and improving programs
- **Utils** - Utility functions for working with Synalinks

## Data Models

Synalinks provides a powerful data modeling system that allows you to define the structure of your program's inputs and outputs.

```python
from synalinks import DataModel, Field
from typing import List, Dict, Any, Optional

class Question(DataModel):
    question: str = Field(description="The question to answer")
    context: Optional[str] = Field(None, description="Optional context for the question")
    
class Answer(DataModel):
    answer: str = Field(description="The direct answer to the question")
    confidence: float = Field(description="Confidence score between 0 and 1")
    sources: List[Dict[str, Any]] = Field(description="Reference sources")
```

## Program API

The Program API is the core of Synalinks, allowing you to define the logic of your AI-driven programs.

### Functional API

The simplest way to create a Synalinks program:

```python
import synalinks

# Define a simple program using the functional API
@synalinks.program
async def question_answerer(question: Question) -> Answer:
    """Answer questions with sources."""
    # Your program logic here
    result = Answer(
        answer="This is the answer to the question.",
        confidence=0.95,
        sources=[{"title": "Example Source", "url": "https://example.com"}]
    )
    return result
```

### Class-based API

For more complex programs, use the class-based API:

```python
import synalinks
from synalinks import Program

class QuestionAnswerer(Program):
    """A program that answers questions with sources."""
    
    async def __call__(self, question: Question) -> Answer:
        # Your program logic here
        result = Answer(
            answer="This is the answer to the question.",
            confidence=0.95,
            sources=[{"title": "Example Source", "url": "https://example.com"}]
        )
        return result
```

### Sequential API

For complex pipelines, use the Sequential API:

```python
import synalinks
from synalinks import Sequential

# Define a sequential program
retriever = synalinks.load("retriever")
analyzer = synalinks.load("analyzer")
generator = synalinks.load("generator")

qa_pipeline = Sequential([
    retriever,
    analyzer,
    generator
])
```

## Language Models

Synalinks provides integrations with various language model providers:

```python
from synalinks.language_models import OpenAILanguageModel, AnthropicLanguageModel

# OpenAI models
openai_model = OpenAILanguageModel(
    model="gpt-4",
    temperature=0.7,
    api_key="your-api-key"
)

# Anthropic models
anthropic_model = AnthropicLanguageModel(
    model="claude-3-opus-20240229",
    temperature=0.5,
    api_key="your-api-key"
)
```

## Evaluation

Synalinks includes tools for evaluating program performance:

```python
from synalinks.evaluation import Evaluator
from synalinks.metrics import AccuracyMetric, F1ScoreMetric

# Create an evaluator with multiple metrics
evaluator = Evaluator(
    metrics=[AccuracyMetric(), F1ScoreMetric()],
    model=qa_program
)

# Run evaluation on a dataset
results = await evaluator.evaluate(test_dataset)
print(f"Accuracy: {results['accuracy']:.2f}")
print(f"F1 Score: {results['f1_score']:.2f}")
```

## Training

Fine-tune and improve your programs:

```python
from synalinks.training import Trainer
from synalinks.optimize import AdamOptimizer

# Create a trainer
trainer = Trainer(
    model=qa_program,
    optimizer=AdamOptimizer(learning_rate=0.001),
    max_iterations=1000
)

# Train on a dataset
await trainer.train(training_dataset)

# Save the trained program
qa_program.save("models/qa_program_v1.json")
```

## REST API Integration

Synalinks programs can be easily exposed as REST APIs using FastAPI:

```python
import uvicorn
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

import synalinks

app = FastAPI(
    title="Synalinks API",
    description="API for a Synalinks program",
    version="1.0.0",
)

# Load your program
qa_program = synalinks.Program.load("models/qa_program.json")

# Define API models
class QuestionRequest(BaseModel):
    question: str
    context: str = None

class AnswerResponse(BaseModel):
    answer: str
    confidence: float
    sources: list

@app.post("/query", response_model=AnswerResponse)
async def query_endpoint(request: QuestionRequest):
    """Process a question and return an answer."""
    try:
        # Create input data model
        input_data = Question(
            question=request.question,
            context=request.context
        )
        
        # Process with the program
        result = await qa_program(input_data)
        
        # Return response
        return AnswerResponse(
            answer=result.answer,
            confidence=result.confidence,
            sources=result.sources
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## Advanced Features

Synalinks includes several advanced features:

- **Caching** - Cache model responses to improve performance
- **Streaming** - Stream responses for real-time updates
- **Observability** - Trace and log program execution
- **Customization** - Extend and customize any part of the framework

Refer to the detailed documentation for each module for more information.

## API Reference

For complete API details, including all classes, methods, and parameters, refer to the following sections:

- [Data Models API](data_models.md)
- [Program API](program.md)
- [Language Models API](language_models.md)
- [Evaluation API](evaluation.md)
- [Training API](training.md)
- [Utils API](utils.md) 