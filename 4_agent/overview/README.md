# Agent Overview

Understand AI Dev Kit's agent system architecture and capabilities.

## Getting Started

Learn the fundamentals of working with agents:

- [How Agents Work](how-agents-work.md) - Core concepts and architecture
- [Your First Agent](your-first-agent.md) - Quick start guide
- [Agent vs Request](agent-vs-request.md) - When to use agents vs direct requests

## Core Components

Dive into the agent system's building blocks:

- [Agent Services](agent-services.md) - Service architecture and dependencies
- [Controllers](controllers.md) - Conversation, audio, and image controllers
- [Event Router](event-router.md) - Advanced event routing and filtering

## Customization

Extend agents with custom implementations:

- [Custom Chat Services](custom-chat-services.md) - Integrate custom LLM providers

## Architecture

```
Agent
 ├─ AgentControlHub (Status & Events)
 ├─ AgentChatApiAdapter (API Communication)
 ├─ ConversationController (History Management)
 ├─ AudioController (Voice I/O)
 ├─ ImageController (Visual Generation)
 └─ ToolController (Function Calling)
```

## Key Features

### Unified API
Interact with any LLM provider using a consistent interface:
- OpenAI (Chat Completions, Assistants, Responses, Realtime)
- Google Gemini
- Anthropic Claude
- And more...

### Stateful Conversations
Maintain context across multiple interactions:
- Automatic history management
- Multiple conversation support
- Persistent storage options

### Multimodal Support
Handle text, voice, images, and more:
- Voice input/output
- Image generation
- Document understanding
- Vision capabilities

### Tool Integration
Extend agent capabilities with tools:
- Built-in tools (search, calculation, etc.)
- Custom tool creation
- MCP (Model Context Protocol) support
- Automatic function calling

### Event System
Monitor and respond to agent activities:
- Status changes
- Streaming updates
- Tool execution
- Conversation events

## Next Steps

Start building with agents:

1. Create your first agent with [Your First Agent](your-first-agent.md)
2. Understand the architecture in [How Agents Work](how-agents-work.md)
3. Learn about [Agent Services](agent-services.md)
4. Explore [Event Router](event-router.md) for advanced patterns
