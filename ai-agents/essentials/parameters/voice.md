# Voice

Configure text-to-speech voices for your agent.

## Overview

AI Dev Kit supports voices from:

- **OpenAI** - 6 voices (alloy, echo, fable, onyx, nova, shimmer)
- **ElevenLabs** - Custom voices and voice library
- **Google** - Neural2 and WaveNet voices
- **Azure** - Neural voices

## OpenAI Voices

### Available Voices

```csharp
public enum OpenAIVoice
{
    Alloy,    // Neutral, balanced
    Echo,     // Male, clear
    Fable,    // British accent
    Onyx,     // Deep male
    Nova,     // Female, friendly
    Shimmer   // Female, warm
}
```

### Set Voice

```csharp
// In AgentSettings
settings.OutputAudioParameters.Voice = "nova";

// At runtime
agent.Voice = new Voice("nova");
```

### Try All Voices

```csharp
public class VoicePreview : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private string[] voices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
    
    public async void PreviewVoice(int index)
    {
        string voice = voices[index];
        agent.Voice = new Voice(voice);
        
        AudioClip clip = await agent.GenerateSpeechAsync(
            $"Hello, I am {voice}. This is how I sound."
        );
        
        agent.OutputAudioPlayer.Play(clip);
    }
}
```

## Voice Characteristics

### OpenAI Voice Guide

```csharp
public class VoiceCharacteristics
{
    public static string GetDescription(string voice)
    {
        return voice switch
        {
            "alloy" => "Neutral, balanced tone. Good for general use.",
            "echo" => "Male voice, clear and direct. Professional.",
            "fable" => "British accent, expressive. Storytelling.",
            "onyx" => "Deep male voice. Authoritative.",
            "nova" => "Female, friendly and conversational.",
            "shimmer" => "Female, warm and soothing.",
            _ => "Unknown voice"
        };
    }
}
```

### Choose by Use Case

```csharp
public Voice GetVoiceForUseCase(string useCase)
{
    return useCase switch
    {
        "assistant" => new Voice("nova"),      // Friendly helper
        "tutorial" => new Voice("alloy"),      // Clear instructor
        "narrator" => new Voice("fable"),      // Storyteller
        "character" => new Voice("onyx"),      // Deep character
        "guide" => new Voice("echo"),          // Professional guide
        _ => new Voice("alloy")
    };
}
```

## Voice Settings

### Speech Parameters

```csharp
// In AgentSettings.OutputAudioParameters
settings.OutputAudioParameters = new SpeechParameters
{
    Model = "tts-1-hd",        // or "tts-1" for standard
    Voice = "nova",
    Speed = 1.0f,              // 0.25 to 4.0
    Format = AudioFormat.mp3    // or wav, opus, etc.
};
```

### Speed Control

```csharp
// Normal speed
agent.Settings.OutputAudioParameters.Speed = 1.0f;

// Slower (better for learning)
agent.Settings.OutputAudioParameters.Speed = 0.8f;

// Faster (efficient)
agent.Settings.OutputAudioParameters.Speed = 1.25f;

// Very slow
agent.Settings.OutputAudioParameters.Speed = 0.5f;

// Very fast
agent.Settings.OutputAudioParameters.Speed = 2.0f;
```

### Runtime Speed Adjustment

```csharp
public class SpeedControl : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Slider speedSlider;
    
    void Start()
    {
        speedSlider.minValue = 0.5f;
        speedSlider.maxValue = 2.0f;
        speedSlider.value = 1.0f;
        
        speedSlider.onValueChanged.AddListener(OnSpeedChanged);
    }
    
    void OnSpeedChanged(float speed)
    {
        agent.Settings.OutputAudioParameters.Speed = speed;
        Debug.Log($"Speech speed: {speed:F2}x");
    }
}
```

## Voice Selection UI

### Dropdown Menu

```csharp
public class VoiceSelector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Dropdown voiceDropdown;
    
    private string[] voices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
    
    void Start()
    {
        // Populate dropdown
        voiceDropdown.ClearOptions();
        voiceDropdown.AddOptions(voices.ToList());
        
        // Set current selection
        int currentIndex = Array.IndexOf(voices, agent.Voice.Id);
        voiceDropdown.value = currentIndex;
        
        // Listen for changes
        voiceDropdown.onValueChanged.AddListener(OnVoiceChanged);
    }
    
    void OnVoiceChanged(int index)
    {
        string voice = voices[index];
        agent.Voice = new Voice(voice);
        
        Debug.Log($"Voice changed to: {voice}");
        
        // Save preference
        PlayerPrefs.SetString("PreferredVoice", voice);
    }
}
```

### Voice Preview Buttons

