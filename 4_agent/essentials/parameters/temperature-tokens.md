# Temperature & Max Tokens

Control response randomness and length with temperature and token limits.

## Temperature

Temperature controls randomness in responses:

- **0.0** - Deterministic, consistent (factual tasks)
- **0.5** - Balanced (general use)
- **1.0** - Creative, varied (creative writing)
- **2.0** - Very random (experimental)

### Setting Temperature

```csharp
// In AgentSettings
settings.Temperature = 0.7f;

// At runtime
agent.Temperature = 0.7f;
```

### Temperature Examples

```csharp
// Factual, consistent responses
agent.Temperature = 0.2f;
await agent.SendAsync("What is the capital of Japan?");
// Always: "Tokyo"

// Creative responses
agent.Temperature = 0.9f;
await agent.SendAsync("Write a short story about a robot");
// Different story each time
```

## Temperature Presets

### By Use Case

```csharp
public float GetTemperature(string useCase)
{
    return useCase switch
    {
        "factual" => 0.2f,        // Code, facts, math
        "balanced" => 0.7f,       // General chat
        "creative" => 1.0f,       // Stories, ideas
        "experimental" => 1.5f,   // Very creative
        _ => 0.7f
    };
}

// Usage
agent.Temperature = GetTemperature("factual");
```

### Task-Based Selection

```csharp
public class TemperatureSelector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void SendWithAppropriateTemperature(string message)
    {
        // Analyze message type
        if (IsCodeRelated(message))
        {
            agent.Temperature = 0.2f; // Precise
        }
        else if (IsCreativeTask(message))
        {
            agent.Temperature = 0.9f; // Creative
        }
        else
        {
            agent.Temperature = 0.7f; // Balanced
        }
        
        await agent.SendAsync(message);
    }
    
    bool IsCodeRelated(string message)
    {
        return message.Contains("code") || 
               message.Contains("function") ||
               message.Contains("bug");
    }
    
    bool IsCreativeTask(string message)
    {
        return message.Contains("story") ||
               message.Contains("creative") ||
               message.Contains("imagine");
    }
}
```

## Max Tokens

Max tokens limits response length:

- Controls how long responses can be
- Prevents excessive token usage
- Manages costs

### Setting Max Tokens

```csharp
// In AgentSettings
settings.MaxTokens = 1000;

// At runtime
agent.MaxTokens = 1000;
```

### Token Guidelines

```csharp
// Short responses (100-200 tokens)
agent.MaxTokens = 200;
await agent.SendAsync("Quick summary please");

// Medium responses (500-1000 tokens)
agent.MaxTokens = 1000;
await agent.SendAsync("Explain this concept");

// Long responses (2000+ tokens)
agent.MaxTokens = 2000;
await agent.SendAsync("Write a detailed guide");

// No limit (use model's max)
agent.MaxTokens = null;
```

### Rough Token Estimates

```csharp
public class TokenEstimator
{
    // Rough estimation: 1 token ≈ 4 characters (English)
    public static int EstimateTokens(string text)
    {
        return text.Length / 4;
    }
    
    // Estimate words
    public static int EstimateWords(int tokens)
    {
        return (int)(tokens * 0.75f); // ~0.75 words per token
    }
}

// Examples:
// 100 tokens ≈ 75 words ≈ 400 characters
// 500 tokens ≈ 375 words ≈ 2000 characters
// 1000 tokens ≈ 750 words ≈ 4000 characters
```

## Dynamic Max Tokens

### Based on Context

```csharp
public int GetMaxTokens(string messageType)
{
    return messageType switch
    {
        "quick_answer" => 100,
        "explanation" => 500,
        "tutorial" => 1500,
        "documentation" => 3000,
        _ => 1000
    };
}

// Usage
agent.MaxTokens = GetMaxTokens("explanation");
```

### Adaptive Tokens

