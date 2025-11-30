# Agent Settings

AgentSettings is a ScriptableObject that defines all persistent configuration for your agent.

## Creating Agent Settings

1. Right-click in Project window
2. Select `Create > AI Dev Kit > Agent Settings`
3. Name it descriptively (e.g., "CustomerSupportAgent")

## Core Settings

### Identification

```csharp
// Unique identifier (auto-generated)
public string Id { get; }

// Human-readable name
public string Name { get; }
```

### Chat Service API

Choose which API type to use:

```csharp
public ChatService ChatServiceApi { get; set; }
```

Options:

- **Default** - Auto-selects best API
- **AssistantsApi** - OpenAI Assistants
- **RealtimeApi** - Low-latency voice

### Model Configuration

```csharp
public string Model { get; set; }
public Temperature Temperature { get; set; }
public TopP TopP { get; set; }
public TokenCount MaxTokens { get; set; }
public Seed Seed { get; set; }
```

Example:

```csharp
settings.Model = "gpt-4o";
settings.Temperature = 0.7f;
settings.MaxTokens = 2000;
```

### Instructions (System Prompt)

```csharp
public string Instructions { get; }
```

Define agent's behavior and personality:

```
You are a helpful customer support assistant.
Be friendly, professional, and concise.
Always ask for clarification if needed.
```

## Agent Profile

Configure the agent's identity:

```csharp
[SerializeField] private AgentProfile agentProfile;

public string Name { get; }
public string Description { get; }
public string StartingMessage { get; }
```

Create via `Create > AI Dev Kit > Agent Profile`.

## User Profile

Configure user information:

```csharp
[SerializeField] private UserProfile userProfile;

public string UserName { get; }
public string UserOccupation { get; }
```

## Audio Settings

### Input Audio

```csharp
public bool EnableInputAudio { get; set; }
public TranscriptionParameters InputAudioParameters { get; }
public SystemLanguage InputAudioLanguage { get; }
```

### Output Audio

```csharp
public bool EnableOutputAudio { get; set; }
public SpeechParameters OutputAudioParameters { get; }
public string Voice { get; }
```

Example:

```csharp
settings.EnableOutputAudio = true;
settings.OutputAudioParameters.Voice = "alloy";
settings.OutputAudioParameters.Speed = 1.0f;
```

## Tool Configuration

### Tool Definitions

```csharp
public List<ToolDefinitionBase> ToolDefinitions { get; }
```

Add tools in Inspector or code:

```csharp
var imageTool = ScriptableObject.CreateInstance<ImageGenerationToolDefinition>();
imageTool.Model = "dall-e-3";
settings.ToolDefinitions.Add(imageTool);
```

### Tool Parameters

```csharp
public int? MaxToolCalls { get; }
public bool ParallelToolCalls { get; set; }
```

## Advanced Settings

### Reasoning

```csharp
public bool EnableReasoning { get; set; }
public ReasoningOptions ReasoningOptions { get; }
```

For models that support reasoning (o1, o3):

```csharp
settings.EnableReasoning = true;
settings.ReasoningOptions.Effort = ReasoningEffort.High;
```

### Moderation

```csharp
public bool EnableModeration { get; set; }
```

Enable content moderation:

```csharp
settings.EnableModeration = true;
```

### Memory

```csharp
public AgentMemorySettings Memory { get; }
```

Configure conversation memory:

```csharp
settings.Memory.MaxContextMessages = 50;
settings.Memory.EnableSummarization = true;
```

### Safety (Google)

```csharp
public List<SafetySetting> SafetySettings { get; }
```

For Google Gemini models:

```csharp
settings.SafetySettings.Add(new SafetySetting
{
    Category = HarmCategory.HateSpeech,
    Threshold = HarmBlockThreshold.BlockMediumAndAbove
});
```

### Assistants API Options

```csharp
public AssistantsApiSettings AssistantsApiOptions { get; }
```

Configure OpenAI Assistants:

```csharp
settings.AssistantsApiOptions.AssistantId = "asst_xxxxx";
settings.AssistantsApiOptions.RunPollingIntervalMs = 2000;
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

[CreateAssetMenu(fileName = "NewAgentSettings", menuName = "Examples/Agent Settings")]
public class CustomAgentSettings : ScriptableObject
{
    public static AgentSettings CreateDefault()
    {
        var settings = CreateInstance<AgentSettings>();
        
        // Core settings
        settings.ChatServiceApi = ChatService.Default;
        settings.Model = "gpt-4o";
        settings.Temperature = 0.7f;
        settings.MaxTokens = 2000;
        
        // Audio
        settings.EnableOutputAudio = true;
        settings.OutputAudioParameters = new SpeechParameters
        {
            Voice = "alloy",
            Speed = 1.0f
        };
        
        // Tools
        settings.ParallelToolCalls = true;
        
        // Memory
        settings.Memory.MaxContextMessages = 50;
        settings.Memory.EnableSummarization = true;
        
        return settings;
    }
}
```

## Runtime Modification

Some settings can be changed at runtime:

```csharp
// Can change
agent.Model = "gpt-4o-mini";
agent.Temperature = 0.9f;
agent.Voice = "nova";

// Cannot change (require reinitialize)
// settings.ChatServiceApi
// settings.EnableInputAudio
// settings.ToolDefinitions
```

## Validation

Settings are validated on initialization:

```csharp
try
{
    await agent.InitializeAsync();
}
catch (ArgumentException ex)
{
    Debug.LogError($"Invalid settings: {ex.Message}");
}
```

Common validation errors:

- Missing or invalid model ID
- Incompatible feature combinations
- Invalid API key
- Missing required tool definitions

## Best Practices

### 1. Create Presets

Create different settings for different purposes:

- `FastResponse.asset` - GPT-3.5, low temperature
- `Creative.asset` - GPT-4, high temperature
- `VoiceAssistant.asset` - Realtime API, audio enabled

### 2. Use Profiles

Separate personality (AgentProfile) from functionality (AgentSettings):

```
Settings/
├─ SupportAgent.asset (settings)
└─ Profiles/
   ├─ FriendlyHelper.asset
   └─ ProfessionalAdvisor.asset
```

### 3. Version Control

Keep settings in version control but exclude API keys.

### 4. Environment-Specific

Use different settings for dev/prod:

```
Settings/
├─ Dev/
│  └─ TestAgent.asset
└─ Prod/
   └─ ProductionAgent.asset
```

## Next Steps

- [Chat Service API Types](chat-service-types.md) - Choose API
- [Audio Setup](audio-setup.md) - Configure voice I/O
- [Memory Settings](memory-settings.md) - Manage context