```csharp
public class VoicePreviewUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private VoiceButton[] voiceButtons;
    
    void Start()
    {
        foreach (var button in voiceButtons)
        {
            button.onClick.AddListener(() => PreviewVoice(button.voiceName));
        }
    }
    
    async void PreviewVoice(string voiceName)
    {
        // Temporarily set voice
        var originalVoice = agent.Voice;
        agent.Voice = new Voice(voiceName);
        
        // Generate preview
        AudioClip clip = await agent.GenerateSpeechAsync(
            $"Hello, I'm {voiceName}. Nice to meet you!"
        );
        
        // Play
        agent.OutputAudioPlayer.Play(clip);
        
        // Wait for playback
        await UniTask.Delay((int)(clip.length * 1000));
        
        // Restore original voice
        agent.Voice = originalVoice;
    }
}
```

## Character Voices

### Assign Voices to Characters

```csharp
public class CharacterVoiceManager : MonoBehaviour
{
    [System.Serializable]
    public class Character
    {
        public string Name;
        public string Voice;
        public float Speed;
    }
    
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Character[] characters = new[]
    {
        new Character { Name = "Hero", Voice = "onyx", Speed = 1.0f },
        new Character { Name = "Guide", Voice = "nova", Speed = 0.9f },
        new Character { Name = "Narrator", Voice = "fable", Speed = 0.8f }
    };
    
    public async UniTask SpeakAs(string characterName, string text)
    {
        var character = characters.FirstOrDefault(c => c.Name == characterName);
        if (character == null)
        {
            Debug.LogWarning($"Character not found: {characterName}");
            return;
        }
        
        // Set voice and speed
        agent.Voice = new Voice(character.Voice);
        agent.Settings.OutputAudioParameters.Speed = character.Speed;
        
        // Generate speech
        AudioClip clip = await agent.GenerateSpeechAsync(text);
        agent.OutputAudioPlayer.Play(clip);
    }
}
```

## ElevenLabs Integration

### Custom Voices

```csharp
// Set ElevenLabs voice (if using ElevenLabs provider)
agent.Settings.AudioProvider = AudioProvider.ElevenLabs;
agent.Voice = new Voice("voice_id_from_elevenlabs");

// With custom settings
agent.Settings.OutputAudioParameters = new SpeechParameters
{
    Voice = "your_voice_id",
    Stability = 0.5f,      // 0-1
    SimilarityBoost = 0.75f // 0-1
};
```

### Voice Cloning

```csharp
// Clone voice from samples (ElevenLabs API)
public async UniTask<string> CloneVoice(AudioClip[] samples, string name)
{
    // Upload samples to ElevenLabs
    // Returns voice_id
    
    string voiceId = await ElevenLabsAPI.CloneVoiceAsync(samples, name);
    
    // Use cloned voice
    agent.Voice = new Voice(voiceId);
    
    return voiceId;
}
```

## Voice Persistence

### Save User Preference

```csharp
void Start()
{
    // Load saved voice
    string savedVoice = PlayerPrefs.GetString("PreferredVoice", "alloy");
    agent.Voice = new Voice(savedVoice);
    
    Debug.Log($"Using voice: {savedVoice}");
}

public void SetVoice(string voiceName)
{
    agent.Voice = new Voice(voiceName);
    
    // Save preference
    PlayerPrefs.SetString("PreferredVoice", voiceName);
    PlayerPrefs.Save();
    
    Debug.Log($"Voice saved: {voiceName}");
}
```

### Per-Agent Voices

```csharp
public class MultiAgentVoices : MonoBehaviour
{
    [SerializeField] private AgentBehaviour assistantAgent;
    [SerializeField] private AgentBehaviour tutorAgent;
    [SerializeField] private AgentBehaviour narratorAgent;
    
    void Start()
    {
        // Each agent has different voice
        assistantAgent.Voice = new Voice("nova");
        tutorAgent.Voice = new Voice("echo");
        narratorAgent.Voice = new Voice("fable");
    }
}
```

## Voice Quality

### Model Selection

```csharp
// High quality (slower, more expensive)
agent.Settings.OutputAudioParameters.Model = "tts-1-hd";

// Standard quality (faster, cheaper)
agent.Settings.OutputAudioParameters.Model = "tts-1";
```

### Audio Format

```csharp
// MP3 (compressed, smaller files)
agent.Settings.OutputAudioParameters.Format = AudioFormat.mp3;

// WAV (uncompressed, high quality)
agent.Settings.OutputAudioParameters.Format = AudioFormat.wav;

// Opus (web optimized)
agent.Settings.OutputAudioParameters.Format = AudioFormat.opus;
```

## Best Practices

### 1. Test Voices for Your Use Case

