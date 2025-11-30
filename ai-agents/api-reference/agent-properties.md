# Agent Properties

Complete reference for Agent class properties.

## Overview

The `Agent` class provides core runtime properties for AI conversation management. Properties are organized by category: configuration, state, conversation, parameters, and capabilities.

## Model & Provider Properties

### Model

```csharp
public Model Model { get; set; }
```

Active language model for text generation.

**Example:**

```csharp
agent.Model = Model.GPT4;
agent.Model = Model.Claude35Sonnet;
agent.Model = Model.Gemini15Pro;
```

### SpeechModel

```csharp
public Model SpeechModel { get; set; }
```

Model used for text-to-speech synthesis.

**Example:**

```csharp
agent.SpeechModel = Model.TTS1HD;
```

### TranscriptionModel

```csharp
public Model TranscriptionModel { get; set; }
```

Model used for audio transcription.

**Example:**

```csharp
agent.TranscriptionModel = Model.Whisper1;
```

### ImageModel

```csharp
public Model ImageModel { get; set; }
```

Model used for image generation.

**Example:**

```csharp
agent.ImageModel = Model.DallE3;
```

### ChatServiceApi

```csharp
public ChatService ChatServiceApi { get; }
```

**Read-only.** Current chat service API (Responses/Realtime).

**Values:**

- `ChatService.Default` - Responses API
- `ChatService.ResponsesApi` - Responses API
- `ChatService.RealtimeApi` - Realtime API

## Voice & Audio Properties

### Voice

```csharp
public Voice Voice { get; set; }
```

TTS voice for audio responses.

**Example:**

```csharp
agent.Voice = Voice.Alloy;
agent.Voice = Voice.Nova;
```

### OutputAudioVolume

```csharp
public float OutputAudioVolume { get; set; }
```

Master volume for speech output (0.0 to 1.0).

**Example:**

```csharp
agent.OutputAudioVolume = 0.8f;
```

### InputAudioLanguage

```csharp
public SystemLanguage InputAudioLanguage { get; set; }
```

Language for audio input and transcription.

**Example:**

```csharp
agent.InputAudioLanguage = SystemLanguage.English;
agent.InputAudioLanguage = SystemLanguage.Korean;
```

## Configuration Properties

### Name

```csharp
public string Name { get; set; }
```

Human-readable agent name.

**Example:**

```csharp
agent.Name = "Assistant";
agent.Name = "GameMaster";
```

### Id

```csharp
public string Id { get; }
```

**Read-only.** Unique agent identifier.

### ConversationId

```csharp
public string ConversationId { get; set; }
```

Current conversation identifier.

**Example:**

```csharp
agent.ConversationId = "conv_123";
```

### Instructions

```csharp
public string Instructions { get; }
```

**Read-only.** System-level instructions for the agent.

### Settings

```csharp
public AgentSettings Settings { get; }
```

**Read-only.** Agent configuration settings.

## State Properties

### Status

```csharp
public AgentStatus Status { get; }
```

**Read-only.** Current agent lifecycle state.

**Values:**

- `None` - Not initialized
- `Initializing` - Initialization in progress
- `InitializationFailed` - Failed to initialize
- `Ready` - Ready for requests
- `Processing` - Handling request
- `Terminating` - Shutting down

**Example:**

```csharp
if (agent.Status == AgentStatus.Ready)
{
    await agent.GenerateResponseAsync("Hello");
}
```

### IsInitialized

```csharp
public bool IsInitialized { get; }
```

**Read-only.** True when agent is fully initialized.

**Example:**

```csharp
if (!agent.IsInitialized)
{
    await agent.InitializeAsync();
}
```

### CurrentResponse

```csharp
public Response CurrentResponse { get; }
```

**Read-only.** Response currently being generated.

## Conversation Properties

### Conversation

```csharp
public Conversation Conversation { get; }
```

**Read-only.** Active conversation instance.

**Example:**

```csharp
var messages = agent.Conversation.Messages;
var id = agent.Conversation.Id;
```

### Items

```csharp
public List<ConversationItem> Items { get; }
```

**Read-only.** All items in the conversation (messages, tool calls, outputs).

**Example:**

```csharp
foreach (var item in agent.Items)
{
    Debug.Log($"{item.Type}: {item}");
}
```

### Messages

```csharp
public List<Message> Messages { get; }
```

**Read-only.** User and assistant messages only.

**Example:**

```csharp
foreach (var message in agent.Messages)
{
    Debug.Log($"{message.Role}: {message.Content}");
}
```

### LastMessage

```csharp
public Message LastMessage { get; }
```

**Read-only.** Most recent message in conversation.

**Example:**

```csharp
if (agent.LastMessage?.Role == ConversationRole.Assistant)
{
    Debug.Log($"Last response: {agent.LastMessage.Content}");
}
```

## Parameter Properties

### Tools

```csharp
public List<Tool> Tools { get; }
```

**Read-only.** Registered tool definitions.

**Example:**

```csharp
Debug.Log($"Registered tools: {agent.Tools.Count}");
foreach (var tool in agent.Tools)
{
    Debug.Log($"- {tool.Function.Name}");
}
```

### ToolChoice

```csharp
public ToolChoice ToolChoice { get; set; }
```

Tool selection strategy.

**Values:**

- `ToolChoice.Auto` - Model decides
- `ToolChoice.None` - No tools
- `ToolChoice.Required` - Must use tool
- `ToolChoice.Function("name")` - Specific tool

