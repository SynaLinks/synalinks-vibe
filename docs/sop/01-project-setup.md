# Project Setup for AI-First Development

This SOP covers setting up a new Synalinks project optimized for vibe coding and AI-assisted development.

## Prerequisites

Before you begin, ensure you have:
- Python 3.9+ installed
- Cursor IDE installed (recommended for optimal AI pair programming)
- Basic understanding of AI-assisted development
- GitHub Copilot access (optional but recommended)

## Development Environment

### 1. IDE Setup

#### Cursor IDE (Recommended)
- Download from https://cursor.sh
- Enable AI features
- Configure keyboard shortcuts for AI commands
- Set up project-specific AI contexts

#### VS Code Alternative
- Install Python extension
- Install GitHub Copilot
- Configure AI-assisted coding features

## Project Structure

A Synalinks project optimized for vibe coding:

```
my_project/
├── README.md           # Project documentation
├── requirements.txt    # Project dependencies
├── .env               # Environment variables
├── .gitignore         # Git ignore file
├── .ai/               # AI configuration
│   ├── prompts/       # Reusable AI prompts
│   ├── contexts/      # Project context files
│   └── templates/     # Code generation templates
├── src/               # Source code
│   ├── __init__.py
│   ├── models/        # Data models
│   ├── programs/      # Synalinks programs
│   └── utils/         # Utilities
├── tests/             # Test files
└── app/               # Application code
    └── api.py         # API endpoints
```

## Setup Steps

### 1. Create Project Structure

```bash
# Create main directories
mkdir -p my_project/{src/{models,programs,utils},tests,app,.ai/{prompts,contexts,templates}}
cd my_project
```

### 2. Initialize AI Environment

Create `.ai/contexts/project.md`:
```markdown
# Project Context

This is a Synalinks project that:
- [Describe main functionality]
- [List key features]
- [Note any constraints]

## Architecture
- [Describe system architecture]
- [List main components]

## Requirements
- [List key requirements]
```

### 3. Set Up Virtual Environment

Using UV (recommended):
```bash
pip install uv
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
```

### 4. Install Dependencies

```bash
uv pip install synalinks fastapi uvicorn pydantic
uv pip freeze > requirements.txt
```

### 5. Configure AI Templates

Create `.ai/templates/program.md`:
```markdown
# Program Template

Description:
[Program description]

Inputs:
- [Input description]

Outputs:
- [Output description]

Requirements:
- [List requirements]

Example Usage:
[Example code]
```

### 6. Create README.md

```markdown
# [Project Name]

## Overview
[Project description]

## AI-First Development
This project uses vibe coding principles:
- High-level problem description
- AI-assisted implementation
- Rapid prototyping
- Continuous refinement

## Setup
1. Install dependencies
2. Configure AI environment
3. Start development

## Development Workflow
1. Describe features in natural language
2. Use AI to generate implementation
3. Review and refine code
4. Add tests and documentation
```

### 7. Configure .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
.env
.venv/

# IDE
.vscode/
.idea/

# Project specific
/data/
/models/

# AI artifacts
.ai/generated/
```

## Development Workflow

1. **Feature Planning**
   - Write natural language description
   - Define inputs and outputs
   - List requirements and constraints

2. **Implementation**
   - Use AI to generate code
   - Review and refine
   - Add tests and documentation

3. **Iteration**
   - Gather feedback
   - Refine implementation
   - Optimize performance

## Best Practices

1. **AI Communication**
   - Be clear and specific
   - Provide context
   - Use consistent terminology

2. **Code Organization**
   - Keep AI prompts versioned
   - Document AI decisions
   - Maintain context files

3. **Quality Assurance**
   - Review AI-generated code
   - Add comprehensive tests
   - Document limitations

## Next Steps

1. Review [Vibe Coding Patterns](./02-vibe-coding-patterns.md)
2. Set up your first program using AI assistance
3. Explore advanced AI interactions

For more details on working with AI, see the [Language Models](./04-language-models.md) section. 