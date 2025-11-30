# Text-to-Speech

Generate natural speech from text using OpenAI TTS.

## Overview

Text-to-Speech provides:

- Natural voice synthesis
- Multiple voice options
- Speed control
- Audio streaming
- Character voice presets

## Basic Setup

### Generate Speech

```csharp
AudioClip clip = await agent.GenerateSpeechAsync("Hello, world!");
agent.AudioController.OutputPlayer.Play(clip);
```

## Voice Selection

### Available Voices

OpenAI TTS offers 6 voices:

- **alloy** - Neutral, balanced
- **echo** - Male, clear
- **fable** - British, warm
- **onyx** - Deep, authoritative
- **nova** - Female, friendly
- **shimmer** - Soft, gentle

### Set Voice

```csharp
public class VoiceSelector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Set voice
        agent.ParametersController.SetVoice("nova");
        
        Debug.Log("Voice set to: nova");
    }
    
    public async UniTask SpeakWithVoice(string text, string voice)
    {
        agent.ParametersController.SetVoice(voice);
        
        AudioClip clip = await agent.GenerateSpeechAsync(text);
        agent.AudioController.OutputPlayer.Play(clip);
    }
}

// Examples
await SpeakWithVoice("Welcome!", "alloy");
await SpeakWithVoice("Warning!", "onyx");
await SpeakWithVoice("Thank you!", "nova");
```

### Voice Comparison UI

```csharp
public class VoiceComparator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Dropdown voiceDropdown;
    [SerializeField] private Button testButton;
    
    private string[] voices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
    
    void Start()
    {
        voiceDropdown.ClearOptions();
        voiceDropdown.AddOptions(voices.ToList());
        
        testButton.onClick.AddListener(TestVoice);
    }
    
    async void TestVoice()
    {
        string voice = voices[voiceDropdown.value];
        string testText = $"This is the {voice} voice.";
        
        agent.ParametersController.SetVoice(voice);
        
        AudioClip clip = await agent.GenerateSpeechAsync(testText);
        agent.AudioController.OutputPlayer.Play(clip);
        
        Debug.Log($"Testing voice: {voice}");
    }
}
```

## Speed Control

### Adjust Speech Speed

```csharp
public class SpeechSpeedController : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Slider speedSlider;
    [SerializeField] private TMP_Text speedText;
    
    void Start()
    {
        speedSlider.minValue = 0.25f;
        speedSlider.maxValue = 4.0f;
        speedSlider.value = 1.0f;
        
        speedSlider.onValueChanged.AddListener(SetSpeed);
    }
    
    void SetSpeed(float speed)
    {
        agent.ParametersController.SetSpeechSpeed(speed);
        speedText.text = $"{speed:F2}x";
        
        Debug.Log($"Speech speed: {speed}x");
    }
}
```

## Character Voices

### Voice Presets

```csharp
public class CharacterVoicePresets : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public enum CharacterType
    {
        Hero,
        Villain,
        Companion,
        Merchant,
        Elder
    }
    
    Dictionary<CharacterType, string> voicePresets = new()
    {
        { CharacterType.Hero, "echo" },
        { CharacterType.Villain, "onyx" },
        { CharacterType.Companion, "nova" },
        { CharacterType.Merchant, "fable" },
        { CharacterType.Elder, "alloy" }
    };
    
    public async UniTask SpeakAsCharacter(CharacterType character, string dialogue)
    {
        string voice = voicePresets[character];
        agent.ParametersController.SetVoice(voice);
        
        Debug.Log($"{character} ({voice}): {dialogue}");
        
        AudioClip clip = await agent.GenerateSpeechAsync(dialogue);
        agent.AudioController.OutputPlayer.Play(clip);
    }
}

// Example usage
var presets = GetComponent<CharacterVoicePresets>();
await presets.SpeakAsCharacter(CharacterVoicePresets.CharacterType.Hero, 
    "I will save the kingdom!");
await presets.SpeakAsCharacter(CharacterVoicePresets.CharacterType.Villain, 
    "You cannot stop me!");
```

### Dynamic Voice Assignment

