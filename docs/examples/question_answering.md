# Building a Question Answering System with Synalinks

This guide demonstrates how to build a complete question answering system using Synalinks. The system will take questions as input and return detailed answers with confidence scores and citations.

## Overview

This question answering system will:

1. Accept a question and optional domain context
2. Retrieve relevant information to answer the question
3. Process and analyze the information
4. Generate a comprehensive answer with citations
5. Return the results with a confidence score

## Step 1: Define Data Models

First, let's define the data models for our question answering system:

```python
# models.py
from typing import List, Dict, Any, Optional
from synalinks import DataModel, Field

class Question(DataModel):
    """Input model for the question answering system."""
    
    question: str = Field(
        description="The question to be answered"
    )
    domain: Optional[str] = Field(
        None, 
        description="Optional domain to provide context (e.g., 'science', 'history', 'technology')"
    )

class Answer(DataModel):
    """Output model for the question answering system."""
    
    answer: str = Field(
        description="The direct answer to the question"
    )
    explanation: str = Field(
        description="Detailed explanation supporting the answer"
    )
    confidence: float = Field(
        description="Confidence score between 0 and 1"
    )
    citations: List[Dict[str, Any]] = Field(
        description="List of citations and sources supporting the answer"
    )
```

## Step 2: Create a Basic Program

Next, let's create a simple version of our question answering program:

```python
# program.py
import synalinks
from synalinks import Program
from models import Question, Answer

class QuestionAnswerer(Program):
    """A program that answers questions with detailed explanations and citations."""
    
    def __init__(self, language_model=None):
        """Initialize the question answerer."""
        super().__init__()
        
        # Use provided language model or default to OpenAI
        if language_model is None:
            from synalinks.language_models import OpenAILanguageModel
            language_model = OpenAILanguageModel(
                model="gpt-4",
                temperature=0.2
            )
        
        self.language_model = language_model
    
    async def call(self, inputs, training=False):
        """
        Process a question and generate an answer.
        
        Args:
            question: The question to answer
            
        Returns:
            An answer with explanation, confidence, and citations
        """
        # Construct a prompt for the language model
        prompt = self._build_prompt(question)
        
        # Get response from language model
        response = await self.language_model.generate(prompt)
        
        # Parse the response into our Answer model
        parsed_answer = self._parse_response(response)
        
        return parsed_answer
    
    def _build_prompt(self, question: Question) -> str:
        """Build a prompt for the language model based on the question."""
        domain_context = f" in the domain of {question.domain}" if question.domain else ""
        
        prompt = f"""
        Answer the following question{domain_context} with a detailed explanation and citations.
        
        Question: {question.question}
        
        Provide your answer in the following format:
        
        Answer: <concise direct answer>
        
        Explanation: <detailed explanation supporting the answer>
        
        Confidence: <a number between 0 and 1 indicating your confidence>
        
        Citations:
        1. <citation 1>
        2. <citation 2>
        ...
        """
        
        return prompt
    
    def _parse_response(self, response: str) -> Answer:
        """Parse the language model response into an Answer object."""
        # Simple parsing logic - in a real application, this would be more robust
        lines = response.strip().split("\n")
        
        answer_text = ""
        explanation_text = ""
        confidence_value = 0.7  # Default
        citations_list = []
        
        current_section = None
        citation_buffer = []
        
        for line in lines:
            line = line.strip()
            if not line:
                continue
                
            if line.startswith("Answer:"):
                current_section = "answer"
                answer_text = line.replace("Answer:", "").strip()
            elif line.startswith("Explanation:"):
                current_section = "explanation"
                explanation_text = line.replace("Explanation:", "").strip()
            elif line.startswith("Confidence:"):
                try:
                    confidence_value = float(line.replace("Confidence:", "").strip())
                except ValueError:
                    confidence_value = 0.7
            elif line.startswith("Citations:"):
                current_section = "citations"
            elif current_section == "explanation" and not line.startswith("Confidence:") and not line.startswith("Citations:"):
                explanation_text += " " + line
            elif current_section == "citations" and line[0].isdigit() and line[1] in [".", ")", ":"]:
                if citation_buffer:
                    citations_list.append({"text": " ".join(citation_buffer)})
                    citation_buffer = []
                citation_buffer.append(line[2:].strip())
            elif current_section == "citations":
                citation_buffer.append(line)
        
        # Add the last citation if any
        if citation_buffer:
            citations_list.append({"text": " ".join(citation_buffer)})
        
        # Ensure we have at least something in each field
        if not answer_text:
            answer_text = "Unable to determine a clear answer."
        
        if not explanation_text:
            explanation_text = "No explanation provided."
        
        # Create and return the Answer object
        return Answer(
            answer=answer_text,
            explanation=explanation_text,
            confidence=confidence_value,
            citations=citations_list
        )
```

