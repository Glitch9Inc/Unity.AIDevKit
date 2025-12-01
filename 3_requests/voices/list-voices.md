---
icon: list-music
---

# List Voices

Browse all built-in voices using `.ListVoices()`.

## Basic Usage

```csharp
var response = await Api.ElevenLabs
    .ListVoices()
    .ExecuteAsync();

foreach (var voice in response.Voices)
{
    Debug.Log($"Voice: {voice.Name} ({voice.Gender})");
}
```

## Unity Integration Examples

### Example 1: Voice Selector Dropdown

```csharp
public class VoiceSelector : MonoBehaviour
{
    [SerializeField] private TMPro.TMP_Dropdown dropdown;
    
    async void Start()
    {
        await PopulateDropdown();
    }
    
    async UniTask PopulateDropdown()
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        dropdown.ClearOptions();
        
        var options = response.Voices
            .Select(v => new TMPro.TMP_Dropdown.OptionData($"{v.Name} ({v.Gender})"))
            .ToList();
        
        dropdown.AddOptions(options);
    }
}
```

### Example 2: Filter by Gender

```csharp
public class VoiceFilter : MonoBehaviour
{
    public async UniTask<List<VoiceData>> GetMaleVoices()
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        return response.Voices
            .Where(v => v.Gender == Gender.Male)
            .ToList();
    }
    
    public async UniTask<List<VoiceData>> GetFemaleVoices()
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        return response.Voices
            .Where(v => v.Gender == Gender.Female)
            .ToList();
    }
}
```

### Example 3: Voice Browser UI

```csharp
public class VoiceBrowser : MonoBehaviour
{
    [SerializeField] private Transform contentParent;
    [SerializeField] private GameObject voiceItemPrefab;
    [SerializeField] private AudioSource previewSource;
    
    async void Start()
    {
        await LoadVoices();
    }
    
    async UniTask LoadVoices()
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        foreach (var voice in response.Voices)
        {
            var item = Instantiate(voiceItemPrefab, contentParent);
            var button = item.GetComponent<UnityEngine.UI.Button>();
            var text = item.GetComponentInChildren<TMPro.TextMeshProUGUI>();
            
            text.text = $"{voice.Name}\n<size=80%>{voice.Accent} • {voice.Gender}</size>";
            
            button.onClick.AddListener(() => PlayPreview(voice));
        }
    }
    
    async void PlayPreview(VoiceData voice)
    {
        if (string.IsNullOrEmpty(voice.PreviewUrl)) return;
        
        AudioClip preview = await LoadAudioFromUrl(voice.PreviewUrl);
        previewSource.clip = preview;
        previewSource.Play();
    }
    
    async UniTask<AudioClip> LoadAudioFromUrl(string url)
    {
        using var www = UnityEngine.Networking.UnityWebRequestMultimedia
            .GetAudioClip(url, AudioType.MPEG);
        
        await www.SendWebRequest();
        return UnityEngine.Networking.DownloadHandlerAudioClip.GetContent(www);
    }
}
```

### Example 4: Voice Recommender

```csharp
public class VoiceRecommender : MonoBehaviour
{
    public async UniTask<VoiceData> RecommendVoice(
        Gender preferredGender, 
        string preferredAccent = null)
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        var filtered = response.Voices
            .Where(v => v.Gender == preferredGender);
        
        if (!string.IsNullOrEmpty(preferredAccent))
        {
            filtered = filtered.Where(v => 
                v.Accent.Contains(preferredAccent, StringComparison.OrdinalIgnoreCase)
            );
        }
        
        return filtered.FirstOrDefault();
    }
}
```

## Provider Support

### ElevenLabs

```csharp
var voices = await Api.ElevenLabs.ListVoices().ExecuteAsync();
// Returns: Built-in professional voices
```

### OpenAI

OpenAI voices are pre-defined constants:

```csharp
// No API call needed - use constants directly
var voices = new[]
{
    OpenAIVoice.Alloy,
    OpenAIVoice.Echo,
    OpenAIVoice.Fable,
    OpenAIVoice.Onyx,
    OpenAIVoice.Nova,
    OpenAIVoice.Shimmer
};
```

### Google

```csharp
var voices = await Api.Google.ListVoices().ExecuteAsync();
// Returns: Google Cloud TTS voices
```

## Filtering Voices

```csharp
public class VoiceFilters
{
    public async UniTask<List<VoiceData>> GetAmericanVoices()
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        return response.Voices
            .Where(v => v.Accent.Contains("American"))
            .ToList();
    }
    
    public async UniTask<List<VoiceData>> GetBritishVoices()
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        return response.Voices
            .Where(v => v.Accent.Contains("British"))
            .ToList();
    }
    
    public async UniTask<VoiceData> FindVoiceByName(string name)
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        
        return response.Voices
            .FirstOrDefault(v => v.Name.Equals(name, StringComparison.OrdinalIgnoreCase));
    }
}
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache the list
private List<VoiceData> cachedVoices;
private float lastFetchTime;

async UniTask<List<VoiceData>> GetVoicesCache()
{
    if (cachedVoices == null || Time.time - lastFetchTime > 3600f)
    {
        var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
        cachedVoices = response.Voices;
        lastFetchTime = Time.time;
    }
    return cachedVoices;
}

// ✅ Filter on client side
var maleVoices = cachedVoices.Where(v => v.Gender == Gender.Male);

// ✅ Sort by name
var sorted = voices.OrderBy(v => v.Name).ToList();
```

### ❌ Bad Practices

```csharp
// ❌ Don't list in Update()
void Update()
{
    await Api.ElevenLabs.ListVoices().ExecuteAsync();  // NO!
}

// ❌ Don't fetch repeatedly
// Cache the results

// ❌ Don't ignore errors
var voices = await Api.ElevenLabs.ListVoices().ExecuteAsync();  // May fail
```

## Next Steps

- [Get Voice](get-voice.md) - Get specific voice info
- [List Custom Voices](list-custom-voices.md) - Browse user voices
- [Text to Speech](../audio/text-to-speech.md) - Use voices for TTS
