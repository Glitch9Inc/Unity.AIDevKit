---
icon: magnifying-glass
---

# Get Model

Retrieve information about a specific model using `.GetModel()`.

## Basic Usage

```csharp
var model = await Api.OpenAI
    .GetModel("gpt-4o")
    .ExecuteAsync();

Debug.Log($"Model ID: {model.Id}");
Debug.Log($"Created: {model.Created}");
Debug.Log($"Owner: {model.OwnedBy}");
```

## Model Properties

```csharp
public class ModelData
{
    public string Id { get; set; }           // Model identifier
    public long Created { get; set; }        // Unix timestamp
    public string OwnedBy { get; set; }      // Organization
    public ModelType Type { get; set; }      // Language, Image, Audio, etc.
}
```

## Unity Integration Examples

### Example 1: Model Info Display

```csharp
public class ModelInfoDisplay : MonoBehaviour
{
    public async UniTask ShowModelInfo(string modelId)
    {
        var model = await Api.OpenAI.GetModel(modelId).ExecuteAsync();
        
        Debug.Log($"╔══════════════════════════════════");
        Debug.Log($"║ Model: {model.Id}");
        Debug.Log($"║ Owner: {model.OwnedBy}");
        Debug.Log($"║ Created: {GetDateFromUnix(model.Created)}");
        Debug.Log($"║ Type: {model.Type}");
        Debug.Log($"╚══════════════════════════════════");
    }
    
    string GetDateFromUnix(long timestamp)
    {
        return DateTimeOffset.FromUnixTimeSeconds(timestamp).ToString();
    }
}
```

### Example 2: Model Validator

```csharp
public class ModelValidator : MonoBehaviour
{
    public async UniTask<bool> ValidateModel(string modelId)
    {
        try
        {
            var model = await Api.OpenAI.GetModel(modelId).ExecuteAsync();
            
            if (model == null)
            {
                Debug.LogWarning($"Model {modelId} not found");
                return false;
            }
            
            Debug.Log($"✓ Model {modelId} is valid");
            return true;
        }
        catch (Exception ex)
        {
            Debug.LogError($"✗ Model validation failed: {ex.Message}");
            return false;
        }
    }
}
```

### Example 3: Model Type Checker

```csharp
public class ModelTypeChecker : MonoBehaviour
{
    public async UniTask<ModelType> GetModelType(string modelId)
    {
        var model = await Api.OpenAI.GetModel(modelId).ExecuteAsync();
        return model.Type;
    }
    
    public async UniTask<bool> SupportsChat(string modelId)
    {
        var type = await GetModelType(modelId);
        return type == ModelType.Language;
    }
    
    public async UniTask<bool> SupportsImages(string modelId)
    {
        var type = await GetModelType(modelId);
        return type == ModelType.Image;
    }
}
```

## Provider Support

### OpenAI

```csharp
var model = await Api.OpenAI.GetModel("gpt-4o").ExecuteAsync();
```

### Anthropic

```csharp
var model = await Api.Anthropic.GetModel("claude-3-5-sonnet-20241022").ExecuteAsync();
```

### Google Gemini

```csharp
var model = await Api.Gemini.GetModel("gemini-1.5-pro").ExecuteAsync();
```

## Common Model IDs

### OpenAI

- `gpt-4o` - Latest GPT-4 Omni
- `gpt-4o-mini` - Smaller, faster GPT-4
- `gpt-4-turbo` - GPT-4 Turbo
- `gpt-3.5-turbo` - GPT-3.5 Turbo
- `dall-e-3` - DALL-E 3
- `whisper-1` - Whisper STT
- `tts-1` - Text-to-Speech

### Anthropic

- `claude-3-5-sonnet-20241022` - Claude 3.5 Sonnet
- `claude-3-opus-20240229` - Claude 3 Opus
- `claude-3-haiku-20240307` - Claude 3 Haiku

### Google

- `gemini-1.5-pro` - Gemini 1.5 Pro
- `gemini-1.5-flash` - Gemini 1.5 Flash
- `gemini-1.0-pro` - Gemini 1.0 Pro

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Validate before using
if (await ValidateModel("gpt-4o"))
{
    // Use model
}

// ✅ Cache model info
Dictionary<string, ModelData> modelCache = new();

// ✅ Handle errors
try
{
    var model = await Api.OpenAI.GetModel("unknown-model").ExecuteAsync();
}
catch (AIApiException ex)
{
    Debug.LogError($"Model not found: {ex.Message}");
}
```

### ❌ Bad Practices

```csharp
// ❌ Don't query in Update()
void Update()
{
    await Api.OpenAI.GetModel("gpt-4o").ExecuteAsync();  // NO!
}

// ❌ Don't ignore errors
var model = await Api.OpenAI.GetModel("unknown").ExecuteAsync();  // May fail

// ❌ Don't query repeatedly
// Cache the result instead
```

## Error Handling

```csharp
public async UniTask<ModelData> GetModelSafe(string modelId)
{
    try
    {
        return await Api.OpenAI.GetModel(modelId).ExecuteAsync();
    }
    catch (AIApiException ex)
    {
        Debug.LogError($"API Error: {ex.Message}");
        return null;
    }
    catch (Exception ex)
    {
        Debug.LogError($"Unexpected error: {ex.Message}");
        return null;
    }
}
```

## Next Steps

- [List Models](list-models.md) - Browse all available models
- [Get Custom Model](get-custom-model.md) - Get fine-tuned models
- [Fine-tuning](fine-tuning.md) - Create custom models
