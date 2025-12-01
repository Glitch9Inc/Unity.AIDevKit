# Speech Generation

Generate speech audio from text using AI voices.

## Overview

Speech Generation allows agents to:

- Convert text to speech
- Use AI voices (OpenAI TTS, ElevenLabs)
- Generate character dialogue
- Create voice-overs
- Produce audio feedback

## Basic Setup

### Enable Speech Generation

```csharp
agent.AddLocalTool(LocalToolType.SpeechGeneration);
```

### Configure Voice

```csharp
agent.Settings.Voice = new VoiceSettings
{
    Provider = VoiceProvider.OpenAI,  // or ElevenLabs
    Voice = "alloy",                   // OpenAI voices: alloy, echo, fable, onyx, nova, shimmer
    Speed = 1.0f                       // 0.25 - 4.0
};
```

## Generate Speech

### Simple Text-to-Speech

```csharp
await agent.SpeakAsync("Hello! This is a test of speech generation.");
```

### Generate Without Playing

```csharp
AudioClip clip = await agent.GenerateSpeechAsync("Text to convert");

// Use clip later
audioSource.clip = clip;
audioSource.Play();
```

## Voice Configuration

### OpenAI Voices

```csharp
// Available OpenAI voices
agent.Settings.Voice.Voice = "alloy";    // Neutral, balanced
agent.Settings.Voice.Voice = "echo";     // Male, clear
agent.Settings.Voice.Voice = "fable";    // British accent
agent.Settings.Voice.Voice = "onyx";     // Deep, authoritative
agent.Settings.Voice.Voice = "nova";     // Female, energetic
agent.Settings.Voice.Voice = "shimmer";  // Female, soft
```

### ElevenLabs Voices

```csharp
agent.Settings.Voice = new VoiceSettings
{
    Provider = VoiceProvider.ElevenLabs,
    Voice = "21m00Tcm4TlvDq8ikWAM",  // Voice ID
    Model = "eleven_monolingual_v1",
    Stability = 0.5f,
    SimilarityBoost = 0.75f
};
```

### Speech Speed

```csharp
// Normal speed
agent.Settings.Voice.Speed = 1.0f;

// Slow (for emphasis)
agent.Settings.Voice.Speed = 0.75f;

// Fast (for excitement)
agent.Settings.Voice.Speed = 1.25f;
```

## Character Dialogue

### NPC Speech System

```csharp
public class NPCDialogueSystem : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private AudioSource audioSource;
    
    private Dictionary<string, VoiceProfile> characterVoices = new()
    {
        { "Warrior", new VoiceProfile { Voice = "onyx", Speed = 1.0f } },
        { "Mage", new VoiceProfile { Voice = "fable", Speed = 0.9f } },
        { "Rogue", new VoiceProfile { Voice = "echo", Speed = 1.1f } },
        { "Princess", new VoiceProfile { Voice = "shimmer", Speed = 0.95f } }
    };
    
    public async void SpeakAs(string characterName, string dialogue)
    {
        if (!characterVoices.TryGetValue(characterName, out var voice))
        {
            Debug.LogWarning($"No voice profile for {characterName}");
            return;
        }
        
        // Set voice
        agent.Settings.Voice.Voice = voice.Voice;
        agent.Settings.Voice.Speed = voice.Speed;
        
        // Generate and play
        AudioClip clip = await agent.GenerateSpeechAsync(dialogue);
        
        audioSource.clip = clip;
        audioSource.Play();
        
        Debug.Log($"{characterName}: {dialogue}");
    }
}

[System.Serializable]
public class VoiceProfile
{
    public string Voice;
    public float Speed;
}

// Usage
npcSystem.SpeakAs("Warrior", "Stand and fight!");
npcSystem.SpeakAs("Mage", "Let me cast a spell.");
```

### Dialogue Queue