## Step 3: Add Retrieval Capabilities

To make our QA system more powerful, let's add information retrieval:

```python
# retrieval.py
import asyncio
from typing import List, Dict, Any

class SimpleRetriever:
    """A simple retriever that simulates searching for information."""
    
    async def search(self, query: str, domain: str = None) -> List[Dict[str, Any]]:
        """
        Search for information relevant to the query.
        
        In a real implementation, this would connect to a database, search engine,
        or other information source.
        
        Args:
            query: The search query
            domain: Optional domain to narrow the search
            
        Returns:
            A list of retrieved documents
        """
        # Simulate some network latency
        await asyncio.sleep(0.5)
        
        # In a real implementation, this would perform an actual search
        # For this example, we'll return mock data
        
        if "quantum" in query.lower() and (domain is None or domain.lower() == "physics"):
            return [
                {
                    "title": "Quantum Mechanics: An Overview",
                    "content": "Quantum mechanics is a fundamental theory in physics that provides a description of the physical properties of nature at the scale of atoms and subatomic particles.",
                    "source": "Introduction to Quantum Physics, 2023",
                    "url": "https://example.com/quantum-physics"
                },
                {
                    "title": "The Double-Slit Experiment",
                    "content": "The double-slit experiment demonstrates the wave-particle duality of quantum objects. When particles like electrons are sent through a double-slit apparatus, they create an interference pattern on the detector screen.",
                    "source": "Journal of Physics Research, Vol. 45, 2022",
                    "url": "https://example.com/double-slit"
                }
            ]
        elif "climate" in query.lower() and (domain is None or domain.lower() == "environment"):
            return [
                {
                    "title": "Climate Change: Global Impact",
                    "content": "Climate change refers to long-term shifts in temperatures and weather patterns. These shifts may be natural, but since the 1800s, human activities have been the main driver of climate change.",
                    "source": "Environmental Science Journal, 2023",
                    "url": "https://example.com/climate-change"
                }
            ]
        else:
            # Generic fallback for other queries
            return [
                {
                    "title": "General Knowledge Database",
                    "content": "This is a placeholder for retrieved content about: " + query,
                    "source": "Knowledge Base, 2023",
                    "url": "https://example.com/knowledge"
                }
            ]
```

## Step 4: Enhance the Program with Retrieval

Now let's update our program to use the retriever:

