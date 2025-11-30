# Agent Configuration

Configure your agent's behavior, capabilities, and features through AgentSettings.

## Overview

Agent configuration is managed through two main components:

1. **AgentSettings** (ScriptableObject) - Persistent configuration
2. **IAgentBehaviour** (Interface) - Runtime behavior settings

This section covers how to configure every aspect of your agent.

## Configuration Hierarchy

```
AgentSettings (ScriptableObject)
├─ Chat Service API Type
├─ Model Parameters (Model, Temperature, etc.)
├─ User & Agent Profiles
├─ Audio Settings (Input/Output)
├─ Tool Definitions
├─ Memory Settings
├─ Reasoning & Safety Options
└─ Assistants API Options

IAgentBehaviour (Runtime)
├─ Conversation Store Type
├─ Streaming Options
├─ Tool Call Behavior
├─ MCP Settings
└─ Logging Level
```

## Core Configuration Files

### AgentSettings

Create via: `Create > AI Dev Kit > Agent Settings`

**Key Properties:**

- **Id** - Unique identifier (auto-generated)
- **Chat Service API** - Which API to use
- **Model** - Language model ID
- **Instructions** - System prompt
- **User Profile** - User information
- **Agent Profile** - Agent personality and behavior

### AgentBehaviour Configuration

Set in Inspector or code:

```csharp
[SerializeField] private AgentBehaviour agent;

void Configure()
{
    agent.ConversationStore = ConversationStoreType.LocalFile;
    agent.Stream = true;
    agent.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.RaiseEvent;
    agent.SubmitToolOutputTimeoutSeconds = 60;
}
```

## Configuration Categories

### [Agent Settings](agent-settings.md)

Core configuration including models, instructions, and profiles.

### [Chat Service API Types](chat-service-types.md)

Choose between Chat Completions, Assistants API, or Realtime API.

### [Conversation Store](conversation-store.md)

Configure how conversations are persisted and loaded.

### [Audio Setup](audio-setup.md)

Enable and configure voice input and output.

### [Memory Settings](memory-settings.md)

Configure conversation memory and context management.

## Quick Configuration Examples

### Basic Chat Agent

```csharp
// In AgentSettings
Chat Service API: ChatCompletions
Model: gpt-4o
Instructions: "You are a helpful assistant."
Temperature: 0.7
Max Tokens: 1000

// In AgentBehaviour
Conversation Store: LocalFile
Stream: true
```

### Voice Assistant

```csharp
// In AgentSettings
Chat Service API: RealtimeApi
Model: gpt-4o-realtime-preview
Enable Input Audio: true
Enable Output Audio: true
Speech Model: tts-1
Voice: alloy

// In AgentBehaviour
Conversation Store: Memory
Stream: true
```

### Tool-Using Agent

```csharp
// In AgentSettings
Chat Service API: ChatCompletions
Model: gpt-4o
Tools: [Function Calling, Web Search]
Parallel Tool Calls: true

// In AgentBehaviour
Unhandled Tool Call Behaviour: RaiseEvent
Wait For Tool Calls Completion: true
```

### Persistent Assistant

```csharp
// In AgentSettings
Chat Service API: AssistantsApi
Assistant Id: "asst_xxxxx"
Enable File Search: true
Enable Code Interpreter: true

// In AgentBehaviour
Conversation Store: ThreadsApi
Stream: true
```

## Configuration Best Practices

### 1. Separate Settings for Different Agents

Create different AgentSettings for different purposes:

```
Assets/Settings/
├─ CustomerSupportAgent.asset
├─ CreativeWriterAgent.asset
├─ CodeAssistantAgent.asset
└─ VoiceCompanionAgent.asset
```

### 2. Use Profiles for Reusable Personalities

```
Assets/Profiles/
├─ FriendlyAssistant.asset
├─ ProfessionalAdvisor.asset
└─ CasualChatbot.asset
```

### 3. Environment-Specific Configuration

```csharp
#if UNITY_EDITOR
    agent.LogLevel = TraceLevel.Verbose;
#else
    agent.LogLevel = TraceLevel.Warning;
#endif
```

### 4. Runtime Model Switching

```csharp
// Switch to faster model for simple queries
if (isSimpleQuery)
{
    agent.Model = "gpt-3.5-turbo";
}
else
{
    agent.Model = "gpt-4o";
}
```

## Configuration Validation

AgentSettings validates configuration on initialization:

```csharp
// Invalid configurations will throw exceptions
try
{
    await agent.InitializeAsync();
}
catch (ArgumentException ex)
{
    Debug.LogError($"Invalid configuration: {ex.Message}");
}
```

**Common Validation Errors:**

- Missing or invalid model ID
- Invalid API key
- Incompatible feature combinations
- Missing required tool definitions

## Dynamic Configuration

Change settings at runtime:

```csharp
// Change model
agent.Model = "gpt-4o";

// Change voice
agent.Voice = "nova";

// Change temperature
agent.Temperature = 0.9f;

// Add tools dynamically
agent.RegisterToolExecutor(new MyCustomTool());
```

## Configuration Inheritance

AgentBehaviour can inherit settings:

```csharp
public class CustomAgent : MonoBehaviour, IAgentBehaviour
{
    // Override default behavior
    public bool AutoInit => true;
    public ConversationStoreType ConversationStoreType => ConversationStoreType.LocalFile;
    public bool Stream => true;
    
    // Custom settings
    public int CustomTimeout => 120;
}
```

## Next Steps

- [Agent Settings](agent-settings.md) - Detailed AgentSettings configuration
- [Chat Service Types](chat-service-types.md) - Choose the right API
- [Conversation Store](conversation-store.md) - Persist conversations
- [Audio Setup](audio-setup.md) - Configure voice features
- [Memory Settings](memory-settings.md) - Manage context and memory
