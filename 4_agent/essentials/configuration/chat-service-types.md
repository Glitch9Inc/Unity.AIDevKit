---
icon: list-ul
---

# Chat Service API Types

AI Dev Kit supports multiple chat service APIs, each with different capabilities and use cases.

## Available Chat Services

### Default (Responses API / Chat Completions)

**Automatically selects the best API for your provider:**

- OpenAI: Uses Responses API (new unified API)
- Other providers: Falls back to Chat Completions API

**When to use:**

- Most general-purpose applications
- You want automatic API selection
- Standard request/response pattern

**Example:**

```csharp
Chat Service API: Default
Model: gpt-4o
```

### Chat Completions API

**Traditional stateless chat API:**

- Supported by most providers (OpenAI, Anthropic, Google, etc.)
- Client manages conversation history
- Full control over context

**When to use:**

- Need cross-provider compatibility
- Want full control over conversation state
- Building custom conversation management

**Example:**

```csharp
Chat Service API: ChatCompletions
Model: claude-3-5-sonnet-20241022
Conversation Store: LocalFile
```

### Assistants API (OpenAI)

**Stateful assistant with server-side management:**

- OpenAI manages conversation threads
- Built-in tools (file search, code interpreter)
- Persistent across sessions

**When to use:**

- Building with OpenAI specifically
- Need file search or code interpreter
- Want server-side conversation persistence

**Example:**

```csharp
Chat Service API: AssistantsApi
Assistant Id: asst_xxxxx
Conversation Store: ThreadsApi
Enable File Search: true
Enable Code Interpreter: true
```

### Realtime API (OpenAI)

**Low-latency bidirectional voice conversations:**

- WebSocket-based streaming
- Optimized for voice interactions
- Real-time audio processing

**When to use:**

- Building voice assistants
- Need ultra-low latency
- Real-time audio conversations

**Example:**

```csharp
Chat Service API: RealtimeApi
Model: gpt-4o-realtime-preview
Enable Input Audio: true
Enable Output Audio: true
```

## Feature Comparison

| Feature | Chat Completions | Assistants API | Realtime API |
|---------|-----------------|----------------|--------------|
| **Provider Support** | All | OpenAI only | OpenAI only |
| **Conversation Management** | Client-side | Server-side | Server-side |
| **Streaming** | ✅ Text | ✅ Text | ✅ Audio |
| **Function Calling** | ✅ | ✅ | ✅ |
| **File Search** | ❌ | ✅ | ❌ |
| **Code Interpreter** | ❌ | ✅ | ❌ |
| **Voice I/O** | Via separate APIs | Via separate APIs | ✅ Native |
| **Latency** | Medium | Medium-High | Ultra-low |

## Configuration Examples

### Multi-Provider Chat App

```csharp
// Use Chat Completions for maximum compatibility
settings.ChatServiceApi = ChatService.ChatCompletions;

// Switch models at runtime
agent.Model = "gpt-4o"; // OpenAI
agent.Model = "claude-3-5-sonnet-20241022"; // Anthropic
agent.Model = "gemini-2.0-flash-exp"; // Google
```

### OpenAI Assistant with Tools

```csharp
settings.ChatServiceApi = ChatService.AssistantsApi;
settings.AssistantsApiOptions.AssistantId = "asst_xxxxx";
settings.EnableFileSearch = true;
settings.EnableCodeInterpreter = true;

behaviour.ConversationStoreType = ConversationStoreType.ThreadsApi;
```

### Voice Assistant

```csharp
settings.ChatServiceApi = ChatService.RealtimeApi;
settings.Model = "gpt-4o-realtime-preview";
settings.EnableInputAudio = true;
settings.EnableOutputAudio = true;
settings.Voice = "alloy";

behaviour.Stream = true;
```

## Switching APIs at Runtime

```csharp
// Not recommended - requires reinitializing agent
// Better to create separate agents for different APIs

// If you must switch:
agent.Dispose();
settings.ChatServiceApi = ChatService.AssistantsApi;
agent = new Agent(settings, behaviour);
await agent.InitializeAsync();
```

## Next Steps

- [Agent Settings](agent-settings.md) - Configure models and parameters
- [Conversation Store](conversation-store.md) - Choose persistence strategy
- [Tools](../tools/README.md) - Add function calling capabilities
