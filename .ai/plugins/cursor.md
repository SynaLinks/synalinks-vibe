# Cursor IDE Configuration

## Setup

### Extensions
- Python
- Pylance
- Jupyter
- GitLens
- GitHub Copilot

### Settings
```json
{
  "editor.formatOnSave": true,
  "python.analysis.typeCheckingMode": "strict",
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.testing.pytestEnabled": true
}
```

## AI Features

### Code Generation
- Use `/generate` for new code
- Use `/explain` for code explanation
- Use `/refactor` for code improvements
- Use `/test` for test generation

### Context Management
- Keep .ai directory in workspace
- Reference templates in prompts
- Maintain project context
- Update documentation

### Keyboard Shortcuts
```
Ctrl+Shift+P: Command palette
Ctrl+Space: AI suggestions
Ctrl+Enter: Accept suggestion
Ctrl+/: Toggle comment
```

## Best Practices

### Code Navigation
- Use Go to Definition
- Use Find References
- Use Symbol Search
- Use Quick Fix

### AI Interaction
- Be specific in prompts
- Provide context
- Review suggestions
- Validate changes

### Version Control
- Use GitLens
- Review changes
- Add meaningful commits
- Keep history clean

### Testing
- Run tests in IDE
- Debug with breakpoints
- Check coverage
- Fix failures quickly

## Project Integration

### Workspace Setup
```
.vscode/
  settings.json
  launch.json
  tasks.json
.ai/
  templates/
  contexts/
  plugins/
```

### Task Automation
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "test",
      "type": "shell",
      "command": "pytest",
      "group": {
        "kind": "test",
        "isDefault": true
      }
    }
  ]
}
```

### Debug Configuration
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal"
    }
  ]
}
```

## Troubleshooting

### Common Issues
1. AI not responding
2. Linting errors
3. Test failures
4. Git conflicts

### Solutions
1. Reload window
2. Check configuration
3. Update dependencies
4. Clean workspace 