```csharp
public class DynamicVoiceAssignment : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, string> characterVoices = new();
    private string[] availableVoices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
    
    public async UniTask SpeakAsNPC(string npcName, string dialogue)
    {
        // Assign voice if not assigned
        if (!characterVoices.ContainsKey(npcName))
        {
            string voice = AssignVoice(npcName);
            characterVoices[npcName] = voice;
        }
        
        string assignedVoice = characterVoices[npcName];
        agent.ParametersController.SetVoice(assignedVoice);
        
        Debug.Log($"{npcName} ({assignedVoice}): {dialogue}");
        
        AudioClip clip = await agent.GenerateSpeechAsync(dialogue);
        agent.AudioController.OutputPlayer.Play(clip);
    }
    
    string AssignVoice(string npcName)
    {
        // Hash name to consistently assign same voice
        int hash = npcName.GetHashCode();
        int index = Mathf.Abs(hash) % availableVoices.Length;
        return availableVoices[index];
    }
}
```

## Dialogue Queue

### Sequential Dialogue

```csharp
public class DialogueQueue : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Queue<(string text, string voice)> dialogueQueue = new();
    private bool isPlaying;
    
    public void QueueDialogue(string text, string voice)
    {
        dialogueQueue.Enqueue((text, voice));
        
        if (!isPlaying)
        {
            PlayNextDialogue().Forget();
        }
    }
    
    async UniTaskVoid PlayNextDialogue()
    {
        if (dialogueQueue.Count == 0)
        {
            isPlaying = false;
            return;
        }
        
        isPlaying = true;
        
        var (text, voice) = dialogueQueue.Dequeue();
        
        agent.ParametersController.SetVoice(voice);
        
        AudioClip clip = await agent.GenerateSpeechAsync(text);
        
        var player = agent.AudioController.OutputPlayer;
        player.Play(clip);
        
        // Wait for completion
        await UniTask.WaitUntil(() => !player.IsPlaying);
        
        // Play next
        await PlayNextDialogue();
    }
    
    public void ClearQueue()
    {
        dialogueQueue.Clear();
        agent.AudioController.OutputPlayer.Stop();
        isPlaying = false;
    }
}

// Example usage
dialogueQueue.QueueDialogue("Hello traveler!", "nova");
dialogueQueue.QueueDialogue("What brings you here?", "nova");
dialogueQueue.QueueDialogue("I need your help!", "echo");
```

## Subtitle Synchronization

### Show Subtitles with Speech

```csharp
public class SubtitleSync : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text subtitleText;
    [SerializeField] private float wordsPerSecond = 3f;
    
    public async UniTask SpeakWithSubtitles(string text, string voice)
    {
        agent.ParametersController.SetVoice(voice);
        
        // Generate speech
        AudioClip clip = await agent.GenerateSpeechAsync(text);
        
        // Play audio and show subtitles
        var player = agent.AudioController.OutputPlayer;
        player.Play(clip);
        
        await ShowSubtitlesAnimated(text, clip.length);
    }
    
    async UniTask ShowSubtitlesAnimated(string text, float duration)
    {
        string[] words = text.Split(' ');
        float timePerWord = duration / words.Length;
        
        StringBuilder current = new();
        
        foreach (string word in words)
        {
            current.Append(word).Append(" ");
            subtitleText.text = current.ToString();
            
            await UniTask.Delay(TimeSpan.FromSeconds(timePerWord));
        }
        
        // Clear after a delay
        await UniTask.Delay(TimeSpan.FromSeconds(2));
        subtitleText.text = "";
    }
}
```

## Caching

### Cache Generated Speech

