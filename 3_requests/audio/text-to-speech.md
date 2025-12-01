---
icon: microphone-lines
---

# Text to Speech

Generate natural-sounding speech from text using `.GENSpeech()` or `.TextToSpeech()`.

## Basic Usage

```csharp
AudioClip speech = await "Welcome back, Commander!"
    .GENSpeech()
    .ExecuteAsync();

audioSource.clip = speech;
audioSource.Play();
```

## Input Types

### String Input

```csharp
AudioClip speech = await "Hello, world!"
    .GENSpeech()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("The {character} says: {dialogue}");
AudioClip speech = await prompt
    .GENSpeech()
    .ExecuteAsync();
```

### Alias Method

```csharp
// TextToSpeech() is an alias for GENSpeech()
AudioClip speech = await "Hello"
    .TextToSpeech()
    .ExecuteAsync();
```

## Configuration

### Voice Selection

```csharp
// OpenAI voices
AudioClip speech = await "Hello"
    .GENSpeech()
    .SetVoice(OpenAIVoice.Alloy)
    .ExecuteAsync();

// ElevenLabs voices
AudioClip speech = await "Hello"
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Rachel)
    .ExecuteAsync();
```

**Available OpenAI Voices:**

- `OpenAIVoice.Alloy` - Neutral, balanced
- `OpenAIVoice.Echo` - Male, clear
- `OpenAIVoice.Fable` - British, expressive
- `OpenAIVoice.Onyx` - Deep, authoritative
- `OpenAIVoice.Nova` - Energetic, young
- `OpenAIVoice.Shimmer` - Soft, gentle

**Available ElevenLabs Voices:**

- `ElevenLabsVoice.Rachel` - Calm, natural
- `ElevenLabsVoice.Adam` - Deep, confident
- `ElevenLabsVoice.Antoni` - Warm, friendly
- `ElevenLabsVoice.Arnold` - Mature, strong
- And many more...

### Model Selection

```csharp
// OpenAI TTS models
AudioClip speech = await "Hello"
    .GENSpeech()
    .SetModel(OpenAIModel.TTS1)        // Standard quality
    .ExecuteAsync();

AudioClip hd = await "Hello"
    .GENSpeech()
    .SetModel(OpenAIModel.TTS1HD)      // HD quality
    .ExecuteAsync();

// ElevenLabs models
AudioClip turbo = await "Hello"
    .GENSpeech()
    .SetModel(ElevenLabsModel.Turbo)   // Fast
    .ExecuteAsync();
```

### Voice Settings (ElevenLabs)

```csharp
AudioClip speech = await "Your text"
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Rachel)
    .SetStability(0.5f)                 // 0.0-1.0
    .SetSimilarityBoost(0.8f)          // 0.0-1.0
    .SetStyle(0.5f)                     // 0.0-1.0 (optional)
    .ExecuteAsync();
```

**Parameters:**

- **Stability** (0.0-1.0): Higher = more consistent, Lower = more expressive
- **Similarity Boost** (0.0-1.0): How closely to match the original voice
- **Style** (0.0-1.0): Style exaggeration (model-dependent)

### Audio Format

```csharp
AudioClip speech = await "Hello"
    .GENSpeech()
    .SetFormat(AudioFormat.MP3)        // MP3, WAV, OGG
    .ExecuteAsync();
```

### Speed Control

```csharp
// OpenAI speed control (0.25x to 4.0x)
AudioClip fast = await "Hello"
    .GENSpeech()
    .SetSpeed(1.5f)
    .ExecuteAsync();

AudioClip slow = await "Hello"
    .GENSpeech()
    .SetSpeed(0.75f)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: NPC Dialogue

```csharp
public class NPCController : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private Voice npcVoice = ElevenLabsVoice.Rachel;
    
    public async UniTask Say(string dialogue)
    {
        AudioClip speech = await dialogue
            .GENSpeech()
            .SetVoice(npcVoice)
            .ExecuteAsync();
        
        audioSource.clip = speech;
        audioSource.Play();
        
        // Wait for speech to finish
        await UniTask.WaitUntil(() => !audioSource.isPlaying);
    }
}