```python
# enhanced_program.py
from retrieval import SimpleRetriever
from models import Question, Answer
import synalinks
from synalinks import Program

class EnhancedQuestionAnswerer(Program):
    """An enhanced question answerer that uses retrieval for better answers."""
    
    def __init__(self, language_model=None, retriever=None):
        """Initialize the enhanced question answerer."""
        super().__init__()
        
        # Use provided language model or default to OpenAI
        if language_model is None:
            from synalinks.language_models import OpenAILanguageModel
            language_model = OpenAILanguageModel(
                model="gpt-4",
                temperature=0.2
            )
        
        self.language_model = language_model
        
        # Use provided retriever or create a simple one
        self.retriever = retriever if retriever is not None else SimpleRetriever()
    
    async def call(self, inputs, training=False):
        """
        Process a question and generate an answer using retrieval.
        
        Args:
            question: The question to answer
            
        Returns:
            An answer with explanation, confidence, and citations
        """
        # Retrieve relevant information
        retrieved_docs = await self.retriever.search(
            query=question.question,
            domain=question.domain
        )
        
        # Construct a prompt that includes the retrieved information
        prompt = self._build_prompt(question, retrieved_docs)
        
        # Get response from language model
        response = await self.language_model.generate(prompt)
        
        # Parse the response into our Answer model
        parsed_answer = self._parse_response(response, retrieved_docs)
        
        return parsed_answer
    
    def _build_prompt(self, question: Question, retrieved_docs: List[Dict[str, Any]]) -> str:
        """Build a prompt that includes retrieved information."""
        domain_context = f" in the domain of {question.domain}" if question.domain else ""
        
        # Build context from retrieved documents
        context = ""
        for i, doc in enumerate(retrieved_docs, 1):
            context += f"\nDocument {i}:\nTitle: {doc['title']}\nContent: {doc['content']}\nSource: {doc['source']}\n"
        
        prompt = f"""
        You are answering the following question{domain_context}:
        
        Question: {question.question}
        
        Here is some information that may help you answer:
        {context}
        
        Based on the information provided and your knowledge, please answer the question.
        
        Provide your answer in the following format:
        
        Answer: <concise direct answer>
        
        Explanation: <detailed explanation supporting the answer>
        
        Confidence: <a number between 0 and 1 indicating your confidence, based on the quality and relevance of the retrieved information>
        
        Citations:
        1. <citation 1 - include the source and title from the retrieved documents>
        2. <citation 2>
        ...
        
        Note: Only cite the provided documents if they are relevant. If they are not relevant, you may rely on your own knowledge but assign a lower confidence score.
        """
        
        return prompt
    
    def _parse_response(self, response: str, retrieved_docs: List[Dict[str, Any]]) -> Answer:
        """Parse the language model response into an Answer object, enhancing citations with URLs."""
        # Similar to the previous parsing logic, but enhanced to include URLs from retrieved docs
        lines = response.strip().split("\n")
        
        answer_text = ""
        explanation_text = ""
        confidence_value = 0.7  # Default
        citations_list = []
        
        current_section = None
        citation_buffer = []
        
        for line in lines:
            line = line.strip()
            if not line:
                continue
                
            if line.startswith("Answer:"):
                current_section = "answer"
                answer_text = line.replace("Answer:", "").strip()
            elif line.startswith("Explanation:"):
                current_section = "explanation"
                explanation_text = line.replace("Explanation:", "").strip()
            elif line.startswith("Confidence:"):
                try:
                    confidence_value = float(line.replace("Confidence:", "").strip())
                except ValueError:
                    confidence_value = 0.7
            elif line.startswith("Citations:"):
                current_section = "citations"
            elif current_section == "explanation" and not line.startswith("Confidence:") and not line.startswith("Citations:"):
                explanation_text += " " + line
            elif current_section == "citations" and line[0].isdigit() and line[1] in [".", ")", ":"]:
                if citation_buffer:
                    # Create a citation with text
                    citation = {"text": " ".join(citation_buffer)}
                    
                    # Try to match with retrieved docs to add URL
                    for doc in retrieved_docs:
                        if doc["source"] in " ".join(citation_buffer) or doc["title"] in " ".join(citation_buffer):
                            citation["url"] = doc["url"]
                            break
                    
                    citations_list.append(citation)
                    citation_buffer = []
                citation_buffer.append(line[2:].strip())
            elif current_section == "citations":
                citation_buffer.append(line)
        
        # Add the last citation if any
        if citation_buffer:
            citation = {"text": " ".join(citation_buffer)}
            
            # Try to match with retrieved docs to add URL
            for doc in retrieved_docs:
                if doc["source"] in " ".join(citation_buffer) or doc["title"] in " ".join(citation_buffer):
                    citation["url"] = doc["url"]
                    break
            
            citations_list.append(citation)
        
        # Ensure we have at least something in each field
        if not answer_text:
            answer_text = "Unable to determine a clear answer."
        
        if not explanation_text:
            explanation_text = "No explanation provided."
        
        # Create and return the Answer object
        return Answer(
            answer=answer_text,
            explanation=explanation_text,
            confidence=confidence_value,
            citations=citations_list
        )
```