```csharp
public class SpeechCache : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxCacheSize = 50;
    
    private Dictionary<string, AudioClip> cache = new();
    private Queue<string> cacheKeys = new();
    
    public async UniTask<AudioClip> GetOrGenerateSpeech(string text, string voice)
    {
        string key = $"{voice}_{text}";
        
        // Check cache
        if (cache.ContainsKey(key))
        {
            Debug.Log($"✓ Cache hit: {text.Substring(0, Math.Min(30, text.Length))}...");
            return cache[key];
        }
        
        // Generate
        Debug.Log($"Generating speech: {text.Substring(0, Math.Min(30, text.Length))}...");
        
        agent.ParametersController.SetVoice(voice);
        AudioClip clip = await agent.GenerateSpeechAsync(text);
        
        // Add to cache
        AddToCache(key, clip);
        
        return clip;
    }
    
    void AddToCache(string key, AudioClip clip)
    {
        // Remove oldest if cache full
        if (cache.Count >= maxCacheSize)
        {
            string oldestKey = cacheKeys.Dequeue();
            cache.Remove(oldestKey);
        }
        
        cache[key] = clip;
        cacheKeys.Enqueue(key);
    }
    
    public void ClearCache()
    {
        cache.Clear();
        cacheKeys.Clear();
        
        Debug.Log("Speech cache cleared");
    }
}
```

## Emotion and Emphasis

### SSML-like Formatting

```csharp
public class EmotionalSpeech : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async UniTask SpeakWithEmotion(string text, string emotion)
    {
        // Add emotional context
        string emotionalText = AddEmotionalContext(text, emotion);
        
        AudioClip clip = await agent.GenerateSpeechAsync(emotionalText);
        agent.AudioController.OutputPlayer.Play(clip);
    }
    
    string AddEmotionalContext(string text, string emotion)
    {
        switch (emotion.ToLower())
        {
            case "excited":
                return text + "!";
            case "sad":
                return text.ToLower() + "...";
            case "angry":
                return text.ToUpper() + "!";
            case "whisper":
                return $"*{text}*";
            default:
                return text;
        }
    }
}
```

## Batch Generation

### Pre-generate Multiple Lines

```csharp
public class BatchSpeechGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async UniTask<Dictionary<string, AudioClip>> GenerateBatch(
        Dictionary<string, string> dialogues,
        string voice)
    {
        agent.ParametersController.SetVoice(voice);
        
        Dictionary<string, AudioClip> results = new();
        
        int count = 0;
        foreach (var kvp in dialogues)
        {
            count++;
            Debug.Log($"Generating {count}/{dialogues.Count}: {kvp.Key}");
            
            AudioClip clip = await agent.GenerateSpeechAsync(kvp.Value);
            results[kvp.Key] = clip;
            
            // Rate limiting
            await UniTask.Delay(TimeSpan.FromSeconds(1));
        }
        
        Debug.Log($"✓ Batch generation complete: {count} clips");
        return results;
    }
}

// Example usage
var dialogues = new Dictionary<string, string>
{
    { "greeting", "Welcome to the shop!" },
    { "thanks", "Thank you for your purchase!" },
    { "goodbye", "Come back soon!" }
};

var clips = await batchGenerator.GenerateBatch(dialogues, "fable");
```

## Save and Load

### Export Audio Files

```csharp
public class SpeechExporter : MonoBehaviour
{
    [SerializeField] private string exportPath = "Speech";
    
    public void SaveSpeech(AudioClip clip, string filename)
    {
        byte[] wavData = ConvertToWav(clip);
        
        string fullPath = Path.Combine(
            Application.persistentDataPath,
            exportPath,
            $"{filename}.wav"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(fullPath));
        File.WriteAllBytes(fullPath, wavData);
        
        Debug.Log($"💾 Saved: {fullPath}");
    }
    
    byte[] ConvertToWav(AudioClip clip)
    {
        float[] samples = new float[clip.samples * clip.channels];
        clip.GetData(samples, 0);
        
        byte[] wav = new byte[44 + samples.Length * 2];
        
        // WAV header
        System.Text.Encoding.UTF8.GetBytes("RIFF").CopyTo(wav, 0);
        BitConverter.GetBytes(wav.Length - 8).CopyTo(wav, 4);
        System.Text.Encoding.UTF8.GetBytes("WAVE").CopyTo(wav, 8);
        
        // fmt chunk
        System.Text.Encoding.UTF8.GetBytes("fmt ").CopyTo(wav, 12);
        BitConverter.GetBytes(16).CopyTo(wav, 16);
        BitConverter.GetBytes((short)1).CopyTo(wav, 20);
        BitConverter.GetBytes((short)clip.channels).CopyTo(wav, 22);
        BitConverter.GetBytes(clip.frequency).CopyTo(wav, 24);
        BitConverter.GetBytes(clip.frequency * clip.channels * 2).CopyTo(wav, 28);
        BitConverter.GetBytes((short)(clip.channels * 2)).CopyTo(wav, 32);
        BitConverter.GetBytes((short)16).CopyTo(wav, 34);
        
        // data chunk
        System.Text.Encoding.UTF8.GetBytes("data").CopyTo(wav, 36);
        BitConverter.GetBytes(samples.Length * 2).CopyTo(wav, 40);
        
        // Audio data
        int offset = 44;
        for (int i = 0; i < samples.Length; i++)
        {
            short sample = (short)(samples[i] * short.MaxValue);
            BitConverter.GetBytes(sample).CopyTo(wav, offset);
            offset += 2;
        }
        
        return wav;
    }
}
```

