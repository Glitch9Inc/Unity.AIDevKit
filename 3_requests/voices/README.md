---
icon: waveform-lines
---

# Voice Operations

AI Dev Kit provides voice management operations for browsing and managing TTS voices.

## Available Operations

### Query Operations

```csharp
// Get single voice
var voice = await Api.ElevenLabs.GetVoice("rachel").ExecuteAsync();

// List built-in voices
var voices = await Api.ElevenLabs.ListVoices().ExecuteAsync();

// List custom voices
var customVoices = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
```

## Common Use Cases

### Browse Available Voices

```csharp
public async UniTask<List<string>> GetVoiceNames()
{
    var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
    return response.Voices.Select(v => v.Name).ToList();
}
```

### Find Voice by Characteristics

```csharp
public async UniTask<Voice> FindVoice(Gender gender, string accent)
{
    var response = await Api.ElevenLabs.ListVoices().ExecuteAsync();
    
    var voice = response.Voices.FirstOrDefault(v =>
        v.Gender == gender && v.Accent.Contains(accent)
    );
    
    return voice;
}
```

### List Custom Voices

```csharp
public async UniTask<List<Voice>> GetMyCustomVoices()
{
    var response = await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
    return response.Voices;
}
```

## Provider Support

### OpenAI

```csharp
// Built-in voices only (no API, use constants)
Voice voice = OpenAIVoice.Alloy;
Voice voice = OpenAIVoice.Echo;
```

### ElevenLabs

```csharp
// Full support
await Api.ElevenLabs.GetVoice("rachel").ExecuteAsync();
await Api.ElevenLabs.ListVoices().ExecuteAsync();
await Api.ElevenLabs.ListCustomVoices().ExecuteAsync();
```

### Google

```csharp
// List voices only
await Api.Google.ListVoices().ExecuteAsync();
```

## Next Steps

- [Get Voice](get-voice.md) - Retrieve single voice
- [List Voices](list-voices.md) - Browse all voices
- [List Custom Voices](list-custom-voices.md) - Browse user voices
