---
icon: waveform-lines
---

# Voice Change

Convert voice characteristics in audio using `.GENVoiceChange()`.

## Basic Usage

```csharp
AudioClip original = Resources.Load<AudioClip>("VoiceRecording");
AudioClip changed = await original
    .GENVoiceChange()
    .SetVoice(ElevenLabsVoice.Adam)
    .ExecuteAsync();
```

## Input Types

### AudioClip Input

```csharp
AudioClip original = Resources.Load<AudioClip>("Voice");
AudioClip changed = await original
    .GENVoiceChange()
    .SetVoice(newVoice)
    .ExecuteAsync();
```

### File Input

```csharp
var audioFile = new File<AudioClip>(audioClip, "voice.wav");
AudioClip changed = await audioFile
    .GENVoiceChange()
    .SetVoice(targetVoice)
    .ExecuteAsync();
```

## Configuration

### Target Voice

```csharp
AudioClip changed = await originalVoice
    .GENVoiceChange()
    .SetVoice(ElevenLabsVoice.Rachel)
    .ExecuteAsync();
```

### Background Noise Removal

```csharp
// Background noise removal is enabled by default (true)
AudioClip changed = await originalVoice
    .GENVoiceChange()
    .SetVoice(ElevenLabsVoice.Adam)
    .SetRemoveBackgroundNoise(true)   // default
    .ExecuteAsync();
```

### ElevenLabs Voice Settings

`Stability` and `SimilarityBoost` are properties on the request object:

```csharp
var request = originalVoice
    .GENVoiceChange()
    .SetVoice(ElevenLabsVoice.Adam);

request.Stability = 0.5;          // 0.0-1.0
request.SimilarityBoost = 0.8;    // 0.0-1.0

AudioClip changed = await request.ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Character Voice System

```csharp
public class CharacterVoiceSystem : MonoBehaviour
{
    [SerializeField] private Voice maleVoice = ElevenLabsVoice.Adam;
    [SerializeField] private Voice femaleVoice = ElevenLabsVoice.Rachel;
    
    public async UniTask<AudioClip> ApplyCharacterVoice(
        AudioClip recording, 
        CharacterGender gender)
    {
        Voice targetVoice = gender == CharacterGender.Male 
            ? maleVoice 
            : femaleVoice;
        
        return await recording
            .GENVoiceChange()
            .SetVoice(targetVoice)
            .ExecuteAsync();
    }
}

public enum CharacterGender { Male, Female }
```

### Example 2: NPC Voice Customization

```csharp
public class NPCVoiceCustomizer : MonoBehaviour
{
    private Dictionary<string, Voice> npcVoices = new();
    
    void Start()
    {
        npcVoices["shopkeeper"] = ElevenLabsVoice.Antoni;
        npcVoices["guard"] = ElevenLabsVoice.Arnold;
        npcVoices["child"] = ElevenLabsVoice.Rachel;
    }
    
    public async UniTask<AudioClip> GetNPCVoice(string npcType, AudioClip baseAudio)
    {
        if (!npcVoices.ContainsKey(npcType))
            return baseAudio;
        
        return await baseAudio
            .GENVoiceChange()
            .SetVoice(npcVoices[npcType])
            .ExecuteAsync();
    }
}
```

### Example 3: Voice Effects System

```csharp
public class VoiceEffectsSystem : MonoBehaviour
{
    public async UniTask<AudioClip> ApplyRobotVoice(AudioClip humanVoice)
    {
        // Use a deep, mechanical-sounding voice
        var req = humanVoice.GENVoiceChange().SetVoice(ElevenLabsVoice.Adam);
        req.Stability = 0.2;  // More robotic
        return await req.ExecuteAsync();
    }
    
    public async UniTask<AudioClip> ApplyGhostVoice(AudioClip normalVoice)
    {
        // Use ethereal, whispery voice
        var req = normalVoice.GENVoiceChange().SetVoice(ElevenLabsVoice.Serena);
        req.Stability = 0.8;  // More consistent
        return await req.ExecuteAsync();
    }
}
```

### Example 4: Voice Aging System

```csharp
public class VoiceAgingSystem : MonoBehaviour
{
    public async UniTask<AudioClip> AgeVoice(AudioClip youngVoice, int targetAge)
    {
        Voice targetVoice = targetAge switch
        {
            < 18 => ElevenLabsVoice.Rachel,    // Young
            < 40 => ElevenLabsVoice.Antoni,    // Adult
            < 60 => ElevenLabsVoice.Adam,      // Middle-aged
            _ => ElevenLabsVoice.Arnold         // Elderly
        };
        
        return await youngVoice
            .GENVoiceChange()
            .SetVoice(targetVoice)
            .ExecuteAsync();
    }
}
```

### Example 5: Voice Anonymizer

```csharp
public class VoiceAnonymizer : MonoBehaviour
{
    private List<Voice> anonymousVoices = new()
    {
        ElevenLabsVoice.Adam,
        ElevenLabsVoice.Antoni,
        ElevenLabsVoice.Arnold
    };
    
