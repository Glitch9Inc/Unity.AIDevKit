# Models

Configure AI models for your agent.

## Overview

AI Dev Kit supports models from:

- **OpenAI** - GPT-4, GPT-3.5, o1, etc.
- **Anthropic** - Claude 3.5 Sonnet, Claude 3 Opus, etc.
- **Google** - Gemini 1.5 Pro, Gemini 1.5 Flash, etc.
- **Custom** - Any OpenAI-compatible API

## Setting Models

### In AgentSettings

```csharp
// Create or edit AgentSettings ScriptableObject
settings.Model = "gpt-4o";
settings.SpeechModel = "tts-1-hd";
settings.TranscriptionModel = "whisper-1";
settings.ImageModel = "dall-e-3";
```

### At Runtime

```csharp
// Change model dynamically
agent.Model = new Model("gpt-4o");

// Or using string
agent.Settings.Model = "claude-3-5-sonnet-20241022";
```

## OpenAI Models

### GPT-4o (Recommended)

```csharp
agent.Model = new Model("gpt-4o");

// Features:
// - Latest GPT-4 optimization
// - Vision capabilities
// - 128K context window
// - Function calling
// - JSON mode
// - Fast responses
```

### GPT-4 Turbo

```csharp
agent.Model = new Model("gpt-4-turbo");

// Features:
// - 128K context window
// - Vision capabilities
// - Knowledge up to April 2023
// - More affordable than GPT-4
```

### GPT-3.5 Turbo

```csharp
agent.Model = new Model("gpt-3.5-turbo");

// Features:
// - Fast responses
// - Lower cost
// - 16K context window
// - Good for simple tasks
```

### o1 Models (Reasoning)

```csharp
agent.Model = new Model("o1-preview");
// or
agent.Model = new Model("o1-mini");

// Features:
// - Advanced reasoning
// - Complex problem solving
// - Math and coding tasks
// - Slower but more accurate
```

## Anthropic Models

### Claude 3.5 Sonnet (Recommended)

```csharp
agent.ChatServiceApi = ChatService.ChatCompletions;
agent.Model = new Model("claude-3-5-sonnet-20241022");

// Features:
// - Excellent reasoning
// - 200K context window
// - Vision capabilities
// - Tool use
// - Best balance of speed/quality
```

### Claude 3 Opus

```csharp
agent.Model = new Model("claude-3-opus-20240229");

// Features:
// - Highest capability
// - 200K context window
// - Complex tasks
// - More expensive
```

### Claude 3 Haiku

```csharp
agent.Model = new Model("claude-3-haiku-20240307");

// Features:
// - Fastest responses
// - Most affordable
// - 200K context window
// - Simple tasks
```

## Google Models

### Gemini 1.5 Pro

```csharp
agent.Model = new Model("gemini-1.5-pro");

// Features:
// - 2M token context window
// - Multi-modal (text, image, audio, video)
// - Function calling
// - JSON mode
```

### Gemini 1.5 Flash

```csharp
agent.Model = new Model("gemini-1.5-flash");

// Features:
// - Fast responses
// - 1M token context window
// - Lower cost
// - Good performance
```

## Model Selection

### Choose by Task

```csharp
public Model GetModelForTask(string taskType)
{
    return taskType switch
    {
        "reasoning" => new Model("o1-preview"),
        "coding" => new Model("gpt-4o"),
        "chat" => new Model("gpt-4o"),
        "simple" => new Model("gpt-3.5-turbo"),
        "creative" => new Model("claude-3-5-sonnet-20241022"),
        "fast" => new Model("gpt-3.5-turbo"),
        _ => new Model("gpt-4o")
    };
}

// Usage
agent.Model = GetModelForTask("coding");
```

### Choose by Budget

```csharp
public class BudgetModelSelector
{
    public enum Budget { Low, Medium, High }
    
    public Model SelectModel(Budget budget)
    {
        return budget switch
        {
            Budget.Low => new Model("gpt-3.5-turbo"),      // ~$0.50/1M tokens
            Budget.Medium => new Model("gpt-4o-mini"),     // ~$0.15/1M tokens
            Budget.High => new Model("gpt-4o"),            // ~$5/1M tokens
            _ => new Model("gpt-4o-mini")
        };
    }
}
```

### Dynamic Model Switching

