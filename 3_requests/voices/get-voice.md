---
icon: user-music
---

# Get Voice

Retrieve information about a specific voice using `.GetVoice()`.

## Basic Usage

```csharp
var voice = await Api.ElevenLabs
    .GetVoice("rachel")
    .ExecuteAsync();

Debug.Log($"Voice: {voice.Name}");
Debug.Log($"Gender: {voice.Gender}");
Debug.Log($"Accent: {voice.Accent}");
```

## Voice Properties

```csharp
public class VoiceData
{
    public string Id { get; set; }
    public string Name { get; set; }
    public Gender Gender { get; set; }
    public string Accent { get; set; }
    public string Description { get; set; }
    public string PreviewUrl { get; set; }
}
```

## Unity Integration Examples

### Example 1: Voice Info Display

```csharp
public class VoiceInfoDisplay : MonoBehaviour
{
    public async UniTask ShowVoiceInfo(string voiceId)
    {
        var voice = await Api.ElevenLabs.GetVoice(voiceId).ExecuteAsync();
        
        Debug.Log($"╔══════════════════════════════════");
        Debug.Log($"║ Voice: {voice.Name}");
        Debug.Log($"║ Gender: {voice.Gender}");
        Debug.Log($"║ Accent: {voice.Accent}");
        Debug.Log($"║ Description: {voice.Description}");
        Debug.Log($"╚══════════════════════════════════");
    }
}
```

### Example 2: Voice Preview Player

```csharp
public class VoicePreviewPlayer : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    
    public async UniTask PlayVoicePreview(string voiceId)
    {
        var voice = await Api.ElevenLabs.GetVoice(voiceId).ExecuteAsync();
        
        if (!string.IsNullOrEmpty(voice.PreviewUrl))
        {
            AudioClip preview = await LoadAudioFromUrl(voice.PreviewUrl);
            audioSource.clip = preview;
            audioSource.Play();
        }
    }
    
    async UniTask<AudioClip> LoadAudioFromUrl(string url)
    {
        using var www = UnityEngine.Networking.UnityWebRequestMultimedia
            .GetAudioClip(url, AudioType.MPEG);
        
        await www.SendWebRequest();
        return UnityEngine.Networking.DownloadHandlerAudioClip
            .GetContent(www);
    }
}
```

### Example 3: Voice Validator

```csharp
public class VoiceValidator : MonoBehaviour
{
    public async UniTask<bool> ValidateVoice(string voiceId)
    {
        try
        {
            var voice = await Api.ElevenLabs.GetVoice(voiceId).ExecuteAsync();
            
            if (voice == null)
            {
                Debug.LogWarning($"Voice {voiceId} not found");
                return false;
            }
            
            Debug.Log($"✓ Voice {voiceId} is valid");
            return true;
        }
        catch (Exception ex)
        {
            Debug.LogError($"✗ Voice validation failed: {ex.Message}");
            return false;
        }
    }
}
```

## Provider Support

### ElevenLabs

```csharp
var voice = await Api.ElevenLabs.GetVoice("rachel").ExecuteAsync();
```

**Available voices:**

- `rachel` - Calm American female
- `adam` - Deep American male
- `antoni` - Well-rounded American male
- `arnold` - Crisp American male
- `bella` - Soft American female
- `domi` - Strong American female
- `elli` - Emotional American female
- `josh` - Deep American male
- `sam` - Raspy American male

### OpenAI

OpenAI voices are constants, not retrieved via API:

```csharp
// Use directly without API call
Voice voice = OpenAIVoice.Alloy;
Voice voice = OpenAIVoice.Echo;
Voice voice = OpenAIVoice.Fable;
Voice voice = OpenAIVoice.Onyx;
Voice voice = OpenAIVoice.Nova;
Voice voice = OpenAIVoice.Shimmer;
```

### Google

```csharp
var voice = await Api.Gemini.GetVoice("en-US-Standard-A").ExecuteAsync();
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Validate before using
if (await ValidateVoice("rachel"))
{
    // Use voice
}

// ✅ Cache voice info
Dictionary<string, VoiceData> voiceCache = new();

async UniTask<VoiceData> GetCachedVoice(string voiceId)
{
    if (!voiceCache.ContainsKey(voiceId))
    {
        voiceCache[voiceId] = await Api.ElevenLabs
            .GetVoice(voiceId)
            .ExecuteAsync();
    }
    return voiceCache[voiceId];
}

// ✅ Handle errors
try
{
    var voice = await Api.ElevenLabs.GetVoice("unknown").ExecuteAsync();
}
catch (AIApiException ex)
{
    Debug.LogError($"Voice not found: {ex.Message}");
}
```

### ❌ Bad Practices

```csharp
// ❌ Don't query in Update()
void Update()
{
    await Api.ElevenLabs.GetVoice("rachel").ExecuteAsync();  // NO!
}

// ❌ Don't ignore errors
var voice = await Api.ElevenLabs.GetVoice("unknown").ExecuteAsync();

// ❌ Don't query repeatedly
// Cache the result instead
```

## Error Handling

```csharp
public async UniTask<VoiceData> GetVoiceSafe(string voiceId)
{
    try
    {
        return await Api.ElevenLabs.GetVoice(voiceId).ExecuteAsync();
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

- [List Voices](list-voices.md) - Browse all voices
- [List Custom Voices](list-custom-voices.md) - Browse user voices
- [Text to Speech](../audio/text-to-speech.md) - Use voices for TTS