```csharp
public class AdaptiveTokens : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int minTokens = 100;
    [SerializeField] private int maxTokens = 2000;
    
    public async void SendAdaptive(string message)
    {
        // Estimate desired response length based on question length
        int questionTokens = EstimateTokens(message);
        int responseTokens = Mathf.Clamp(
            questionTokens * 3, // 3x the question length
            minTokens,
            maxTokens
        );
        
        agent.MaxTokens = responseTokens;
        
        Debug.Log($"Question: {questionTokens} tokens");
        Debug.Log($"Max response: {responseTokens} tokens");
        
        await agent.SendAsync(message);
    }
    
    int EstimateTokens(string text)
    {
        return text.Length / 4;
    }
}
```

## Combining Temperature & Max Tokens

### Response Presets

```csharp
[System.Serializable]
public class ResponsePreset
{
    public string Name;
    public float Temperature;
    public int MaxTokens;
    public string Description;
}

public class PresetManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [SerializeField] private ResponsePreset[] presets = new[]
    {
        new ResponsePreset
        {
            Name = "Quick Answer",
            Temperature = 0.3f,
            MaxTokens = 100,
            Description = "Short, factual responses"
        },
        new ResponsePreset
        {
            Name = "Balanced",
            Temperature = 0.7f,
            MaxTokens = 500,
            Description = "Normal conversation"
        },
        new ResponsePreset
        {
            Name = "Detailed",
            Temperature = 0.5f,
            MaxTokens = 2000,
            Description = "Long, thorough responses"
        },
        new ResponsePreset
        {
            Name = "Creative",
            Temperature = 1.0f,
            MaxTokens = 1500,
            Description = "Creative, varied responses"
        }
    };
    
    public void ApplyPreset(string presetName)
    {
        var preset = System.Array.Find(presets, p => p.Name == presetName);
        if (preset == null) return;
        
        agent.Temperature = preset.Temperature;
        agent.MaxTokens = preset.MaxTokens;
        
        Debug.Log($"Applied preset: {preset.Name}");
        Debug.Log($"  Temperature: {preset.Temperature}");
        Debug.Log($"  Max Tokens: {preset.MaxTokens}");
    }
}
```

## UI Controls

### Temperature Slider

```csharp
public class TemperatureControl : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Slider temperatureSlider;
    [SerializeField] private TMP_Text temperatureText;
    
    void Start()
    {
        temperatureSlider.minValue = 0f;
        temperatureSlider.maxValue = 2f;
        temperatureSlider.value = 0.7f;
        
        temperatureSlider.onValueChanged.AddListener(OnTemperatureChanged);
        UpdateText(0.7f);
    }
    
    void OnTemperatureChanged(float value)
    {
        agent.Temperature = value;
        UpdateText(value);
    }
    
    void UpdateText(float value)
    {
        string mode = value switch
        {
            < 0.3f => "Precise",
            < 0.7f => "Balanced",
            < 1.2f => "Creative",
            _ => "Experimental"
        };
        
        temperatureText.text = $"Temperature: {value:F2} ({mode})";
    }
}
```

### Max Tokens Input

```csharp
public class MaxTokensControl : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField maxTokensInput;
    [SerializeField] private TMP_Text estimateText;
    
    void Start()
    {
        maxTokensInput.text = "1000";
        maxTokensInput.onValueChanged.AddListener(OnMaxTokensChanged);
        UpdateEstimate(1000);
    }
    
    void OnMaxTokensChanged(string value)
    {
        if (int.TryParse(value, out int tokens))
        {
            agent.MaxTokens = tokens;
            UpdateEstimate(tokens);
        }
    }
    
    void UpdateEstimate(int tokens)
    {
        int words = (int)(tokens * 0.75f);
        estimateText.text = $"≈ {words} words";
    }
}
```

### Preset Dropdown

