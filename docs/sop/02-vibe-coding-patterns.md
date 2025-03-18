# Vibe Coding Patterns

This SOP covers essential patterns and practices for effective vibe coding using Synalinks and AI assistance.

## Core Principles

Vibe coding emphasizes:
- High-level problem description over manual implementation
- AI-assisted code generation and refinement
- Rapid prototyping and iteration
- Natural language interaction with AI tools

## Effective Problem Description

### Pattern: Component Description
```python
# Instead of manually coding a data model, describe it to your AI assistant:
"""
I need a data model for a user profile with:
- Username (string, required)
- Email (string, required, must be valid email)
- Age (integer, optional, must be > 0)
- Interests (list of strings, optional)
"""
```

### Pattern: Functionality Description
```python
# Instead of writing implementation details, describe the desired behavior:
"""
Create an async function that:
1. Takes a user profile as input
2. Validates the email format
3. Ensures age is positive if provided
4. Returns a formatted profile summary
"""
```

## AI Interaction Patterns

### 1. Iterative Refinement
```python
# Start with high-level description:
"""
Build a REST API endpoint for user registration
"""

# Refine with specific requirements:
"""
Add input validation for:
- Password strength
- Unique username
- Age restrictions
"""
```

### 2. Context Enhancement
```python
# Provide relevant context for better AI understanding:
"""
This is part of a larger user authentication system that:
- Uses JWT for session management
- Integrates with PostgreSQL
- Requires email verification
"""
```

## Quality Assurance

### Pattern: Test Description
```python
"""
Create tests that verify:
1. Email validation works correctly
2. Age validation handles edge cases
3. Profile formatting is consistent
"""
```

### Pattern: Error Handling
```python
"""
Add error handling for:
- Invalid email formats
- Network timeouts
- Database connection issues
"""
```

## Best Practices

1. **Clear Intent**
   - Be specific about desired outcomes
   - Provide context when needed
   - Specify constraints upfront

2. **Iterative Development**
   - Start with basic functionality
   - Add features incrementally
   - Validate each iteration

3. **Documentation**
   - Document AI-generated code
   - Explain key decisions
   - Note any limitations

4. **Testing**
   - Describe test cases in natural language
   - Verify edge cases
   - Test integration points

## Example Workflow

1. **Initial Description**
```python
"""
Create a user authentication system with:
- Registration endpoint
- Login endpoint
- Password reset functionality
"""
```

2. **Feature Refinement**
```python
"""
Add to the registration endpoint:
- Email verification
- Password strength requirements
- Username availability check
"""
```

3. **Implementation Review**
```python
"""
Review the generated code for:
- Security best practices
- Error handling
- Performance optimization
"""
```

## Common Pitfalls

1. **Vague Descriptions**
   - Too general requirements
   - Missing context
   - Unclear constraints

2. **Over-specification**
   - Too detailed implementation
   - Limiting AI creativity
   - Rigid requirements

3. **Insufficient Testing**
   - Missing edge cases
   - Incomplete validation
   - Poor error handling

## Next Steps

After mastering these patterns:
1. Explore advanced AI interactions
2. Develop custom prompt templates
3. Create reusable code patterns

Follow the [Language Models SOP](./04-language-models.md) for details on configuring and optimizing AI interactions. 