# Synalinks Vibe Coding Template

A template project for getting started with vibe coding using Synalinks and Cursor IDE. This template provides everything you need to start building AI-powered applications using natural language programming.

## What is Vibe Coding?

Vibe coding is a development paradigm where you:
- Describe features in natural language
- Let AI handle implementation details
- Focus on high-level design and problem-solving
- Iterate rapidly with AI assistance

## Features

- 🧠 **AI-First Development** - Optimized for Cursor IDE and AI pair programming
- 📝 **Prompt Templates** - Ready-to-use templates for common development tasks
- 🏗️ **Project Structure** - Organized for effective AI collaboration
- 🔧 **Development Tools** - Configured for optimal AI-assisted workflow
- 📚 **Documentation** - Comprehensive guides for vibe coding practices
- 🤖 **dotai Integration** - Structured AI configuration and development rules

## Quick Start

1. **Clone the Template**
```bash
git clone https://github.com/SynaLinks/synalinks-vibe.git my-project
cd my-project
```

2. **Set Up Environment**
```bash
pip install uv
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
uv pip install -r requirements.txt
```

3. **Open in Cursor IDE**
- Download Cursor from https://cursor.sh
- Open the project
- Enable AI features

4. **Start Vibe Coding**
- Check `.ai/lib/prompts.md` for ready-to-use prompts
- Use `.ai/codex/codex.md` for project patterns
- Follow the development workflow in docs

## Using the .ai Directory

The `.ai` directory is your command center for AI-assisted development:

### 1. Codex (`/.ai/codex/`)
```markdown
# Example: Using Codex for New Features
1. Check codex.md for existing patterns
2. Follow architectural guidelines
3. Update with new patterns you discover
```

### 2. Rules (`/.ai/rules/`)
```markdown
# Example: Python Development Rules
- Use type hints for all functions
- Write PEP 257 docstrings
- Follow project-specific conventions
```

### 3. Library (`/.ai/lib/`)
```markdown
# Example: Common Prompts
"Create a FastAPI endpoint that handles user authentication with JWT tokens"
"Generate a data model with validation for user profiles"
```

### 4. Plugins (`/.ai/plugins/`)
```json
{
  "cursor": {
    "aiCommands": {
      "test": "Generate pytest test for current file",
      "doc": "Add docstrings to current file"
    }
  }
}
```

### 5. Session & Status
- Track AI interactions in `session/`
- Monitor progress in `status/`
- Review and improve effectiveness

### 6. Blueprints
- Check `blueprints/` for full project templates
- Use for common application patterns
- Contribute your own blueprints

## Project Structure

```
my-project/
├── .ai/                # AI configuration and templates
│   ├── codex/         # Project-wide AI context
│   ├── rules/         # Development rules and standards
│   ├── lib/           # Reusable prompts and patterns
│   ├── plugins/       # IDE and tool configurations
│   ├── session/       # AI session management
│   ├── status/        # AI task tracking
│   └── blueprints/    # Larger templates
├── src/               # Source code
│   ├── models/        # Data models
│   ├── programs/      # Synalinks programs
│   └── utils/         # Utilities
├── tests/             # Test files
├── docs/              # Documentation
│   ├── sop/          # Standard Operating Procedures
│   ├── api/          # API Documentation
│   └── examples/      # Usage examples
└── app/               # Application code
```

## Development Workflow

1. **Describe Features**
   - Write clear natural language descriptions
   - Reference `.ai/lib/prompts.md` for examples
   - Follow patterns in `.ai/codex/codex.md`

2. **Generate & Refine**
   - Let AI generate initial implementation
   - Follow rules in `.ai/rules/python-dev.md`
   - Use IDE features from `.ai/plugins/cursor.md`

3. **Test & Document**
   - Write tests following project standards
   - Document using templates from `.ai/lib/prompts.md`
   - Keep documentation up to date

## Documentation

- [Getting Started](docs/quickstart.md) - Quick start guide
- [Vibe Coding Patterns](docs/sop/02-vibe-coding-patterns.md) - Essential patterns
- [Project Setup](docs/sop/01-project-setup.md) - Detailed setup guide
- [Examples](docs/examples/index.md) - Real-world examples

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Resources

- [Synalinks Documentation](https://docs.synalinks.dev)
- [Cursor IDE](https://cursor.sh)
- [Community Forum](https://community.synalinks.dev)
- [Discord Community](https://discord.gg/synalinks)
