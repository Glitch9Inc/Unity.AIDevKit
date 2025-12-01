---
icon: microchip
---

# Model Operations

AI Dev Kit provides model management operations for browsing and managing AI models.

## Available Operations

### Query Operations

```csharp
// Get single model
var model = await Api.OpenAI.GetModel("gpt-4o").ExecuteAsync();

// List all models
var models = await Api.OpenAI.ListModels().ExecuteAsync();
```

### Custom Model Operations

```csharp
// Get custom/fine-tuned model
var customModel = await Api.OpenAI
    .GetCustomModel("ft:gpt-4o:org:model:id")
    .ExecuteAsync();

// List custom models
var customModels = await Api.OpenAI.ListCustomModels().ExecuteAsync();

// Delete custom model
await Api.OpenAI.DeleteModel("ft:gpt-4o:org:model:id").ExecuteAsync();
```

### Fine-tuning

```csharp
// Start fine-tuning job
var job = await model
    .FineTuneModel(trainingFileId)
    .SetValidationFile(validationFileId)
    .ExecuteAsync();
```

## Common Use Cases

### Check Model Availability

```csharp
public async UniTask<bool> IsModelAvailable(string modelId)
{
    try
    {
        var model = await Api.OpenAI.GetModel(modelId).ExecuteAsync();
        return model != null;
    }
    catch
    {
        return false;
    }
}
```

### List Available Models

```csharp
public async UniTask<List<string>> GetAvailableModels()
{
    var response = await Api.OpenAI.ListModels().ExecuteAsync();
    return response.Data.Select(m => m.Id).ToList();
}
```

### Manage Fine-tuned Models

```csharp
public async UniTask<List<string>> GetMyFineTunedModels()
{
    var response = await Api.OpenAI.ListCustomModels().ExecuteAsync();
    return response.Data.Select(m => m.Id).ToList();
}
```

## Provider Support

### OpenAI

```csharp
// Full support
await Api.OpenAI.GetModel("gpt-4o").ExecuteAsync();
await Api.OpenAI.ListModels().ExecuteAsync();
await Api.OpenAI.GetCustomModel("ft:...").ExecuteAsync();
await Api.OpenAI.ListCustomModels().ExecuteAsync();
await Api.OpenAI.DeleteModel("ft:...").ExecuteAsync();
```

### Google Gemini

```csharp
// List models only
await Api.Gemini.ListModels().ExecuteAsync();
```

### Anthropic

```csharp
// List models only
await Api.Anthropic.ListModels().ExecuteAsync();
```

## Next Steps

- [Get Model](get-model.md) - Retrieve single model
- [List Models](list-models.md) - Browse all models
- [Get Custom Model](get-custom-model.md) - Retrieve fine-tuned model
- [List Custom Models](list-custom-models.md) - Browse fine-tuned models
- [Delete Model](delete-model.md) - Remove custom model
- [Fine-tuning](fine-tuning.md) - Create fine-tuned models
