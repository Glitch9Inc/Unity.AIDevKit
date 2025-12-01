---
icon: language
---

# Speech Translation

Translate speech from any language to English using `.GENTranslation()` or `.SpeechToEnglish()`.

## Basic Usage

```csharp
AudioClip foreignAudio = Resources.Load<AudioClip>("Spanish");
string english = await foreignAudio
    .GENTranslation()
    .ExecuteAsync();

Debug.Log($"Translation: {english}");
```

## Input Types

### AudioClip Input

```csharp
AudioClip audio = Resources.Load<AudioClip>("ForeignSpeech");
string english = await audio
    .GENTranslation()
    .ExecuteAsync();
```

### File Input

```csharp
var audioFile = new File<AudioClip>(audioClip, "recording.mp3");
string english = await audioFile
    .GENTranslation()
    .ExecuteAsync();
```

### Alias Method

```csharp
// SpeechToEnglish() is an alias for GENTranslation()
string english = await audioClip
    .SpeechToEnglish()
    .ExecuteAsync();
```

## Configuration

### Model Selection

```csharp
string english = await audioClip
    .GENTranslation()
    .SetModel(OpenAIModel.Whisper1)
    .ExecuteAsync();
```

### Context Prompt

Provide context for better translation:

```csharp
string english = await audioClip
    .GENTranslation()
    .SetPrompt("Gaming terminology: player, level, quest")
    .ExecuteAsync();
```

### Temperature

```csharp
string english = await audioClip
    .GENTranslation()
    .SetTemperature(0.0f)  // More consistent
    .ExecuteAsync();
```

## Supported Languages

OpenAI Whisper supports translation from **99+ languages** to English:

- Spanish, French, German, Italian, Portuguese
- Chinese (Mandarin, Cantonese), Japanese, Korean
- Russian, Arabic, Turkish, Polish
- Dutch, Swedish, Norwegian, Danish
- And many more...

**Note:** Translation is always **to English**. For other target languages, use `.GENTranscript()` + separate translation API.

## Unity Integration Examples

### Example 1: Multi-Language Voice Chat

```csharp
public class MultiLanguageChat : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    
    public async UniTask TranslateAndPlay(AudioClip foreignAudio)
    {
        // Translate to English
        string english = await foreignAudio
            .GENTranslation()
            .ExecuteAsync();
        
        Debug.Log($"Translated: {english}");
        
        // Generate English speech
        AudioClip englishSpeech = await english
            .GENSpeech()
            .SetVoice(OpenAIVoice.Alloy)
            .ExecuteAsync();
        
        // Play translation
        audioSource.clip = englishSpeech;
        audioSource.Play();
    }
}
```

### Example 2: Real-time Translation System

```csharp
public class RealtimeTranslator : MonoBehaviour
{
    [SerializeField] private TMPro.TextMeshProUGUI subtitleText;
    
    public async UniTask StartTranslation()
    {
        // Record audio
        AudioClip recording = Microphone.Start(null, false, 10, 44100);
        await UniTask.Delay(5000);
        Microphone.End(null);
        
        // Translate
        string english = await recording
            .GENTranslation()
            .ExecuteAsync();
        
        // Display
        subtitleText.text = english;
    }
}
```

### Example 3: Multi-Language Game Tutorial

```csharp
public class TutorialTranslator : MonoBehaviour
{
    private Dictionary<string, string> translationCache = new();
    
    public async UniTask<string> GetEnglishTutorial(AudioClip foreignTutorial)
    {
        string key = foreignTutorial.name;
        
        if (!translationCache.ContainsKey(key))
        {
            string english = await foreignTutorial
                .GENTranslation()
                .SetPrompt("Game tutorial: jump, move, attack, defend")
                .ExecuteAsync();
            
            translationCache[key] = english;
        }
        
        return translationCache[key];
    }
}
```

### Example 4: International Customer Support

```csharp
public class SupportTicketTranslator : MonoBehaviour
{
    public async UniTask<string> TranslateCustomerIssue(AudioClip issueRecording)
    {
        string english = await issueRecording
            .GENTranslation()
            .SetPrompt("Customer support: bug, error, crash, issue")
            .ExecuteAsync();
        
        // Log for support team
        Debug.Log($"Customer Issue (English): {english}");
        
        return english;
    }
}
```

### Example 5: Language Learning Assistant

```csharp
public class LanguageLearning : MonoBehaviour
{
    public async UniTask CheckPronunciation(AudioClip studentSpeech)
    {
        // Get English translation
        string english = await studentSpeech
            .GENTranslation()
            .ExecuteAsync();
        
        // Also get original language transcript
        string original = await studentSpeech
            .GENTranscript()
            .ExecuteAsync();
        
        Debug.Log($"Original: {original}");
        Debug.Log($"English: {english}");
        
        // Compare and provide feedback
        ProvideFeedback(original, english);
    }
    
    void ProvideFeedback(string original, string english)
    {
        // Implementation
    }
}
```

### Example 6: Conference Call Translator

