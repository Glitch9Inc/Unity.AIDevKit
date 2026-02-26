---
icon: sliders
---

# Parameters

Configure model parameters and runtime settings.

## Overview

Parameters control how the AI model generates responses:

- **Model** - Which AI model to use
- **Temperature** - Randomness/creativity
- **Max Tokens** - Response length limit
- **Top P** - Nucleus sampling
- **Tools** - Available functions
- **Voice** - TTS voice selection

## Models

### Set Model

```csharp
// At initialization
settings.Model = "gpt-4o";

// At runtime
agent.Model = "gpt-4o-mini";
```

### Available Models

```csharp
// OpenAI
agent.Model = "gpt-4o";
agent.Model = "gpt-4o-mini";
agent.Model = "o1";
agent.Model = "o1-mini";

// Anthropic
agent.Model = "claude-3-5-sonnet-20241022";
agent.Model = "claude-3-5-haiku-20241022";

// Google
agent.Model = "gemini-2.0-flash-exp";
agent.Model = "gemini-1.5-pro";
```

### Model Properties

```csharp
var model = agent.Model;
Debug.Log($"Name: {model.Name}");
Debug.Log($"Context Window: {model.ContextWindow}");
Debug.Log($"Max Output: {model.MaxOutputTokens}");
Debug.Log($"Supports Tools: {model.SupportsTools}");
Debug.Log($"Supports Vision: {model.SupportsVision}");
```

## Temperature

Controls randomness (0.0 - 2.0):

```csharp
// Low temperature (deterministic, focused)
agent.Temperature = 0.2f;

// Medium temperature (balanced)
agent.Temperature = 0.7f;

// High temperature (creative, random)
agent.Temperature = 1.5f;
```

**Guidelines:**

- **0.0 - 0.3** - Factual, consistent, predictable
- **0.4 - 0.8** - Balanced creativity
- **0.9 - 2.0** - Very creative, unpredictable

## Max Tokens

Limit response length:

```csharp
// Short responses
agent.MaxTokens = 500;

// Medium responses
agent.MaxTokens = 2000;

// Long responses
agent.MaxTokens = 4000;
```

**Note:** 1 token ≈ 0.75 words

## Top P

Nucleus sampling (0.0 - 1.0):

```csharp
// Very focused
agent.TopP = 0.1f;

// Balanced (default)
agent.TopP = 1.0f;
```

Usually leave at 1.0 and control via temperature instead.

## Voice

For text-to-speech:

```csharp
agent.Voice = "alloy";   // Neutral
agent.Voice = "echo";    // Warm
agent.Voice = "fable";   // British
agent.Voice = "onyx";    // Deep
agent.Voice = "nova";    // Friendly
agent.Voice = "shimmer"; // Energetic
```

## Tools & Tool Choice

### Add Tools

```csharp
agent.RegisterToolExecutor(new WeatherTool());
```

### Tool Choice

```csharp
// Let agent decide
agent.ToolChoice = ToolChoice.Auto;

// Require tool use
agent.ToolChoice = ToolChoice.Required;

// Disable tools
agent.ToolChoice = ToolChoice.None;

// Force specific tool
agent.ToolChoice = ToolChoice.Function("get_weather");
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ParameterController : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    // Preset configurations
    public void SetFastMode()
    {
        agent.Model = "gpt-4o-mini";
        agent.Temperature = 0.3f;
        agent.MaxTokens = 500;
    }
    
    public void SetCreativeMode()
    {
        agent.Model = "gpt-4o";
        agent.Temperature = 1.2f;
        agent.MaxTokens = 2000;
    }
    
    public void SetAnalyticalMode()
    {
        agent.Model = "o1";
        agent.Temperature = 1.0f;
        agent.MaxTokens = 4000;
    }
    
    // Dynamic adjustment
    public void AdjustTemperature(float value)
    {
        agent.Temperature = value;
        Debug.Log($"Temperature: {value}");
    }
    
    public void AdjustMaxTokens(int value)
    {
        agent.MaxTokens = value;
        Debug.Log($"Max Tokens: {value}");
    }
}
```

## Preferences Store

Save user preferences:

```csharp
// Preferences are auto-saved
agent.Model = "gpt-4o";
agent.Temperature = 0.8f;

// Restored on next initialization
```

## Next Steps

- [Models](models.md)
- [Voice](voice.md)
- [Tools & Tool Choice](tools.md)
- [Temperature & Max Tokens](temperature-tokens.md)
- [Preferences Store](preferences-store.md)
