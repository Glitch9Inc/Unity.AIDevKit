---
icon: volume-high
---

# Audio Generation

AI Dev Kit provides comprehensive audio generation capabilities including speech synthesis, transcription, translation, and audio effects.

## Available Methods

### 1. Text to Speech (`.GENSpeech()`)

Convert text to natural-sounding speech:

```csharp
AudioClip speech = await "Welcome back, Commander!"
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Rachel)
    .ExecuteAsync();
```

**Best for:**

- ✅ Voice-overs and narration
- ✅ NPC dialogue
- ✅ UI feedback
- ✅ Accessibility features

### 2. Speech to Text (`.GENTranscript()`)

Transcribe audio to text:

```csharp
string transcript = await audioClip
    .GENTranscript()
    .ExecuteAsync();
```

**Best for:**

- ✅ Voice commands
- ✅ Speech recognition
- ✅ Dictation
- ✅ Audio file transcription

### 3. Speech Translation (`.GENTranslation()`)

Translate speech to English:

```csharp
string english = await foreignAudioClip
    .GENTranslation()
    .ExecuteAsync();
```

**Best for:**

- ✅ Multi-language support
- ✅ Translation services
- ✅ Accessibility

### 4. Sound Effects (`.GENSoundEffect()`)

Generate sound effects from text:

```csharp
AudioClip sfx = await "Dog barking"
    .GENSoundEffect()
    .ExecuteAsync();
```

**Best for:**

- ✅ Dynamic SFX generation
- ✅ Prototyping
- ✅ Asset creation

### 5. Voice Change (`.GENVoiceChange()`)

Convert voice to different style:

```csharp
AudioClip changed = await originalVoice
    .GENVoiceChange()
    .SetTargetVoice(newVoice)
    .ExecuteAsync();
```

**Best for:**

- ✅ Voice effects
- ✅ Character voices
- ✅ Voice modification

### 6. Audio Isolation (`.GENAudioIsolation()`)

Isolate or enhance audio elements:

```csharp
AudioClip isolated = await noisyAudio
    .GENAudioIsolation()
    .ExecuteAsync();
```

**Best for:**

- ✅ Noise reduction
- ✅ Voice isolation
- ✅ Audio cleanup

## Quick Comparison

| Method | Input | Output | Providers |
|--------|-------|--------|-----------|
| `GENSpeech()` | Text | AudioClip | OpenAI, ElevenLabs |
| `GENTranscript()` | AudioClip | String | OpenAI, Google |
| `GENTranslation()` | AudioClip | String | OpenAI |
| `GENSoundEffect()` | Text | AudioClip | ElevenLabs |
| `GENVoiceChange()` | AudioClip | AudioClip | ElevenLabs |
| `GENAudioIsolation()` | AudioClip | AudioClip | ElevenLabs |

## Basic Examples

### Example 1: Simple TTS

```csharp
AudioClip greeting = await "Hello, player!"
    .GENSpeech()
    .ExecuteAsync();

audioSource.clip = greeting;
audioSource.Play();
```

### Example 2: Transcribe Microphone

```csharp
AudioClip recording = Microphone.Start(null, false, 10, 44100);
await new WaitForSeconds(5);
Microphone.End(null);

string text = await recording
    .GENTranscript()
    .ExecuteAsync();

Debug.Log($"You said: {text}");
```

### Example 3: Generate Game SFX

```csharp
AudioClip explosion = await "Massive explosion sound"
    .GENSoundEffect()
    .ExecuteAsync();

AudioClip footsteps = await "Footsteps on gravel"
    .GENSoundEffect()
    .ExecuteAsync();
```

## Configuration Options

### Text to Speech

```csharp
AudioClip speech = await "Your text"
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Rachel)     // Voice selection
    .SetModel(ElevenLabsModel.Turbo)     // Model selection
    .SetStability(0.5f)                   // Voice stability
    .SetSimilarityBoost(0.8f)             // Voice similarity
    .ExecuteAsync();
```

### Speech to Text

