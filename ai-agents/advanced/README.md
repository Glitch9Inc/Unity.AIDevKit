# Advanced Topics

Deep dive into agent architecture, services, and customization.

## Overview

This section covers advanced concepts for developers who need:

- Deep understanding of agent internals
- Custom service implementations
- Advanced event handling
- Performance optimization

## Topics

### [Agent Services](agent-services.md)

Internal services that power the agent:

- Chat API Service
- Speech API Service
- Transcription API Service
- Image API Service
- File API Service

```csharp
var services = new AgentServices
{
    ChatApi = customChatService,
    SpeechApi = customSpeechService,
    TranscriptionApi = customTranscriptionService,
    ImageApi = customImageService
};

var agent = new Agent(settings, behaviour, hooks, services);
```

### [Controllers Architecture](controllers.md)

Agent uses specialized controllers for different concerns:

- **ConversationController** - Manages conversation state
- **AudioController** - Handles audio I/O
- **ImageController** - Manages image generation
- **ToolCallController** - Executes tool calls
- **McpController** - MCP integration
- **ParametersController** - Model parameters

```csharp
// Access controllers
agent.conversationController.LoadAsync(conversationId);
agent.audioController.StartRecording();
agent.toolCallController.RegisterExecutor(tool);
```

### [Event Router](event-router.md)

Internal event routing and propagation system.

```csharp
// Event flow:
Service → Controller → Agent → AgentBehaviour → UI
```

### [Custom Chat Services](custom-chat-services.md)

Implement custom AI providers:

```csharp
public class CustomChatService : IChatService
{
    public async UniTask<Response> SendAsync(Parameters parameters)
    {
        // Call custom AI API
        var result = await MyCustomApi.GenerateAsync(parameters);
        return ConvertToResponse(result);
    }
}
```

## Architecture Diagram

```
┌─────────────────────────────────────────────┐
│           AgentBehaviour (Unity)            │
│  ┌───────────────────────────────────────┐  │
│  │         Agent (Core Logic)            │  │
│  │  ┌────────────────────────────────┐   │  │
│  │  │      Controllers               │   │  │
│  │  │  ┌──────────────────────────┐  │   │  │
│  │  │  │  Services                │  │   │  │
│  │  │  │  - Chat API              │  │   │  │
│  │  │  │  - Speech API            │  │   │  │
│  │  │  │  - Transcription API     │  │   │  │
│  │  │  │  - Image API             │  │   │  │
│  │  │  └──────────────────────────┘  │   │  │
│  │  └────────────────────────────────┘   │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

## Controllers Responsibility

### ConversationController

- Load/save conversations
- Manage conversation items
- Assemble context for API calls
- Handle conversation listeners

### AudioController

- Record input audio
- Transcribe speech to text
- Generate output audio
- Play audio responses

### ImageController

- Generate images from text
- Handle image responses
- Manage image parameters

### ToolCallController

- Execute function tools
- Handle local shell tools
- Manage tool choice
- Submit tool outputs

### McpController

- Connect to MCP servers
- Handle tool approvals
- Manage OAuth tokens
- Execute MCP tools

### ParametersController

- Manage model parameters
- Handle preferences storage
- Dynamic tool registration
- Model API switching

## Custom Service Implementation

### 1. Define Service Interface

```csharp
public interface ICustomService : IAgentService<Input, Output, Parameters>
{
    UniTask<Output> ProcessAsync(Input input, Parameters parameters);
}
```

### 2. Implement Service

```csharp
public class CustomServiceImpl : ICustomService
{
    public async UniTask<Output> ProcessAsync(Input input, Parameters parameters)
    {
        // Custom implementation
        return await CustomApiCall(input, parameters);
    }
    
    public void Dispose()
    {
        // Cleanup
    }
}
```

### 3. Register Service

```csharp
var services = new AgentServices();
services.RegisterService<ICustomService>(new CustomServiceImpl());

var agent = new Agent(settings, behaviour, hooks, services);
```

## Performance Optimization

### 1. Minimize Conversation Context

```csharp
agent.Memory.MaxContextMessages = 20;
agent.Memory.SummarizationThreshold = 50;
```

### 2. Use Streaming

```csharp
agent.Stream = true; // Reduces perceived latency
```

### 3. Cache Responses

```csharp
var cache = new ResponseCache();
agent.onResponseCompleted += response => cache.Add(response);
```

### 4. Parallel Tool Execution

```csharp
settings.ParallelToolCalls = true;
```

## Debugging

### Enable Verbose Logging

```csharp
behaviour.LogLevel = TraceLevel.Verbose;
```

### Monitor Controller State

```csharp
Debug.Log($"Conversation: {agent.conversationController.IsInitialized}");
Debug.Log($"Audio: {agent.audioController.IsRecording}");
Debug.Log($"Tools: {agent.toolCallController.RegisteredToolCount}");
```

### Trace API Calls

```csharp
var logger = new CustomLogger();
var agent = new Agent(settings, behaviour, hooks, services, logger: logger);
```

## Next Steps

- [Agent Services](agent-services.md) - Service details
- [Controllers Architecture](controllers.md) - Controller deep dive
- [Event Router](event-router.md) - Event system
- [Custom Chat Services](custom-chat-services.md) - Custom providers
- [API Reference](../api-reference/README.md) - Complete API docs