// Usage
await npc.Say("Greetings, traveler!");
```

### Example 2: Tutorial Narrator

```csharp
public class TutorialNarrator : MonoBehaviour
{
    [SerializeField] private AudioSource narratorSource;
    private Queue<string> narrationQueue = new();
    
    public void QueueNarration(string text)
    {
        narrationQueue.Enqueue(text);
        if (!narratorSource.isPlaying)
            PlayNextNarration().Forget();
    }
    
    private async UniTaskVoid PlayNextNarration()
    {
        while (narrationQueue.Count > 0)
        {
            string text = narrationQueue.Dequeue();
            
            AudioClip speech = await text
                .GENSpeech()
                .SetVoice(OpenAIVoice.Fable)
                .ExecuteAsync();
            
            narratorSource.clip = speech;
            narratorSource.Play();
            
            await UniTask.WaitUntil(() => !narratorSource.isPlaying);
        }
    }
}
```

### Example 3: Dynamic UI Feedback

```csharp
public class UIAudioFeedback : MonoBehaviour
{
    private Dictionary<string, AudioClip> speechCache = new();
    
    public async UniTask SpeakFeedback(string feedback)
    {
        // Check cache first
        if (!speechCache.ContainsKey(feedback))
        {
            AudioClip speech = await feedback
                .GENSpeech()
                .SetVoice(OpenAIVoice.Nova)
                .SetSpeed(1.2f)
                .ExecuteAsync();
            
            speechCache[feedback] = speech;
        }
        
        AudioSource.PlayClipAtPoint(speechCache[feedback], Camera.main.transform.position);
    }
}

// Usage
await uiFeedback.SpeakFeedback("Button clicked");
await uiFeedback.SpeakFeedback("Settings saved");
```

### Example 4: Accessibility Reader

```csharp
public class AccessibilityReader : MonoBehaviour
{
    [SerializeField] private bool autoReadUI = true;
    private AudioSource readerSource;
    
    void Start()
    {
        readerSource = gameObject.AddComponent<AudioSource>();
    }
    
    public async UniTask ReadText(string text)
    {
        if (string.IsNullOrEmpty(text)) return;
        
        // Stop current speech
        readerSource.Stop();
        
        AudioClip speech = await text
            .GENSpeech()
            .SetVoice(OpenAIVoice.Shimmer)
            .SetSpeed(0.9f)
            .ExecuteAsync();
        
        readerSource.clip = speech;
        readerSource.Play();
    }
    
    public void StopReading()
    {
        readerSource.Stop();
    }
}
```

### Example 5: Multi-Language Support

```csharp
public class MultiLanguageTTS : MonoBehaviour
{
    private SystemLanguage currentLanguage;
    
    public async UniTask<AudioClip> GenerateSpeech(string text, SystemLanguage language)
    {
        Voice voice = GetVoiceForLanguage(language);
        
        return await text
            .GENSpeech()
            .SetVoice(voice)
            .ExecuteAsync();
    }
    
    private Voice GetVoiceForLanguage(SystemLanguage language)
    {
        return language switch
        {
            SystemLanguage.English => OpenAIVoice.Alloy,
            SystemLanguage.Spanish => ElevenLabsVoice.Antoni,
            SystemLanguage.French => ElevenLabsVoice.Rachel,
            SystemLanguage.German => ElevenLabsVoice.Adam,
            _ => OpenAIVoice.Alloy
        };
    }
}
```

### Example 6: Subtitle Generator

```csharp
public class SubtitledSpeech : MonoBehaviour
{
    [SerializeField] private TMPro.TextMeshProUGUI subtitleText;
    [SerializeField] private AudioSource audioSource;
    