```csharp
public class DialogueQueue : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private AudioSource audioSource;
    
    private Queue<DialogueLine> queue = new();
    private bool isPlaying;
    
    public void AddDialogue(string characterName, string text, string voice)
    {
        queue.Enqueue(new DialogueLine
        {
            Character = characterName,
            Text = text,
            Voice = voice
        });
        
        if (!isPlaying)
        {
            PlayNextLine();
        }
    }
    
    async void PlayNextLine()
    {
        if (queue.Count == 0)
        {
            isPlaying = false;
            return;
        }
        
        isPlaying = true;
        var line = queue.Dequeue();
        
        // Configure voice
        agent.Settings.Voice.Voice = line.Voice;
        
        // Generate
        AudioClip clip = await agent.GenerateSpeechAsync(line.Text);
        
        // Play
        audioSource.clip = clip;
        audioSource.Play();
        
        Debug.Log($"{line.Character}: {line.Text}");
        
        // Wait for completion
        await UniTask.WaitUntil(() => !audioSource.isPlaying);
        
        // Next line
        PlayNextLine();
    }
    
    struct DialogueLine
    {
        public string Character;
        public string Text;
        public string Voice;
    }
}
```

## Audio Management

### Save Audio Files

```csharp
public async void SaveSpeech(string text, string fileName)
{
    AudioClip clip = await agent.GenerateSpeechAsync(text);
    
    // Convert to WAV
    byte[] wavData = ConvertToWav(clip);
    
    // Save
    string path = Path.Combine(Application.persistentDataPath, $"{fileName}.wav");
    File.WriteAllBytes(path, wavData);
    
    Debug.Log($"Saved: {path}");
}

byte[] ConvertToWav(AudioClip clip)
{
    // WAV conversion implementation
    // (Use Unity's AudioClip data or a WAV library)
    return new byte[0]; // Placeholder
}
```

### Cache Generated Audio

```csharp
public class SpeechCache : MonoBehaviour
{
    private Dictionary<string, AudioClip> cache = new();
    
    public async UniTask<AudioClip> GetOrGenerate(AgentBehaviour agent, string text)
    {
        string key = $"{agent.Settings.Voice.Voice}_{text}";
        
        if (cache.TryGetValue(key, out var cached))
        {
            Debug.Log("Using cached audio");
            return cached;
        }
        
        AudioClip clip = await agent.GenerateSpeechAsync(text);
        cache[key] = clip;
        
        return clip;
    }
    
    public void ClearCache()
    {
        foreach (var clip in cache.Values)
        {
            Destroy(clip);
        }
        cache.Clear();
    }
}
```

## UI Integration

### Speech Button

```csharp
public class SpeechButton : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button speakButton;
    [SerializeField] private TMP_Text textToSpeak;
    
    void Start()
    {
        speakButton.onClick.AddListener(OnSpeakClicked);
    }
    
    async void OnSpeakClicked()
    {
        speakButton.interactable = false;
        
        await agent.SpeakAsync(textToSpeak.text);
        
        speakButton.interactable = true;
    }
}
```

### Voice Selection UI

```csharp
public class VoiceSelector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Dropdown voiceDropdown;
    
    void Start()
    {
        // Populate dropdown
        voiceDropdown.options.Clear();
        voiceDropdown.options.Add(new TMP_Dropdown.OptionData("Alloy"));
        voiceDropdown.options.Add(new TMP_Dropdown.OptionData("Echo"));
        voiceDropdown.options.Add(new TMP_Dropdown.OptionData("Fable"));
        voiceDropdown.options.Add(new TMP_Dropdown.OptionData("Onyx"));
        voiceDropdown.options.Add(new TMP_Dropdown.OptionData("Nova"));
        voiceDropdown.options.Add(new TMP_Dropdown.OptionData("Shimmer"));
        
        voiceDropdown.onValueChanged.AddListener(OnVoiceChanged);
    }
    
    void OnVoiceChanged(int index)
    {
        string voice = voiceDropdown.options[index].text.ToLower();
        agent.Settings.Voice.Voice = voice;
        
        Debug.Log($"Voice changed to: {voice}");
    }
}
```

## Advanced Usage

### Multi-Language Support

```csharp
public async void SpeakInLanguage(string text, string language)
{
    // OpenAI TTS supports multiple languages automatically
    // Just provide text in the target language
    
    await agent.SpeakAsync(text);
}

// Usage
SpeakInLanguage("Bonjour!", "french");
SpeakInLanguage("こんにちは", "japanese");
SpeakInLanguage("Hola!", "spanish");
```

### Emotional Speech