## Step 5: Create a Main Script

Now, let's create a main script to run our question answering system:

```python
# main.py
import asyncio
import json
from enhanced_program import EnhancedQuestionAnswerer
from models import Question

async def main():
    # Initialize our question answerer
    qa_system = EnhancedQuestionAnswerer()
    
    # Example questions to answer
    questions = [
        Question(question="How does quantum entanglement work?", domain="physics"),
        Question(question="What are the main effects of climate change?", domain="environment"),
        Question(question="What is machine learning?")
    ]
    
    # Process each question
    for i, question in enumerate(questions, 1):
        print(f"\nProcessing question {i}: {question.question}")
        
        # Generate answer
        answer = await qa_system(question)
        
        # Print the results
        print("\n" + "="*50)
        print(f"Question: {question.question}")
        print(f"Domain: {question.domain or 'Not specified'}")
        print("\nAnswer:", answer.answer)
        print("\nExplanation:", answer.explanation)
        print("\nConfidence:", answer.confidence)
        
        print("\nCitations:")
        for j, citation in enumerate(answer.citations, 1):
            print(f"{j}. {citation['text']}")
            if "url" in citation:
                print(f"   URL: {citation['url']}")
        
        print("="*50 + "\n")
        
        # Save the result to a JSON file
        with open(f"qa_result_{i}.json", "w") as f:
            # Convert answer to dict
            answer_dict = answer.dict()
            
            # Add question info
            result_dict = {
                "question": question.question,
                "domain": question.domain,
                "answer": answer_dict
            }
            
            # Save to file
            json.dump(result_dict, f, indent=2)
            print(f"Result saved to qa_result_{i}.json")

if __name__ == "__main__":
    asyncio.run(main())
```

## Step 6: Testing

Let's write tests for our question answering system:

```python
# test_qa_system.py
import pytest
from models import Question, Answer
from enhanced_program import EnhancedQuestionAnswerer
from unittest.mock import AsyncMock, patch

class MockLanguageModel:
    """Mock language model for testing."""
    
    async def generate(self, prompt):
        """Return a fixed response for testing."""
        return """
        Answer: This is a test answer.
        
        Explanation: This is a detailed explanation for testing purposes.
        
        Confidence: 0.85
        
        Citations:
        1. Test Citation 1, Source: Test Journal, 2023
        2. Test Citation 2, Source: Another Journal, 2022
        """

class MockRetriever:
    """Mock retriever for testing."""
    
    async def search(self, query, domain=None):
        """Return fixed search results for testing."""
        return [
            {
                "title": "Test Document",
                "content": "This is test content for " + query,
                "source": "Test Journal, 2023",
                "url": "https://example.com/test1"
            },
            {
                "title": "Another Test Document",
                "content": "More test content about " + query,
                "source": "Another Journal, 2022",
                "url": "https://example.com/test2"
            }
        ]

@pytest.fixture
def qa_system():
    """Create a QA system with mock components for testing."""
    language_model = MockLanguageModel()
    retriever = MockRetriever()
    return EnhancedQuestionAnswerer(language_model=language_model, retriever=retriever)

@pytest.mark.asyncio
async def test_qa_basic_functionality(qa_system):
    """Test that the QA system returns an answer for a basic question."""
    question = Question(question="What is a test?")
    
    answer = await qa_system(question)
    
    assert isinstance(answer, Answer)
    assert answer.answer == "This is a test answer."
    assert answer.explanation == "This is a detailed explanation for testing purposes."
    assert answer.confidence == 0.85
    assert len(answer.citations) == 2

@pytest.mark.asyncio
async def test_qa_with_domain(qa_system):
    """Test that the QA system handles domain-specific questions."""
    question = Question(question="What is a test?", domain="testing")
    
    answer = await qa_system(question)
    
    assert isinstance(answer, Answer)
    assert answer.answer == "This is a test answer."
    # Other assertions...

@pytest.mark.asyncio
async def test_retrieval_integration():
    """Test that the retriever is properly integrated and called."""
    # Create mocks
    mock_retriever = AsyncMock()
    mock_retriever.search.return_value = [
        {
            "title": "Test Document",
            "content": "Test content",
            "source": "Test Source",
            "url": "https://example.com/test"
        }
    ]
    
    mock_language_model = AsyncMock()
    mock_language_model.generate.return_value = "Answer: Test\n\nExplanation: Test\n\nConfidence: 0.8\n\nCitations:\n1. Test Citation"
    
    # Create QA system with mocks
    qa_system = EnhancedQuestionAnswerer(
        language_model=mock_language_model,
        retriever=mock_retriever
    )
    
    # Run with a test question
    question = Question(question="Test question", domain="test")
    answer = await qa_system(question)
    
    # Verify the retriever was called with the right arguments
    mock_retriever.search.assert_called_once_with(
        query="Test question", 
        domain="test"
    )
    
    # Verify the language model was called (we don't check the exact prompt)
    assert mock_language_model.generate.called
    
    # Verify we got a valid answer
    assert isinstance(answer, Answer)
    assert answer.answer == "Test"
    assert answer.confidence == 0.8
```