    public async UniTask SpeakWithSubtitles(string text)
    {
        // Generate speech
        AudioClip speech = await text
            .GENSpeech()
            .ExecuteAsync();
        
        // Show subtitle
        subtitleText.text = text;
        subtitleText.gameObject.SetActive(true);
        
        // Play audio
        audioSource.clip = speech;
        audioSource.Play();
        
        // Wait for completion
        await UniTask.WaitUntil(() => !audioSource.isPlaying);
        
        // Hide subtitle
        subtitleText.gameObject.SetActive(false);
    }
}
```

## Provider Support

### OpenAI

```csharp
AudioClip speech = await "Hello"
    .GENSpeech()
    .SetModel(OpenAIModel.TTS1)
    .SetVoice(OpenAIVoice.Alloy)
    .SetSpeed(1.0f)
    .ExecuteAsync();
```

**Features:**

- ✅ Multiple voices
- ✅ Speed control
- ✅ HD quality option
- ✅ Low latency

### ElevenLabs

```csharp
AudioClip speech = await "Hello"
    .GENSpeech()
    .SetModel(ElevenLabsModel.Turbo)
    .SetVoice(ElevenLabsVoice.Rachel)
    .SetStability(0.5f)
    .SetSimilarityBoost(0.8f)
    .ExecuteAsync();
```

**Features:**

- ✅ Highly natural voices
- ✅ Voice cloning
- ✅ Fine-grained control
- ✅ Emotional range

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache frequently used phrases
Dictionary<string, AudioClip> cache = new();

// ✅ Clean up audio clips
Destroy(audioClip);

// ✅ Handle long text
string[] sentences = text.Split('.');
foreach (var sentence in sentences)
{
    await sentence.GENSpeech().ExecuteAsync();
}

// ✅ Pre-generate common phrases
await PreloadPhrases(new[] { "Hello", "Goodbye", "Thank you" });
```

### ❌ Bad Practices

```csharp
// ❌ Don't generate in Update()
void Update()
{
    await text.GENSpeech().ExecuteAsync();  // NO!
}

// ❌ Don't forget to clean up
// Memory leak if clips aren't destroyed

// ❌ Don't generate extremely long text
await longNovel.GENSpeech().ExecuteAsync();  // Will fail

// ❌ Don't block main thread
AudioClip clip = text.GENSpeech().ExecuteAsync().Result;  // Blocks!
```

## Performance Tips

```csharp
// ✅ Good - cache and reuse
private Dictionary<string, AudioClip> speechCache = new();

async UniTask<AudioClip> GetCachedSpeech(string text)
{
    if (!speechCache.ContainsKey(text))
    {
        speechCache[text] = await text.GENSpeech().ExecuteAsync();
    }
    return speechCache[text];
}

// ✅ Good - use faster models for UI
await "Button clicked"
    .GENSpeech()
    .SetModel(ElevenLabsModel.Turbo)
    .ExecuteAsync();

// ✅ Good - preload important audio
async UniTask PreloadDialogue(string[] lines)
{
    var tasks = lines.Select(line => line.GENSpeech().ExecuteAsync());
    await UniTask.WhenAll(tasks);
}
```

## Error Handling

```csharp
try
{
    AudioClip speech = await text
        .GENSpeech()
        .ExecuteAsync();
    
    if (speech == null || speech.length == 0)
        throw new Exception("Invalid audio clip");
    
    audioSource.clip = speech;
    audioSource.Play();
}
catch (AIApiException ex)
{
    Debug.LogError($"TTS failed: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Limitations

1. **Text Length**: Most providers have character limits (2000-5000 chars)
2. **Rate Limits**: API calls per minute may be limited
3. **Cost**: Longer text = higher cost
4. **Real-time**: Not suitable for real-time dialogue (use Realtime API instead)

## Next Steps

- [Speech to Text](speech-to-text.md) - Transcription
- [Speech Translation](speech-translation.md) - Translation
- [Voice Change](voice-change.md) - Voice modification