```csharp
string transcript = await audioClip
    .GENTranscript()
    .SetModel(OpenAIModel.Whisper1)       // Model selection
    .SetLanguage(SystemLanguage.English)  // Source language
    .SetPrompt("Technical context")       // Context hint
    .ExecuteAsync();
```

## Provider Support

### OpenAI

```csharp
// TTS
AudioClip speech = await "text"
    .GENSpeech()
    .SetModel(OpenAIModel.TTS1)
    .SetVoice(OpenAIVoice.Alloy)
    .ExecuteAsync();

// STT
string text = await audioClip
    .GENTranscript()
    .SetModel(OpenAIModel.Whisper1)
    .ExecuteAsync();

// Translation
string english = await audioClip
    .GENTranslation()
    .ExecuteAsync();
```

### ElevenLabs

```csharp
// TTS
AudioClip speech = await "text"
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Rachel)
    .ExecuteAsync();

// SFX
AudioClip sfx = await "explosion"
    .GENSoundEffect()
    .ExecuteAsync();

// Voice Change
AudioClip changed = await audioClip
    .GENVoiceChange()
    .SetTargetVoice(ElevenLabsVoice.Adam)
    .ExecuteAsync();
```

### Google

```csharp
// STT
string text = await audioClip
    .GENTranscript()
    .SetModel(GoogleModel.ChirpV2)
    .ExecuteAsync();
```

## Common Workflows

### Workflow 1: NPC Dialogue System

```csharp
public class NPCDialogue : MonoBehaviour
{
    private AudioSource audioSource;
    
    async UniTask SayLine(string text)
    {
        AudioClip speech = await text
            .GENSpeech()
            .SetVoice(ElevenLabsVoice.Rachel)
            .ExecuteAsync();
        
        audioSource.clip = speech;
        audioSource.Play();
        
        await UniTask.WaitUntil(() => !audioSource.isPlaying);
    }
}
```

### Workflow 2: Voice Command System

```csharp
public class VoiceCommands : MonoBehaviour
{
    async UniTask ListenForCommand()
    {
        AudioClip recording = await RecordAudio(5);
        
        string command = await recording
            .GENTranscript()
            .ExecuteAsync();
        
        ProcessCommand(command);
    }
}
```

### Workflow 3: Dynamic SFX Generator

```csharp
public class SFXGenerator : MonoBehaviour
{
    private Dictionary<string, AudioClip> cache = new();
    
    async UniTask<AudioClip> GetSFX(string description)
    {
        if (cache.ContainsKey(description))
            return cache[description];
        
        AudioClip sfx = await description
            .GENSoundEffect()
            .ExecuteAsync();
        
        cache[description] = sfx;
        return sfx;
    }
}
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache audio clips
Dictionary<string, AudioClip> speechCache = new();

// ✅ Use appropriate quality settings
await text.GENSpeech()
    .SetModel(ElevenLabsModel.Turbo)  // Faster for UI
    .ExecuteAsync();

// ✅ Handle microphone permissions
if (!Application.HasUserAuthorization(UserAuthorization.Microphone))
{
    await Application.RequestUserAuthorization(UserAuthorization.Microphone);
}

// ✅ Clean up audio clips
Destroy(audioClip);
```

### ❌ Bad Practices

```csharp
// ❌ Don't generate speech in Update()
void Update()
{
    await text.GENSpeech().ExecuteAsync();  // NO!
}

// ❌ Don't forget to dispose
// Memory leak if clips aren't cleaned up

// ❌ Don't block main thread
string text = clip.GENTranscript().ExecuteAsync().Result;  // Blocks!
```

## Next Steps

- [Text to Speech](text-to-speech.md) - Voice synthesis guide
- [Speech to Text](speech-to-text.md) - Transcription guide
- [Speech Translation](speech-translation.md) - Translation guide
- [Sound Effects](sound-effects.md) - SFX generation
- [Voice Change](voice-change.md) - Voice modification
- [Audio Isolation](audio-isolation.md) - Audio cleanup