```csharp
public class SmartModelSelector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void SendSmart(string message)
    {
        // Use cheaper model for short messages
        if (message.Length < 100)
        {
            agent.Model = new Model("gpt-3.5-turbo");
        }
        // Use better model for complex queries
        else if (message.Contains("explain") || message.Contains("analyze"))
        {
            agent.Model = new Model("gpt-4o");
        }
        // Default
        else
        {
            agent.Model = new Model("gpt-4o-mini");
        }
        
        await agent.SendAsync(message);
    }
}
```

## Model Properties

### Context Window

```csharp
// Check model's max context
Dictionary<string, int> contextWindows = new()
{
    ["gpt-4o"] = 128_000,
    ["gpt-4-turbo"] = 128_000,
    ["gpt-3.5-turbo"] = 16_385,
    ["claude-3-5-sonnet-20241022"] = 200_000,
    ["gemini-1.5-pro"] = 2_000_000,
    ["o1-preview"] = 128_000
};

int maxTokens = contextWindows[agent.Model.Id];
Debug.Log($"Max context: {maxTokens:N0} tokens");
```

### Capabilities

```csharp
public class ModelCapabilities
{
    public bool SupportsVision { get; set; }
    public bool SupportsAudio { get; set; }
    public bool SupportsFunctionCalling { get; set; }
    public bool SupportsJsonMode { get; set; }
    public int MaxContextTokens { get; set; }
    
    public static ModelCapabilities Get(string modelId)
    {
        if (modelId.StartsWith("gpt-4o") || modelId.StartsWith("gpt-4-turbo"))
        {
            return new ModelCapabilities
            {
                SupportsVision = true,
                SupportsAudio = false,
                SupportsFunctionCalling = true,
                SupportsJsonMode = true,
                MaxContextTokens = 128_000
            };
        }
        else if (modelId.StartsWith("claude-3"))
        {
            return new ModelCapabilities
            {
                SupportsVision = true,
                SupportsAudio = false,
                SupportsFunctionCalling = true,
                SupportsJsonMode = false,
                MaxContextTokens = 200_000
            };
        }
        // ... more models
        
        return new ModelCapabilities();
    }
}
```

## Specialized Models

### TTS (Text-to-Speech)

```csharp
agent.SpeechModel = new Model("tts-1-hd");  // High quality
// or
agent.SpeechModel = new Model("tts-1");     // Standard quality

// Usage is automatic
agent.EnableOutputAudio = true;
await agent.SendAsync("Hello!");  // Response will be spoken
```

### Transcription

```csharp
agent.TranscriptionModel = new Model("whisper-1");

// Usage
AudioClip clip = GetAudioClip();
string text = await agent.TranscribeAsync(clip);
```

### Image Generation

```csharp
agent.ImageModel = new Model("dall-e-3");  // High quality
// or
agent.ImageModel = new Model("dall-e-2");  // Faster, cheaper

// Usage with tool
agent.RegisterToolExecutor(new ImageGenerationExecutor());
await agent.SendAsync("Generate an image of a sunset");
```

## Model Presets

### Create Preset System

```csharp
[System.Serializable]
public class ModelPreset
{
    public string Name;
    public string ModelId;
    public float Temperature;
    public int MaxTokens;
}

public class ModelPresetManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [SerializeField] private ModelPreset[] presets = new[]
    {
        new ModelPreset
        {
            Name = "Creative",
            ModelId = "gpt-4o",
            Temperature = 0.9f,
            MaxTokens = 2000
        },
        new ModelPreset
        {
            Name = "Precise",
            ModelId = "gpt-4o",
            Temperature = 0.2f,
            MaxTokens = 1000
        },
        new ModelPreset
        {
            Name = "Fast",
            ModelId = "gpt-3.5-turbo",
            Temperature = 0.7f,
            MaxTokens = 500
        }
    };
    
    public void ApplyPreset(string presetName)
    {
        var preset = presets.FirstOrDefault(p => p.Name == presetName);
        if (preset == null) return;
        
        agent.Model = new Model(preset.ModelId);
        agent.Temperature = preset.Temperature;
        agent.MaxTokens = preset.MaxTokens;
        
        Debug.Log($"Applied preset: {presetName}");
    }
}
```

## Cost Tracking

### Track Usage

