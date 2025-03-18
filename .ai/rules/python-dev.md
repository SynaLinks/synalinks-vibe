# Python Development Rules

## Code Style
- Use type hints for all functions and classes
- Follow PEP 8 style guide
- Use docstrings for all public functions and classes
- Organize imports: stdlib, third-party, local
- Use meaningful variable names

## Project Structure
- Use src-layout for Python packages
- Keep modules focused and single-responsibility
- Use __init__.py files appropriately
- Separate tests from implementation

## Testing
- Write tests first (TDD when possible)
- Use pytest for all testing
- Mock external dependencies
- Test edge cases and error conditions
- Maintain high test coverage

## Documentation
- Keep README.md up to date
- Document all public APIs
- Include usage examples
- Document configuration options
- Maintain CHANGELOG.md

## Dependencies
- Use UV for package management
- Pin dependency versions
- Minimize production dependencies
- Document dev dependencies separately

## Error Handling
- Use custom exception classes
- Provide meaningful error messages
- Handle edge cases gracefully
- Log errors appropriately

## Performance
- Profile before optimizing
- Use appropriate data structures
- Consider memory usage
- Cache expensive operations

## Security
- Never commit secrets
- Validate all inputs
- Use secure defaults
- Follow security best practices

## AI Integration
- Use descriptive prompts
- Validate AI-generated code
- Document AI decisions
- Keep AI context files updated 