**Example:**

```csharp
agent.ToolChoice = ToolChoice.Auto;
agent.ToolChoice = ToolChoice.Function("get_weather");
```

### MaxTokens

```csharp
public int? MaxTokens { get; }
```

**Read-only.** Maximum tokens for response.

### Temperature

```csharp
public float? Temperature { get; }
```

**Read-only.** Randomness/creativity level (0.0 to 2.0).

## Capability Properties

### HasInputAudio

```csharp
public bool HasInputAudio { get; }
```

**Read-only.** True if agent accepts audio input.

**Example:**

```csharp
if (agent.HasInputAudio)
{
    ShowMicrophoneButton();
}
```

### HasOutputAudio

```csharp
public bool HasOutputAudio { get; }
```

**Read-only.** True if agent can output speech.

**Example:**

```csharp
if (agent.HasOutputAudio)
{
    ShowSpeakerButton();
}
```

### HasOutputImage

```csharp
public bool HasOutputImage { get; }
```

**Read-only.** True if agent can generate images.

**Example:**

```csharp
if (agent.HasOutputImage)
{
    ShowImagePanel();
}
```

### HasMcpTool

```csharp
public bool HasMcpTool { get; set; }
```

True if agent has MCP tools registered.

## Behavior Properties

### ConversationStoreType

```csharp
public ConversationStoreType ConversationStoreType { get; }
```

**Read-only.** Storage backend for conversations.

**Values:**

- `LocalFile` - Local file system
- `CloudStorage` - Cloud storage
- `Database` - Database

### Stream

```csharp
public bool Stream { get; }
```

**Read-only.** True if streaming responses enabled.

### LogLevel

```csharp
public TraceLevel LogLevel { get; }
```

**Read-only.** Logging verbosity level.

**Values:**

- `Off` - No logging
- `Error` - Errors only
- `Warning` - Warnings and errors
- `Info` - Informational
- `Verbose` - Detailed debug info

### WaitForToolCallsCompletion

```csharp
public bool WaitForToolCallsCompletion { get; }
```

**Read-only.** True if agent waits for all tool calls to complete.

### UnhandledToolCallBehaviour

```csharp
public UnhandledToolCallBehaviour UnhandledToolCallBehaviour { get; }
```

**Read-only.** Policy for unhandled tool calls.

**Values:**

- `Ignore` - Skip unhandled calls
- `ThrowError` - Throw exception
- `SubmitToolOutput` - Wait for external submission
- `Event` - Fire onUnhandledToolCall event

### SubmitToolOutputTimeoutSeconds

```csharp
public int SubmitToolOutputTimeoutSeconds { get; }
```

**Read-only.** Timeout for tool output submission (seconds).

## Controller Properties

### conversationController

```csharp
public readonly ConversationController conversationController;
```

**Read-only.** Conversation management controller.

### audioController

```csharp
public readonly AudioController audioController;
```

**Read-only.** Audio I/O controller.

### imageController

```csharp
public readonly ImageController imageController;
```

**Read-only.** Image generation controller.

### toolCallController

```csharp
public readonly ToolCallController toolCallController;
```

**Read-only.** Tool execution controller.

### mcpController

```csharp
public readonly McpController mcpController;
```

**Read-only.** MCP integration controller.

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class AgentPropertiesDemo : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        DisplayAgentProperties();
    }
    
    void DisplayAgentProperties()
    {
        Debug.Log("=== Agent Properties ===");
        
        // Identity
        Debug.Log($"Id: {agent.Agent.Id}");
        Debug.Log($"Name: {agent.Agent.Name}");
        
        // State
        Debug.Log($"Status: {agent.Agent.Status}");
        Debug.Log($"IsInitialized: {agent.Agent.IsInitialized}");
        
        // Models
        Debug.Log($"Model: {agent.Agent.Model}");
        Debug.Log($"SpeechModel: {agent.Agent.SpeechModel}");
        Debug.Log($"TranscriptionModel: {agent.Agent.TranscriptionModel}");
        
        // Voice
        Debug.Log($"Voice: {agent.Agent.Voice}");
        Debug.Log($"Volume: {agent.Agent.OutputAudioVolume}");
        Debug.Log($"Language: {agent.Agent.InputAudioLanguage}");
        
        // Conversation
        Debug.Log($"ConversationId: {agent.Agent.ConversationId}");
        Debug.Log($"Messages: {agent.Agent.Messages?.Count ?? 0}");
        Debug.Log($"Tools: {agent.Agent.Tools?.Count ?? 0}");
        
        // Capabilities
        Debug.Log($"HasInputAudio: {agent.Agent.HasInputAudio}");
        Debug.Log($"HasOutputAudio: {agent.Agent.HasOutputAudio}");
        Debug.Log($"HasOutputImage: {agent.Agent.HasOutputImage}");
        
        // Parameters
        Debug.Log($"Temperature: {agent.Agent.Temperature}");
        Debug.Log($"MaxTokens: {agent.Agent.MaxTokens}");
        Debug.Log($"ToolChoice: {agent.Agent.ToolChoice}");
    }
}
```

## Next Steps

- [Agent Methods](agent-methods.md)
- [AgentBehaviour Properties](agentbehaviour-properties.md)
- [AgentBehaviour Methods](agentbehaviour-methods.md)