```csharp
public class VoiceTester : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private string testPhrase = "Welcome to the game. Let me guide you through the tutorial.";
    
    public async void TestAllVoices()
    {
        string[] voices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
        
        foreach (string voice in voices)
        {
            agent.Voice = new Voice(voice);
            
            Debug.Log($"Testing voice: {voice}");
            AudioClip clip = await agent.GenerateSpeechAsync(testPhrase);
            agent.OutputAudioPlayer.Play(clip);
            
            // Wait for playback
            await UniTask.Delay((int)(clip.length * 1000) + 500);
        }
    }
}
```

### 2. Match Voice to Content

```csharp
public Voice GetVoiceForContent(string content)
{
    // Story/narrative content
    if (content.Contains("Once upon a time") || content.Contains("story"))
    {
        return new Voice("fable");
    }
    // Tutorial/instructional
    else if (content.Contains("step") || content.Contains("tutorial"))
    {
        return new Voice("echo");
    }
    // Friendly conversation
    else
    {
        return new Voice("nova");
    }
}
```

### 3. Provide Speed Control

```csharp
// Let users adjust speech speed
public class AccessibleSpeech : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Accessibility")]
    [SerializeField] private float defaultSpeed = 1.0f;
    [SerializeField] private bool allowSpeedControl = true;
    
    void Start()
    {
        // Load saved speed preference
        float savedSpeed = PlayerPrefs.GetFloat("SpeechSpeed", defaultSpeed);
        agent.Settings.OutputAudioParameters.Speed = savedSpeed;
    }
    
    public void SetSpeed(float speed)
    {
        if (!allowSpeedControl) return;
        
        agent.Settings.OutputAudioParameters.Speed = speed;
        PlayerPrefs.SetFloat("SpeechSpeed", speed);
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;
using TMPro;
using System.Linq;

public class VoiceManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("UI")]
    [SerializeField] private TMP_Dropdown voiceDropdown;
    [SerializeField] private Slider speedSlider;
    [SerializeField] private TMP_Text speedText;
    [SerializeField] private Button previewButton;
    
    private string[] voices = { "alloy", "echo", "fable", "onyx", "nova", "shimmer" };
    private string[] descriptions = {
        "Neutral, balanced",
        "Male, clear",
        "British, expressive",
        "Deep male",
        "Female, friendly",
        "Female, warm"
    };
    
    void Start()
    {
        SetupVoiceDropdown();
        SetupSpeedSlider();
        
        previewButton.onClick.AddListener(PreviewCurrentVoice);
        
        // Load saved preferences
        LoadPreferences();
    }
    
    void SetupVoiceDropdown()
    {
        voiceDropdown.ClearOptions();
        
        var options = voices.Select((voice, i) => 
            $"{voice} - {descriptions[i]}"
        ).ToList();
        
        voiceDropdown.AddOptions(options);
        voiceDropdown.onValueChanged.AddListener(OnVoiceChanged);
    }
    
    void SetupSpeedSlider()
    {
        speedSlider.minValue = 0.5f;
        speedSlider.maxValue = 2.0f;
        speedSlider.value = 1.0f;
        speedSlider.onValueChanged.AddListener(OnSpeedChanged);
        
        UpdateSpeedText(1.0f);
    }
    
    void OnVoiceChanged(int index)
    {
        string voice = voices[index];
        agent.Voice = new Voice(voice);
        
        PlayerPrefs.SetString("PreferredVoice", voice);
        Debug.Log($"✓ Voice: {voice}");
    }
    
    void OnSpeedChanged(float speed)
    {
        agent.Settings.OutputAudioParameters.Speed = speed;
        UpdateSpeedText(speed);
        
        PlayerPrefs.SetFloat("SpeechSpeed", speed);
    }
    
    void UpdateSpeedText(float speed)
    {
        speedText.text = $"Speed: {speed:F2}x";
    }
    
    async void PreviewCurrentVoice()
    {
        previewButton.interactable = false;
        
        try
        {
            string previewText = $"Hello, I'm {agent.Voice.Id}. This is my voice at {agent.Settings.OutputAudioParameters.Speed:F1}x speed.";
            
            AudioClip clip = await agent.GenerateSpeechAsync(previewText);
            agent.OutputAudioPlayer.Play(clip);
            
            await UniTask.Delay((int)(clip.length * 1000));
        }
        finally
        {
            previewButton.interactable = true;
        }
    }
    
    void LoadPreferences()
    {
        // Load voice
        string savedVoice = PlayerPrefs.GetString("PreferredVoice", "alloy");
        int voiceIndex = System.Array.IndexOf(voices, savedVoice);
        if (voiceIndex >= 0)
        {
            voiceDropdown.value = voiceIndex;
        }
        
        // Load speed
        float savedSpeed = PlayerPrefs.GetFloat("SpeechSpeed", 1.0f);
        speedSlider.value = savedSpeed;
    }
}
```

## Next Steps

- [Models](models.md)
- [Text-to-Speech](../audio/text-to-speech.md)
- [Audio Setup](../configuration/audio-setup.md)