## Step 7: Add a Web API (Optional)

If you want to expose your question answering system as a web API, you can use FastAPI:

```python
# api.py
import uvicorn
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional, List, Dict, Any

from models import Question, Answer
from enhanced_program import EnhancedQuestionAnswerer

# Initialize FastAPI app
app = FastAPI(
    title="Question Answering API",
    description="API for answering questions with citations using Synalinks",
    version="1.0.0",
)

# Initialize QA system
qa_system = EnhancedQuestionAnswerer()

# API Models
class QuestionRequest(BaseModel):
    question: str
    domain: Optional[str] = None

class CitationResponse(BaseModel):
    text: str
    url: Optional[str] = None

class AnswerResponse(BaseModel):
    answer: str
    explanation: str
    confidence: float
    citations: List[CitationResponse]

@app.post("/answer", response_model=AnswerResponse)
async def answer_question(request: QuestionRequest):
    """
    Answer a question with detailed explanation and citations.
    """
    try:
        # Convert API request to internal Question model
        question = Question(
            question=request.question,
            domain=request.domain
        )
        
        # Process the question
        result = await qa_system(question)
        
        # Convert to response model
        response = AnswerResponse(
            answer=result.answer,
            explanation=result.explanation,
            confidence=result.confidence,
            citations=[CitationResponse(**citation) for citation in result.citations]
        )
        
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    """Health check endpoint."""
    return {"status": "healthy"}

# Run the API server
if __name__ == "__main__":
    uvicorn.run("api:app", host="0.0.0.0", port=8000, reload=True)
```

## Running the Example

To run this example:

1. Save all the files in a directory structure like this:

```
question_answering/
├── models.py
├── program.py
├── retrieval.py
├── enhanced_program.py
├── main.py
├── test_qa_system.py
├── api.py (optional)
└── requirements.txt
```

2. Create a `requirements.txt` file:

```
synalinks>=0.1.0
pytest>=7.0.0
pytest-asyncio>=0.21.0
fastapi>=0.95.0
uvicorn>=0.22.0
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Run the example:

```bash
python main.py
```

5. (Optional) Run the API:

```bash
python api.py
```

## Conclusion

This example demonstrates how to build a comprehensive question answering system using Synalinks. The system includes:

- Structured data models for questions and answers
- A retrieval component to find relevant information
- A robust program for processing questions and generating answers
- Tests to verify functionality
- An optional web API for serving the system

You can extend this example by:

- Implementing a real retrieval system (e.g., using vector databases)
- Adding more sophisticated parsing logic
- Implementing advanced caching for better performance
- Adding more validation and error handling
- Expanding the API with additional endpoints

For more examples, check out the other guides in this documentation. 