```csharp
public class PresetDropdown : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Dropdown presetDropdown;
    
    private string[] presetNames = { "Quick", "Balanced", "Detailed", "Creative" };
    
    void Start()
    {
        presetDropdown.ClearOptions();
        presetDropdown.AddOptions(presetNames.ToList());
        presetDropdown.onValueChanged.AddListener(OnPresetChanged);
    }
    
    void OnPresetChanged(int index)
    {
        switch (index)
        {
            case 0: // Quick
                agent.Temperature = 0.3f;
                agent.MaxTokens = 100;
                break;
            case 1: // Balanced
                agent.Temperature = 0.7f;
                agent.MaxTokens = 500;
                break;
            case 2: // Detailed
                agent.Temperature = 0.5f;
                agent.MaxTokens = 2000;
                break;
            case 3: // Creative
                agent.Temperature = 1.0f;
                agent.MaxTokens = 1500;
                break;
        }
        
        Debug.Log($"Preset: {presetNames[index]}");
    }
}
```

## Token Usage Tracking

### Monitor Usage

```csharp
public class TokenTracker : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private int totalTokensUsed = 0;
    
    void Start()
    {
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseCompleted(Response response)
    {
        int tokens = response.Usage.TotalTokens;
        totalTokensUsed += tokens;
        
        Debug.Log($"Tokens used: {tokens:N0}");
        Debug.Log($"  Input: {response.Usage.PromptTokens:N0}");
        Debug.Log($"  Output: {response.Usage.CompletionTokens:N0}");
        Debug.Log($"Total session: {totalTokensUsed:N0}");
        
        // Check if approaching max
        if (response.Usage.CompletionTokens >= agent.MaxTokens * 0.9f)
        {
            Debug.LogWarning("Response approaching max tokens limit");
        }
    }
}
```

### Budget Management

```csharp
public class TokenBudget : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int dailyTokenBudget = 10000;
    
    private int tokensUsedToday = 0;
    private DateTime lastResetDate;
    
    void Start()
    {
        LoadUsage();
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseCompleted(Response response)
    {
        // Reset if new day
        if (DateTime.Today > lastResetDate)
        {
            tokensUsedToday = 0;
            lastResetDate = DateTime.Today;
        }
        
        tokensUsedToday += response.Usage.TotalTokens;
        SaveUsage();
        
        // Check budget
        float percentUsed = (tokensUsedToday / (float)dailyTokenBudget) * 100f;
        Debug.Log($"Daily budget: {percentUsed:F1}% ({tokensUsedToday}/{dailyTokenBudget})");
        
        if (tokensUsedToday >= dailyTokenBudget)
        {
            Debug.LogWarning("Daily token budget exceeded!");
        }
    }
    
    void LoadUsage()
    {
        tokensUsedToday = PlayerPrefs.GetInt("TokensUsedToday", 0);
        long ticks = long.Parse(PlayerPrefs.GetString("LastResetDate", "0"));
        lastResetDate = new DateTime(ticks);
    }
    
    void SaveUsage()
    {
        PlayerPrefs.SetInt("TokensUsedToday", tokensUsedToday);
        PlayerPrefs.SetString("LastResetDate", lastResetDate.Ticks.ToString());
    }
}
```

## Advanced Settings

### Top P (Nucleus Sampling)

```csharp
// Alternative to temperature
// Controls diversity by limiting token choices
agent.TopP = 0.9f; // Use top 90% probable tokens

// Don't use both temperature and top_p at once
// Typically use one OR the other
```

### Frequency Penalty

```csharp
// Reduce repetition (0 to 2)
agent.FrequencyPenalty = 0.5f;

// Higher values = less repetition
```

### Presence Penalty

```csharp
// Encourage new topics (0 to 2)
agent.PresencePenalty = 0.6f;

// Higher values = more diverse topics
```

## Best Practices

### 1. Start Conservative

```csharp
// Start with safe defaults
agent.Temperature = 0.7f;
agent.MaxTokens = 1000;

// Adjust based on results
```

### 2. Match Parameters to Task