```csharp
public class ConferenceTranslator : MonoBehaviour
{
    private List<TranslationEntry> translations = new();
    
    [System.Serializable]
    public class TranslationEntry
    {
        public string timestamp;
        public string speaker;
        public string translation;
    }
    
    public async UniTask TranslateConferenceAudio(AudioClip segment, string speakerName)
    {
        string english = await segment
            .GENTranslation()
            .SetPrompt("Business meeting terminology")
            .ExecuteAsync();
        
        translations.Add(new TranslationEntry
        {
            timestamp = DateTime.Now.ToString("HH:mm:ss"),
            speaker = speakerName,
            translation = english
        });
        
        Debug.Log($"[{DateTime.Now:HH:mm:ss}] {speakerName}: {english}");
    }
    
    public void SaveTranscript()
    {
        string json = JsonUtility.ToJson(new { translations });
        File.WriteAllText("conference_transcript.json", json);
    }
}
```

## Differences from Transcription

| Feature | Translation | Transcription |
|---------|-------------|---------------|
| **Output** | Always English | Original language |
| **Use Case** | Cross-language communication | Same-language text |
| **Source Languages** | 99+ languages | 99+ languages |
| **Target Language** | English only | Original language |

## When to Use

### ✅ Use Translation for

- Converting foreign speech to English
- International communication
- Content localization
- Customer support across languages
- Educational content

### ❌ Use Transcription for

- Same-language speech-to-text
- Subtitles in original language
- Voice commands
- Dictation

## Provider Support

### OpenAI Whisper

```csharp
string english = await audioClip
    .GENTranslation()
    .SetModel(OpenAIModel.Whisper1)
    .SetPrompt("Context for better accuracy")
    .SetTemperature(0.0f)
    .ExecuteAsync();
```

**Features:**

- ✅ 99+ source languages
- ✅ High accuracy
- ✅ Context support
- ✅ English output only

**Note:** OpenAI Whisper is currently the primary provider for translation. Other providers may be added in future updates.

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Provide domain context
await audio.GENTranslation()
    .SetPrompt("Medical terminology: diagnosis, treatment, symptoms")
    .ExecuteAsync();

// ✅ Use deterministic mode for consistency
await audio.GENTranslation()
    .SetTemperature(0.0f)
    .ExecuteAsync();

// ✅ Cache translations
Dictionary<string, string> cache = new();

// ✅ Handle empty results
string result = await audio.GENTranslation().ExecuteAsync();
if (string.IsNullOrEmpty(result))
    Debug.LogWarning("Translation returned empty");
```

### ❌ Bad Practices

```csharp
// ❌ Don't translate already-English audio
// Use GENTranscript() instead

// ❌ Don't expect non-English output
// Translation is always to English

// ❌ Don't forget error handling
try {
    string english = await audio.GENTranslation().ExecuteAsync();
} catch (Exception ex) {
    Debug.LogError($"Translation failed: {ex.Message}");
}

// ❌ Don't block main thread
string text = audio.GENTranslation().ExecuteAsync().Result;  // NO!
```

## Audio Requirements

Same as [Speech to Text](speech-to-text.md#audio-requirements):

**Supported formats:**

- WAV, MP3, M4A, FLAC, OGG

**Limits:**

- Max file size: 25 MB
- Max duration: ~2 hours

**Quality:**

- Recommended sample rate: 16kHz+
- Channels: Mono or Stereo

## Error Handling

```csharp
try
{
    string english = await audioClip
        .GENTranslation()
        .ExecuteAsync();
    
    if (string.IsNullOrEmpty(english))
    {
        Debug.LogWarning("Empty translation - audio may be silent");
        return;
    }
    
    Debug.Log($"Translation: {english}");
}
catch (AIApiException ex)
{
    Debug.LogError($"API Error: {ex.Message}");
    // Handle: file too large, unsupported format, etc.
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Performance Tips

```csharp
// ✅ Good - parallel processing
var tasks = audioClips.Select(clip => 
    clip.GENTranslation().ExecuteAsync()
);
await UniTask.WhenAll(tasks);

// ✅ Good - cache common translations
Dictionary<string, string> cache = new();

async UniTask<string> GetCachedTranslation(AudioClip audio)
{
    string key = audio.name;
    if (!cache.ContainsKey(key))
    {
        cache[key] = await audio.GENTranslation().ExecuteAsync();
    }
    return cache[key];
}

// ❌ Bad - sequential processing
foreach (var clip in audioClips)
{
    await clip.GENTranslation().ExecuteAsync();  // Slow!
}
```

## Workflow: Translate → Speak

Common pattern for creating English audio from foreign speech:

```csharp
async UniTask<AudioClip> TranslateAndSpeak(AudioClip foreignAudio)
{
    // Step 1: Translate to English text
    string english = await foreignAudio
        .GENTranslation()
        .ExecuteAsync();
    
    // Step 2: Generate English speech
    AudioClip englishAudio = await english
        .GENSpeech()
        .SetVoice(OpenAIVoice.Alloy)
        .ExecuteAsync();
    
    return englishAudio;
}
```

## Next Steps

- [Speech to Text](speech-to-text.md) - Transcribe in original language
- [Text to Speech](text-to-speech.md) - Generate speech from text
- [Voice Change](voice-change.md) - Modify voice characteristics
