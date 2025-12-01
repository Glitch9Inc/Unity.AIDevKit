---
icon: list
---

# List Models

Browse all available models using `.ListModels()`.

## Basic Usage

```csharp
var response = await Api.OpenAI
    .ListModels()
    .ExecuteAsync();

foreach (var model in response.Data)
{
    Debug.Log($"Model: {model.Id}");
}
```

## Unity Integration Examples

### Example 1: Model Selector

```csharp
public class ModelSelector : MonoBehaviour
{
    [SerializeField] private TMPro.TMP_Dropdown dropdown;
    
    async void Start()
    {
        await PopulateDropdown();
    }
    
    async UniTask PopulateDropdown()
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        
        dropdown.ClearOptions();
        
        var options = response.Data
            .Select(m => new TMPro.TMP_Dropdown.OptionData(m.Id))
            .ToList();
        
        dropdown.AddOptions(options);
    }
}
```

### Example 2: Filter by Type

```csharp
public class ModelFilter : MonoBehaviour
{
    public async UniTask<List<ModelData>> GetLanguageModels()
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        
        return response.Data
            .Where(m => m.Type == ModelType.Language)
            .ToList();
    }
    
    public async UniTask<List<ModelData>> GetImageModels()
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        
        return response.Data
            .Where(m => m.Type == ModelType.Image)
            .ToList();
    }
}
```

### Example 3: Model Browser UI

```csharp
public class ModelBrowser : MonoBehaviour
{
    [SerializeField] private Transform contentParent;
    [SerializeField] private GameObject modelItemPrefab;
    
    async void Start()
    {
        await LoadModels();
    }
    
    async UniTask LoadModels()
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        
        foreach (var model in response.Data)
        {
            var item = Instantiate(modelItemPrefab, contentParent);
            var text = item.GetComponentInChildren<TMPro.TextMeshProUGUI>();
            text.text = $"{model.Id}\n<size=80%>{model.OwnedBy}</size>";
        }
    }
}
```

## Provider Support

### OpenAI

```csharp
var models = await Api.OpenAI.ListModels().ExecuteAsync();
// Returns: GPT-4, GPT-3.5, DALL-E, Whisper, TTS, etc.
```

### Anthropic

```csharp
var models = await Api.Anthropic.ListModels().ExecuteAsync();
// Returns: Claude 3.5, Claude 3, etc.
```

### Google Gemini

```csharp
var models = await Api.Google.ListModels().ExecuteAsync();
// Returns: Gemini 1.5 Pro, Flash, etc.
```

### OpenRouter

```csharp
var models = await Api.OpenRouter.ListModels().ExecuteAsync();
// Returns: Various models from multiple providers
```

## Filtering Models

```csharp
public class ModelFilters
{
    public async UniTask<List<ModelData>> GetGPT4Models()
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        
        return response.Data
            .Where(m => m.Id.Contains("gpt-4"))
            .ToList();
    }
    
    public async UniTask<List<ModelData>> GetRecentModels(int months = 6)
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        var cutoff = DateTimeOffset.UtcNow.AddMonths(-months).ToUnixTimeSeconds();
        
        return response.Data
            .Where(m => m.Created >= cutoff)
            .OrderByDescending(m => m.Created)
            .ToList();
    }
}
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache the list
private List<ModelData> cachedModels;
private float lastFetchTime;

async UniTask<List<ModelData>> GetModelsCache()
{
    if (cachedModels == null || Time.time - lastFetchTime > 3600f)
    {
        var response = await Api.OpenAI.ListModels().ExecuteAsync();
        cachedModels = response.Data;
        lastFetchTime = Time.time;
    }
    return cachedModels;
}

// ✅ Filter on client side
var languageModels = cachedModels.Where(m => m.Type == ModelType.Language);

// ✅ Sort by relevance
var sorted = models.OrderByDescending(m => m.Created).ToList();
```

### ❌ Bad Practices

```csharp
// ❌ Don't list in Update()
void Update()
{
    await Api.OpenAI.ListModels().ExecuteAsync();  // NO!
}

// ❌ Don't fetch repeatedly
// Cache the results

// ❌ Don't ignore pagination (if provider supports it)
```

## Next Steps

- [Get Model](get-model.md) - Get specific model info
- [List Custom Models](list-custom-models.md) - Browse fine-tuned models