```csharp
public void ConfigureForTask(string taskType)
{
    switch (taskType)
    {
        case "code":
            agent.Temperature = 0.2f;
            agent.MaxTokens = 500;
            break;
            
        case "story":
            agent.Temperature = 0.9f;
            agent.MaxTokens = 2000;
            break;
            
        case "chat":
            agent.Temperature = 0.7f;
            agent.MaxTokens = 500;
            break;
    }
}
```

### 3. Save User Preferences

```csharp
void SavePreferences()
{
    PlayerPrefs.SetFloat("Temperature", agent.Temperature ?? 0.7f);
    PlayerPrefs.SetInt("MaxTokens", agent.MaxTokens ?? 1000);
}

void LoadPreferences()
{
    agent.Temperature = PlayerPrefs.GetFloat("Temperature", 0.7f);
    agent.MaxTokens = PlayerPrefs.GetInt("MaxTokens", 1000);
}
```

## Complete Example

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;
using TMPro;

public class ParameterManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("UI")]
    [SerializeField] private Slider temperatureSlider;
    [SerializeField] private TMP_Text temperatureText;
    [SerializeField] private TMP_InputField maxTokensInput;
    [SerializeField] private TMP_Text tokensUsedText;
    [SerializeField] private TMP_Dropdown presetDropdown;
    
    private int sessionTokens = 0;
    
    void Start()
    {
        SetupTemperature();
        SetupMaxTokens();
        SetupPresets();
        
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        
        LoadPreferences();
    }
    
    void SetupTemperature()
    {
        temperatureSlider.minValue = 0f;
        temperatureSlider.maxValue = 2f;
        temperatureSlider.onValueChanged.AddListener(OnTemperatureChanged);
    }
    
    void SetupMaxTokens()
    {
        maxTokensInput.onValueChanged.AddListener(OnMaxTokensChanged);
    }
    
    void SetupPresets()
    {
        string[] presets = { "Quick", "Balanced", "Detailed", "Creative" };
        presetDropdown.ClearOptions();
        presetDropdown.AddOptions(presets.ToList());
        presetDropdown.onValueChanged.AddListener(OnPresetSelected);
    }
    
    void OnTemperatureChanged(float value)
    {
        agent.Temperature = value;
        
        string mode = value switch
        {
            < 0.3f => "Precise",
            < 0.7f => "Balanced",
            < 1.2f => "Creative",
            _ => "Very Creative"
        };
        
        temperatureText.text = $"{value:F2} ({mode})";
        SavePreferences();
    }
    
    void OnMaxTokensChanged(string value)
    {
        if (int.TryParse(value, out int tokens))
        {
            agent.MaxTokens = tokens;
            SavePreferences();
        }
    }
    
    void OnPresetSelected(int index)
    {
        switch (index)
        {
            case 0: // Quick
                temperatureSlider.value = 0.3f;
                maxTokensInput.text = "100";
                break;
            case 1: // Balanced
                temperatureSlider.value = 0.7f;
                maxTokensInput.text = "500";
                break;
            case 2: // Detailed
                temperatureSlider.value = 0.5f;
                maxTokensInput.text = "2000";
                break;
            case 3: // Creative
                temperatureSlider.value = 1.0f;
                maxTokensInput.text = "1500";
                break;
        }
    }
    
    void OnResponseCompleted(Response response)
    {
        sessionTokens += response.Usage.TotalTokens;
        tokensUsedText.text = $"Session: {sessionTokens:N0} tokens";
    }
    
    void SavePreferences()
    {
        PlayerPrefs.SetFloat("Temperature", agent.Temperature ?? 0.7f);
        PlayerPrefs.SetInt("MaxTokens", agent.MaxTokens ?? 1000);
    }
    
    void LoadPreferences()
    {
        float temp = PlayerPrefs.GetFloat("Temperature", 0.7f);
        int tokens = PlayerPrefs.GetInt("MaxTokens", 1000);
        
        temperatureSlider.value = temp;
        maxTokensInput.text = tokens.ToString();
    }
}
```

## Next Steps

- [Models](models.md)
- [Voice](voice.md)
- [Context Assembly](../conversation/context-assembly.md)
