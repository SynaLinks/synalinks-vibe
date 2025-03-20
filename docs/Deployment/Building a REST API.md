# Building a REST API with Synalinks

This guide demonstrates how to create a REST API for your Synalinks programs using FastAPI.

## Prerequisites

- A trained Synalinks program
- FastAPI and Uvicorn installed:
  ```bash
  pip install fastapi uvicorn
  ```

## Basic API Structure

Here's a complete example of a simple API for a scientific question answering program:

```python
import asyncio
import os
from typing import Optional, Dict, Any, List

import uvicorn
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

import synalinks
from synalinks import DataModel, Field

# Define data models
class Question(DataModel):
    question: str = Field(description="The scientific question to answer")
    domain: Optional[str] = Field(None, description="The scientific domain (e.g., physics, biology)")

class Answer(DataModel):
    background: str = Field(description="Background information on the topic")
    reasoning: str = Field(description="Reasoning process for the answer")
    answer: str = Field(description="The direct answer to the question")
    confidence: float = Field(description="Confidence score between 0 and 1")
    citations: List[Dict[str, Any]] = Field(description="List of citation sources")

# Initialize FastAPI app
app = FastAPI(
    title="Scientific Q&A API",
    description="API for answering scientific questions with citations",
    version="1.0.0",
)

# Load the trained program
MODEL_PATH = os.environ.get("MODEL_PATH", "models/scientific_qa.json")

# We'll load the program when needed (lazy loading)
_program = None

async def get_program():
    """Lazily load the program to avoid startup delay."""
    global _program
    if _program is None:
        # Load the program
        _program = synalinks.Program.load(MODEL_PATH)
    return _program

# Define API models
class QueryRequest(BaseModel):
    question: str
    domain: Optional[str] = None

class QueryResponse(BaseModel):
    background: str
    reasoning: str
    answer: str
    confidence: float
    citations: List[Dict[str, Any]]

@app.get("/health")
async def health_check():
    """Health check endpoint."""
    return {"status": "healthy", "version": "1.0.0"}

@app.post("/query", response_model=QueryResponse)
async def query_endpoint(request: QueryRequest):
    """Process a scientific question and return an answer with citations."""
    try:
        # Get the program
        program = await get_program()
        
        # Create input data model
        input_data = Question(
            question=request.question,
            domain=request.domain,
        )
        
        # Process the query
        result = await program(input_data)
        
        # Convert to response model
        response = QueryResponse(
            background=result.background,
            reasoning=result.reasoning,
            answer=result.answer,
            confidence=result.confidence,
            citations=result.citations,
        )
        
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# Run the server if executed as a script
if __name__ == "__main__":
    # Get port from environment or use default
    port = int(os.environ.get("PORT", "8000"))
    
    # Run with uvicorn
    uvicorn.run("app:app", host="0.0.0.0", port=port, reload=True)
```

## Running the API

Run the API with:

```bash
python api.py
```

Or directly with Uvicorn:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

## Accessing the API Documentation

FastAPI automatically generates interactive API documentation:

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## Advanced Features

### Batch Processing

Add an endpoint for batch processing multiple questions:

```python
class BatchQueryRequest(BaseModel):
    questions: List[QueryRequest]
    
class BatchQueryResponse(BaseModel):
    results: List[QueryResponse]
    
@app.post("/batch", response_model=BatchQueryResponse)
async def batch_query_endpoint(request: BatchQueryRequest):
    """Process multiple scientific questions in batch."""
    try:
        # Get the program
        program = await get_program()
        
        # Process each question
        results = []
        for query in request.questions:
            # Create input data model
            input_data = Question(
                question=query.question,
                domain=query.domain,
            )
            
            # Process the query
            result = await program(input_data)
            
            # Convert to response model
            response = QueryResponse(
                background=result.background,
                reasoning=result.reasoning,
                answer=result.answer,
                confidence=result.confidence,
                citations=result.citations,
            )
            
            results.append(response)
        
        # Return batch response
        return BatchQueryResponse(results=results)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Authentication

Add API key authentication:

```python
from fastapi import Depends, HTTPException, Security
from fastapi.security.api_key import APIKeyHeader

API_KEY = os.environ.get("API_KEY", "default_api_key")
API_KEY_NAME = "X-API-Key"

api_key_header = APIKeyHeader(name=API_KEY_NAME, auto_error=False)

async def get_api_key(api_key: str = Security(api_key_header)):
    if api_key == API_KEY:
        return api_key
    raise HTTPException(
        status_code=403, detail="Could not validate API key"
    )

@app.post("/query", response_model=QueryResponse)
async def query_endpoint(
    request: QueryRequest, 
    api_key: str = Depends(get_api_key)
):
    # Process as before
    ...
```

### Rate Limiting

Use a rate limiting middleware like `slowapi`:

```bash
pip install slowapi
```

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.post("/query", response_model=QueryResponse)
@limiter.limit("5/minute")
async def query_endpoint(request: QueryRequest):
    # Process as before
    ...
```

## Deploying to Production

For production deployments:

1. **Environment Variables**: Use environment variables for all configuration
2. **Container**: Deploy using Docker
3. **ASGI Server**: Use Uvicorn with Gunicorn for multiple workers
4. **Reverse Proxy**: Place behind Nginx or similar
5. **HTTPS**: Always use HTTPS in production
6. **Monitoring**: Add logging and monitoring

### Docker Configuration

A sample Dockerfile:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Production Gunicorn Configuration

Run with multiple Uvicorn workers:

```bash
gunicorn app:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
```

## Tips for Synalinks API Development

1. **Lazy Loading**: Load models on-demand to reduce startup time
2. **Caching**: Cache responses when possible
3. **Streaming**: Use streaming responses for long-running inferences
4. **Error Handling**: Provide informative error messages
5. **Validation**: Use Pydantic's validation capabilities
6. **Documentation**: Fully document your API endpoints

## Example API Client

A Python client for the API:

```python
import aiohttp
import asyncio

async def query_scientific_api(question, domain=None, api_key=None):
    """Query the scientific Q&A API."""
    url = "http://localhost:8000/query"
    headers = {}
    if api_key:
        headers["X-API-Key"] = api_key
    
    payload = {"question": question}
    if domain:
        payload["domain"] = domain
    
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json=payload, headers=headers) as response:
            if response.status == 200:
                return await response.json()
            else:
                error_text = await response.text()
                raise Exception(f"API error: {response.status} - {error_text}")

async def main():
    result = await query_scientific_api(
        "What is quantum entanglement?", 
        domain="physics"
    )
    print(f"Answer: {result['answer']}")
    print(f"Confidence: {result['confidence']}")

if __name__ == "__main__":
    asyncio.run(main())
``` 