    public async UniTask<AudioClip> AnonymizeVoice(AudioClip originalVoice)
    {
        // Pick random voice
        Voice randomVoice = anonymousVoices[Random.Range(0, anonymousVoices.Count)];
        
        return await originalVoice
            .GENVoiceChange()
            .SetVoice(randomVoice)
            .ExecuteAsync();
    }
}
```

### Example 6: Real-time Voice Chat Modifier

```csharp
public class VoiceChatModifier : MonoBehaviour
{
    [SerializeField] private Voice desiredVoice;
    private Queue<AudioClip> processingQueue = new();
    
    public void OnMicrophoneInput(AudioClip recording)
    {
        ProcessVoiceAsync(recording).Forget();
    }
    
    async UniTaskVoid ProcessVoiceAsync(AudioClip recording)
    {
        AudioClip modified = await recording
            .GENVoiceChange()
            .SetVoice(desiredVoice)
            .ExecuteAsync();
        
        // Play modified voice
        AudioSource.PlayClipAtPoint(modified, Camera.main.transform.position);
    }
}
```

## Provider Support

### ElevenLabs

```csharp
var request = audioClip
    .GENVoiceChange()
    .SetVoice(ElevenLabsVoice.Rachel)
    .SetRemoveBackgroundNoise(true);

request.Stability = 0.5;
request.SimilarityBoost = 0.8;

AudioClip changed = await request.ExecuteAsync();
```

**Features:**

- ✅ Natural voice conversion
- ✅ Multiple target voices
- ✅ Voice characteristic control
- ✅ Preserves speech timing

**Note:** Currently, ElevenLabs is the primary provider for voice change.

## Use Cases

| Use Case | Example |
|----------|---------|
| **Character Voices** | Convert player voice to NPC voice |
| **Privacy** | Anonymize voice in recordings |
| **Localization** | Match voice to character ethnicity |
| **Effects** | Robot, ghost, monster voices |
| **Accessibility** | Convert to clearer voice |
| **Age Matching** | Make voice sound older/younger |

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache converted voices
Dictionary<string, AudioClip> cache = new();

// ✅ Choose appropriate target voice
Voice targetVoice = character.Gender == Gender.Male 
    ? ElevenLabsVoice.Adam 
    : ElevenLabsVoice.Rachel;

// ✅ Preserve original for comparison
AudioClip original = audioClip;
AudioClip modified = await original.GENVoiceChange()...;

// ✅ Clean up unused clips
Destroy(audioClip);
```

### ❌ Bad Practices

```csharp
// ❌ Don't convert in Update()
void Update()
{
    await audio.GENVoiceChange().ExecuteAsync();  // NO!
}

// ❌ Don't convert silent audio
if (audioClip.length == 0) return;  // Check first

// ❌ Don't forget cleanup
// Memory leak if clips aren't destroyed

// ❌ Don't block main thread
AudioClip clip = audio.GENVoiceChange().ExecuteAsync().Result;  // Blocks!
```

## Audio Requirements

**Supported formats:**

- WAV, MP3, M4A, FLAC, OGG

**Quality requirements:**

- Clear speech (minimal background noise)
- Good recording quality
- No heavy distortion

**Limits:**

- Max file size: 25 MB
- Duration: varies by provider

## Error Handling

```csharp
try
{
    AudioClip changed = await audioClip
        .GENVoiceChange()
        .SetVoice(targetVoice)
        .ExecuteAsync();
    
    if (changed == null || changed.length == 0)
        throw new Exception("Voice change failed");
    
    audioSource.clip = changed;
    audioSource.Play();
}
catch (AIApiException ex)
{
    Debug.LogError($"Voice change failed: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Performance Tips

```csharp
// ✅ Good - cache results
Dictionary<string, AudioClip> voiceCache = new();

async UniTask<AudioClip> GetCachedVoiceChange(AudioClip source, Voice target)
{
    string key = $"{source.name}_{target}";
    if (!voiceCache.ContainsKey(key))
    {
        voiceCache[key] = await source
            .GENVoiceChange()
            .SetVoice(target)
            .ExecuteAsync();
    }
    return voiceCache[key];
}

// ✅ Good - parallel processing
var tasks = audioClips.Select(clip =>
    clip.GENVoiceChange().SetVoice(voice).ExecuteAsync()
);
await UniTask.WhenAll(tasks);
```

## Limitations

1. **Speech Only**: Works best with clear speech, not music or effects
2. **Quality Dependent**: Input quality affects output quality
3. **Timing Preserved**: Cannot change speech speed significantly
4. **Language**: Best results with same language as target voice

## Next Steps

- [Audio Isolation](audio-isolation.md) - Clean up audio
- [Text to Speech](text-to-speech.md) - Generate speech
- [Speech to Text](speech-to-text.md) - Transcribe audio