```csharp
public async void SpeakWithEmotion(string text, Emotion emotion)
{
    // Modify text with emotion cues
    string emotionalText = emotion switch
    {
        Emotion.Excited => text + "!",
        Emotion.Sad => text + "...",
        Emotion.Angry => text.ToUpper() + "!",
        Emotion.Calm => text + ".",
        _ => text
    };
    
    // Adjust speed based on emotion
    agent.Settings.Voice.Speed = emotion switch
    {
        Emotion.Excited => 1.2f,
        Emotion.Sad => 0.8f,
        Emotion.Angry => 1.1f,
        Emotion.Calm => 0.9f,
        _ => 1.0f
    };
    
    await agent.SpeakAsync(emotionalText);
}

public enum Emotion
{
    Neutral,
    Excited,
    Sad,
    Angry,
    Calm
}
```

### Batch Generation

```csharp
public async UniTask<List<AudioClip>> GenerateBatch(string[] lines)
{
    List<AudioClip> clips = new();
    
    foreach (var line in lines)
    {
        var clip = await agent.GenerateSpeechAsync(line);
        clips.Add(clip);
        
        await UniTask.Delay(100); // Rate limiting
    }
    
    return clips;
}

// Usage
string[] dialogue = {
    "Welcome to the game!",
    "Let's begin your adventure.",
    "Good luck, hero!"
};

var clips = await GenerateBatch(dialogue);
```

## Error Handling

### Handle Generation Errors

```csharp
agent.onSpeechError.AddListener(error =>
{
    Debug.LogError($"Speech generation failed: {error}");
    
    if (error.Contains("rate_limit"))
    {
        ShowMessage("Too many requests. Please wait.");
    }
    else if (error.Contains("quota"))
    {
        ShowMessage("Speech quota exceeded.");
    }
    else
    {
        ShowMessage("Failed to generate speech.");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class SpeechGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private AudioSource audioSource;
    
    [Header("Voice Settings")]
    [SerializeField] private string defaultVoice = "alloy";
    [SerializeField] private float defaultSpeed = 1.0f;
    
    private Dictionary<string, AudioClip> cache = new();
    
    async void Start()
    {
        await SetupSpeechGeneration();
    }
    
    async UniTask SetupSpeechGeneration()
    {
        // Configure voice
        agent.Settings.Voice = new VoiceSettings
        {
            Provider = VoiceProvider.OpenAI,
            Voice = defaultVoice,
            Speed = defaultSpeed
        };
        
        // Add tool
        agent.AddLocalTool(LocalToolType.SpeechGeneration);
        
        // Listen for events
        agent.onSpeechGenerated.AddListener(OnSpeechGenerated);
        agent.onSpeechError.AddListener(OnSpeechError);
        
        Debug.Log("✓ Speech generation ready");
    }
    
    public async void Speak(string text, string voice = null, float? speed = null)
    {
        // Configure voice
        if (voice != null)
            agent.Settings.Voice.Voice = voice;
        if (speed.HasValue)
            agent.Settings.Voice.Speed = speed.Value;
        
        Debug.Log($"🔊 Speaking: {text}");
        
        try
        {
            // Check cache
            string cacheKey = $"{agent.Settings.Voice.Voice}_{text}";
            
            if (cache.TryGetValue(cacheKey, out var cached))
            {
                PlayAudio(cached);
                return;
            }
            
            // Generate
            AudioClip clip = await agent.GenerateSpeechAsync(text);
            
            // Cache
            cache[cacheKey] = clip;
            
            // Play
            PlayAudio(clip);
        }
        catch (Exception ex)
        {
            Debug.LogError($"Speech failed: {ex.Message}");
        }
    }
    
    void PlayAudio(AudioClip clip)
    {
        audioSource.clip = clip;
        audioSource.Play();
        
        Debug.Log($"▶️ Playing audio: {clip.length:F2}s");
    }
    
    void OnSpeechGenerated(AudioClip clip)
    {
        Debug.Log($"✓ Speech generated: {clip.length:F2}s");
    }
    
    void OnSpeechError(string error)
    {
        Debug.LogError($"Speech error: {error}");
    }
    
    public void ClearCache()
    {
        foreach (var clip in cache.Values)
        {
            Destroy(clip);
        }
        cache.Clear();
        
        Debug.Log("✓ Cache cleared");
    }
    
    void OnDestroy()
    {
        ClearCache();
    }
}
```

## Next Steps

- [Speech Transcription](speech-transcription.md)
- [Audio Input](../../responses/audio-input.md)
- [Voice Settings](../../parameters/voice.md)
