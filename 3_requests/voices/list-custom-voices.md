---
icon: user-gear
---

# List Custom Voices

Browse user-created custom voices using `.ListCustomVoices()`.

## Basic Usage

```csharp
var response = await Api.ElevenLabs
    .ListCustomVoices()
    .ExecuteAsync();

foreach (var voice in response.Voices)
{
    Debug.Log($"Custom Voice: {voice.Name}");
}
```

## What are Custom Voices?

Custom voices are voices that you or your organization have created:

- **Voice cloning** - Clone a real person's voice
- **Professional voice lab** - Professional voice creation service
- **Instant voice cloning** - Quick voice cloning from samples

## Unity Integration Examples

### Example 1: Custom Voice Manager

```csharp
public class CustomVoiceManager : MonoBehaviour
{
    [SerializeField] private TMPro.TMP_Dropdown dropdown;
    private List<VoiceData> customVoices = new();
    
    async void Start()
    {
        await LoadCustomVoices();
    }
    
    async UniTask LoadCustomVoices()
    {
        var response = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
        customVoices = response.Voices;
        
        dropdown.ClearOptions();
        
        var options = customVoices
            .Select(v => new TMPro.TMP_Dropdown.OptionData(v.Name))
            .ToList();
        
        dropdown.AddOptions(options);
    }
    
    public VoiceData GetSelectedVoice()
    {
        int index = dropdown.value;
        return customVoices[index];
    }
}
```

### Example 2: Voice Library UI

```csharp
public class VoiceLibrary : MonoBehaviour
{
    [SerializeField] private Transform builtInParent;
    [SerializeField] private Transform customParent;
    [SerializeField] private GameObject voiceItemPrefab;
    
    async void Start()
    {
        await LoadAllVoices();
    }
    
    async UniTask LoadAllVoices()
    {
        // Load built-in voices
        var builtIn = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        CreateVoiceItems(builtIn.Voices, builtInParent, isCustom: false);
        
        // Load custom voices
        var custom = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
        CreateVoiceItems(custom.Voices, customParent, isCustom: true);
    }
    
    void CreateVoiceItems(List<VoiceData> voices, Transform parent, bool isCustom)
    {
        foreach (var voice in voices)
        {
            var item = Instantiate(voiceItemPrefab, parent);
            var text = item.GetComponentInChildren<TMPro.TextMeshProUGUI>();
            
            string prefix = isCustom ? "★ " : "";
            text.text = $"{prefix}{voice.Name}\n<size=80%>{voice.Description}</size>";
        }
    }
}
```

### Example 3: Voice Type Selector

```csharp
public class VoiceTypeSelector : MonoBehaviour
{
    public enum VoiceType { BuiltIn, Custom, All }
    
    [SerializeField] private VoiceType voiceType = VoiceType.All;
    
    public async UniTask<List<VoiceData>> GetVoices()
    {
        var allVoices = new List<VoiceData>();
        
        if (voiceType == VoiceType.BuiltIn || voiceType == VoiceType.All)
        {
            var builtIn = await Api.ElevenLabs.ListVoices().ExecuteAsync();
            allVoices.AddRange(builtIn.Voices);
        }
        
        if (voiceType == VoiceType.Custom || voiceType == VoiceType.All)
        {
            var custom = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
            allVoices.AddRange(custom.Voices);
        }
        
        return allVoices;
    }
}
```

## Provider Support

### ElevenLabs

```csharp
var customVoices = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
```

**Features:**

- ✅ Voice cloning
- ✅ Professional voice lab
- ✅ Instant voice cloning
- ✅ Voice editing

### OpenAI

❌ OpenAI does not support custom voices. Only built-in voices available.

### Google

```csharp
// Limited custom voice support
var voices = await Api.Gemini.ListCustomVoices().ExecuteAsync();
```

## Combining Built-in and Custom Voices

```csharp
public class AllVoicesManager : MonoBehaviour
{
    public async UniTask<List<VoiceData>> GetAllVoices()
    {
        var allVoices = new List<VoiceData>();
        
        // Get built-in voices
        var builtInResponse = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        allVoices.AddRange(builtInResponse.Voices);
        
        // Get custom voices
        var customResponse = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
        allVoices.AddRange(customResponse.Voices);
        
        return allVoices.OrderBy(v => v.Name).ToList();
    }
    
    public async UniTask<Dictionary<string, List<VoiceData>>> GetVoicesByCategory()
    {
        return new Dictionary<string, List<VoiceData>>
        {
            ["Built-in"] = (await Api.ElevenLabs.ListVoices().ExecuteAsync()).Voices,
            ["Custom"] = (await Api.ElevenLabs.ListCustomVoices().ExecuteAsync()).Voices
        };
    }
}
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache custom voices
private List<VoiceData> cachedCustomVoices;
private float lastFetchTime;

async UniTask<List<VoiceData>> GetCustomVoicesCache()
{
    if (cachedCustomVoices == null || Time.time - lastFetchTime > 3600f)
    {
        var response = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
        cachedCustomVoices = response.Voices;
        lastFetchTime = Time.time;
    }
    return cachedCustomVoices;
}

// ✅ Handle empty results
var custom = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
if (custom.Voices.Count == 0)
{
    Debug.Log("No custom voices found");
}

// ✅ Separate UI for custom voices
CreateVoiceList(builtInVoices, "Built-in Voices");
CreateVoiceList(customVoices, "My Custom Voices");
```

### ❌ Bad Practices

```csharp
// ❌ Don't list in Update()
void Update()
{
    await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();  // NO!
}

// ❌ Don't assume custom voices exist
var voices = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
var first = voices.Voices[0];  // May fail!

// ❌ Don't fetch repeatedly
// Cache the results
```

## Error Handling

```csharp
public async UniTask<List<VoiceData>> GetCustomVoicesSafe()
{
    try
    {
        var response = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
        return response.Voices;
    }
    catch (AIApiException ex)
    {
        Debug.LogError($"API Error: {ex.Message}");
        return new List<VoiceData>();
    }
    catch (Exception ex)
    {
        Debug.LogError($"Unexpected error: {ex.Message}");
        return new List<VoiceData>();
    }
}
```

## Next Steps

- [Get Voice](get-voice.md) - Get specific voice info
- [List Voices](list-voices.md) - Browse built-in voices
- [Text to Speech](../audio/text-to-speech.md) - Use custom voices for TTS
