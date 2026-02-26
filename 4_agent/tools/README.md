---
icon: screwdriver-wrench
---

# Tools & Abilities

Extend your agent's capabilities with powerful function calling tools.

## Overview

Tools enable agents to perform actions beyond text generation:
- Search the web
- Execute code
- Access external APIs
- Interact with your application
- Process data and files

## Getting Started

- [What are Tools?](what-are-tools.md) - Understanding agent tools
- [Built-in Tools](built-in-tools.md) - Pre-configured tools
- [Creating Custom Tools](creating-custom-tools.md) - Build your own tools
- [Tool Examples](tool-examples.md) - Real-world implementations

## Tool Types

### Built-in Tools
AI Dev Kit provides ready-to-use tools:
- Web search
- Image generation
- Code execution
- File operations
- Calculator

### Custom Tools
Create tools specific to your needs:
- Game mechanics
- Database queries
- API integrations
- Unity operations

### MCP Tools
Model Context Protocol tools for advanced integrations.

## How Tools Work

```
User: "Search for Unity tutorials"
  ↓
Agent analyzes request
  ↓
Agent calls search tool
  ↓
Tool returns results
  ↓
Agent synthesizes response
```

## Quick Example

```csharp
using Glitch9.AIDevKit.Agents;
using Glitch9.AIDevKit.Tools;

// Create agent with tools
var agent = new Agent(config, settings);

// Register a tool
var searchTool = new WebSearchTool();
agent.RegisterTool(searchTool);

// Agent automatically uses the tool when needed
await agent.SendAsync("What's the weather like?");
```

## Next Steps

Start building with tools:
1. Learn [What are Tools?](what-are-tools.md)
2. Explore [Built-in Tools](built-in-tools.md)
3. Create your own with [Creating Custom Tools](creating-custom-tools.md)
4. See examples in [Tool Examples](tool-examples.md)