## Error Handling

### Handle TTS Errors

```csharp
try
{
    AudioClip clip = await agent.GenerateSpeechAsync(text);
    agent.AudioController.OutputPlayer.Play(clip);
}
catch (Exception ex)
{
    Debug.LogError($"TTS error: {ex.Message}");
    
    if (ex.Message.Contains("invalid_text"))
    {
        ShowMessage("Invalid text input.");
    }
    else if (ex.Message.Contains("too_long"))
    {
        ShowMessage("Text too long (max 4096 characters).");
    }
    else if (ex.Message.Contains("rate_limit"))
    {
        ShowMessage("Rate limit exceeded. Please wait.");
    }
    else
    {
        ShowMessage("Speech generation failed.");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class TTSManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField textInput;
    [SerializeField] private Dropdown voiceDropdown;
    [SerializeField] private Button speakButton;
    [SerializeField] private TMP_Text subtitleText;
    
    private string[] voices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
    private SpeechCache cache;
    
    async void Start()
    {
        cache = gameObject.AddComponent<SpeechCache>();
        
        SetupUI();
        
        Debug.Log("✓ TTS Manager ready");
    }
    
    void SetupUI()
    {
        voiceDropdown.ClearOptions();
        voiceDropdown.AddOptions(voices.ToList());
        voiceDropdown.value = 4; // nova
        
        speakButton.onClick.AddListener(Speak);
    }
    
    async void Speak()
    {
        string text = textInput.text;
        
        if (string.IsNullOrEmpty(text))
        {
            Debug.LogWarning("No text to speak");
            return;
        }
        
        string voice = voices[voiceDropdown.value];
        
        speakButton.interactable = false;
        subtitleText.text = "Generating...";
        
        try
        {
            // Get or generate speech
            AudioClip clip = await cache.GetOrGenerateSpeech(text, voice);
            
            // Play with subtitles
            await SpeakWithSubtitles(text, clip);
            
            Debug.Log($"✓ Spoke: {text}");
        }
        catch (Exception ex)
        {
            Debug.LogError($"TTS error: {ex.Message}");
            subtitleText.text = $"<color=red>Error: {ex.Message}</color>";
            
            await UniTask.Delay(TimeSpan.FromSeconds(2));
        }
        finally
        {
            speakButton.interactable = true;
            subtitleText.text = "";
        }
    }
    
    async UniTask SpeakWithSubtitles(string text, AudioClip clip)
    {
        var player = agent.AudioController.OutputPlayer;
        player.Play(clip);
        
        // Show subtitles
        string[] words = text.Split(' ');
        float timePerWord = clip.length / words.Length;
        
        StringBuilder current = new();
        
        foreach (string word in words)
        {
            current.Append(word).Append(" ");
            subtitleText.text = current.ToString();
            
            await UniTask.Delay(TimeSpan.FromSeconds(timePerWord));
        }
        
        // Wait for completion
        await UniTask.WaitUntil(() => !player.IsPlaying);
        
        // Clear after delay
        await UniTask.Delay(TimeSpan.FromSeconds(1));
        subtitleText.text = "";
    }
}
```

## Next Steps

- [Audio Input Recorder](input-recorder.md)
- [Audio Output Player](output-player.md)
- [Speech Transcription](transcription.md)