```csharp
public class CostTracker : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    // Approximate costs per 1M tokens (input / output)
    private Dictionary<string, (float input, float output)> costs = new()
    {
        ["gpt-4o"] = (5f, 15f),
        ["gpt-4-turbo"] = (10f, 30f),
        ["gpt-3.5-turbo"] = (0.5f, 1.5f),
        ["claude-3-5-sonnet-20241022"] = (3f, 15f),
        ["gemini-1.5-pro"] = (1.25f, 5f)
    };
    
    private float totalCost = 0f;
    
    void Start()
    {
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseCompleted(Response response)
    {
        if (costs.TryGetValue(agent.Model.Id, out var pricing))
        {
            float inputCost = (response.Usage.PromptTokens / 1_000_000f) * pricing.input;
            float outputCost = (response.Usage.CompletionTokens / 1_000_000f) * pricing.output;
            float requestCost = inputCost + outputCost;
            
            totalCost += requestCost;
            
            Debug.Log($"Request cost: ${requestCost:F4}");
            Debug.Log($"Total cost: ${totalCost:F4}");
        }
    }
}
```

## Best Practices

### 1. Start with Best Model

```csharp
// Start with high-quality model
agent.Model = new Model("gpt-4o");

// Optimize later if needed
```

### 2. Cache Model Settings

```csharp
// Save user's preferred model
PlayerPrefs.SetString("PreferredModel", agent.Model.Id);

// Restore on startup
string preferred = PlayerPrefs.GetString("PreferredModel", "gpt-4o");
agent.Model = new Model(preferred);
```

### 3. Provide Model Selection UI

```csharp
public class ModelSelector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Dropdown dropdown;
    
    private string[] models = new[]
    {
        "gpt-4o",
        "gpt-4-turbo",
        "gpt-3.5-turbo",
        "claude-3-5-sonnet-20241022"
    };
    
    void Start()
    {
        dropdown.ClearOptions();
        dropdown.AddOptions(models.ToList());
        dropdown.onValueChanged.AddListener(OnModelChanged);
    }
    
    void OnModelChanged(int index)
    {
        agent.Model = new Model(models[index]);
        Debug.Log($"Model changed to: {models[index]}");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using System.Collections.Generic;
using System.Linq;

public class ModelManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Model Selection")]
    [SerializeField] private string defaultModel = "gpt-4o";
    
    private Dictionary<string, ModelInfo> modelDatabase = new()
    {
        ["gpt-4o"] = new ModelInfo
        {
            Name = "GPT-4o",
            Provider = "OpenAI",
            ContextWindow = 128_000,
            SupportsVision = true,
            CostPer1MTokens = 5f
        },
        ["gpt-3.5-turbo"] = new ModelInfo
        {
            Name = "GPT-3.5 Turbo",
            Provider = "OpenAI",
            ContextWindow = 16_385,
            SupportsVision = false,
            CostPer1MTokens = 0.5f
        },
        ["claude-3-5-sonnet-20241022"] = new ModelInfo
        {
            Name = "Claude 3.5 Sonnet",
            Provider = "Anthropic",
            ContextWindow = 200_000,
            SupportsVision = true,
            CostPer1MTokens = 3f
        }
    };
    
    void Start()
    {
        // Load saved model preference
        string savedModel = PlayerPrefs.GetString("PreferredModel", defaultModel);
        SetModel(savedModel);
        
        // Track usage
        agent.onResponseCompleted.AddListener(TrackUsage);
    }
    
    public void SetModel(string modelId)
    {
        if (modelDatabase.ContainsKey(modelId))
        {
            agent.Model = new Model(modelId);
            PlayerPrefs.SetString("PreferredModel", modelId);
            
            var info = modelDatabase[modelId];
            Debug.Log($"✓ Model set to: {info.Name} ({info.Provider})");
            Debug.Log($"  Context: {info.ContextWindow:N0} tokens");
            Debug.Log($"  Vision: {info.SupportsVision}");
        }
        else
        {
            Debug.LogWarning($"Unknown model: {modelId}");
        }
    }
    
    void TrackUsage(Response response)
    {
        var info = modelDatabase[agent.Model.Id];
        float cost = (response.Usage.TotalTokens / 1_000_000f) * info.CostPer1MTokens;
        
        Debug.Log($"Tokens: {response.Usage.TotalTokens:N0}");
        Debug.Log($"Cost: ${cost:F4}");
    }
    
    [System.Serializable]
    class ModelInfo
    {
        public string Name;
        public string Provider;
        public int ContextWindow;
        public bool SupportsVision;
        public float CostPer1MTokens;
    }
}
```

## Next Steps

- [Voice](voice.md)
- [Temperature & Max Tokens](temperature-tokens.md)
- [Chat Service Types](../configuration/chat-service-types.md)
