# Speech Transcription

Convert speech audio to text using AI transcription.

## Overview

Speech Transcription allows agents to:

- Convert audio to text
- Transcribe voice recordings
- Process player voice input
- Support multiple languages
- Generate subtitles

## Basic Setup

### Enable Transcription

```csharp
agent.AddLocalTool(LocalToolType.SpeechTranscription);
```

### Configure Settings

```csharp
agent.Settings.Transcription = new TranscriptionSettings
{
    Provider = TranscriptionProvider.OpenAI,  // Whisper
    Language = "en",                          // Language code (or auto-detect)
    Prompt = "",                              // Context prompt for better accuracy
    Temperature = 0.0f                        // 0-1, higher = more creative
};
```

## Transcribe Audio

### From AudioClip

```csharp
AudioClip recording = GetAudioRecording();
string transcription = await agent.TranscribeAsync(recording);

Debug.Log($"Transcribed: {transcription}");
```

### From File

```csharp
string audioPath = "path/to/audio.mp3";
string transcription = await agent.TranscribeFileAsync(audioPath);

Debug.Log($"Transcribed: {transcription}");
```

### From Microphone

```csharp
// Start recording
agent.StartRecording();

// Stop and transcribe
await UniTask.Delay(5000);
string transcription = await agent.StopRecordingAndTranscribeAsync();

Debug.Log($"You said: {transcription}");
```

## Voice Input System

### Push-to-Talk

```csharp
public class VoiceInput : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private KeyCode recordKey = KeyCode.Space;
    
    private bool isRecording;
    
    void Update()
    {
        if (Input.GetKeyDown(recordKey))
        {
            StartRecording();
        }
        else if (Input.GetKeyUp(recordKey))
        {
            StopRecording();
        }
    }
    
    void StartRecording()
    {
        agent.StartRecording();
        isRecording = true;
        
        Debug.Log("🎤 Recording...");
    }
    
    async void StopRecording()
    {
        if (!isRecording) return;
        
        isRecording = false;
        Debug.Log("⏹️ Processing...");
        
        string transcription = await agent.StopRecordingAndTranscribeAsync();
        
        if (!string.IsNullOrEmpty(transcription))
        {
            Debug.Log($"✓ Transcribed: {transcription}");
            
            // Send to agent
            await agent.SendAsync(transcription);
        }
    }
}
```

### Voice Activation Detection (VAD)

```csharp
public class VoiceActivation : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private float noiseThreshold = 0.02f;
    [SerializeField] private float silenceTimeout = 2.0f;
    
    private bool isRecording;
    private float silenceTimer;
    
    void Update()
    {
        if (!isRecording)
        {
            // Check for voice activity
            if (DetectVoice())
            {
                StartAutoRecording();
            }
        }
        else
        {
            // Check for silence
            if (!DetectVoice())
            {
                silenceTimer += Time.deltaTime;
                
                if (silenceTimer >= silenceTimeout)
                {
                    StopAutoRecording();
                }
            }
            else
            {
                silenceTimer = 0;
            }
        }
    }
    
    bool DetectVoice()
    {
        // Simple amplitude detection
        float level = GetMicrophoneLevel();
        return level > noiseThreshold;
    }
    
    float GetMicrophoneLevel()
    {
        // Get current microphone amplitude
        // (Implement using AudioSource.GetOutputData)
        return 0f; // Placeholder
    }
    
    void StartAutoRecording()
    {
        agent.StartRecording();
        isRecording = true;
        silenceTimer = 0;
        
        Debug.Log("🎤 Auto-recording started");
    }
    
    async void StopAutoRecording()
    {
        isRecording = false;
        
        string transcription = await agent.StopRecordingAndTranscribeAsync();
        
        if (!string.IsNullOrEmpty(transcription))
        {
            Debug.Log($"✓ {transcription}");
            await agent.SendAsync(transcription);
        }
    }
}
```

## Language Support

### Auto-Detect Language

```csharp
agent.Settings.Transcription.Language = null; // Auto-detect
string transcription = await agent.TranscribeAsync(audioClip);
```

### Specific Language

```csharp
// English
agent.Settings.Transcription.Language = "en";

// Japanese
agent.Settings.Transcription.Language = "ja";

// Spanish
agent.Settings.Transcription.Language = "es";

// French
agent.Settings.Transcription.Language = "fr";

// Korean
agent.Settings.Transcription.Language = "ko";

// Chinese
agent.Settings.Transcription.Language = "zh";
```

## Context Prompts

### Improve Accuracy

```csharp
// Game-specific vocabulary
agent.Settings.Transcription.Prompt = @"
This is a fantasy RPG game dialogue.
Common terms: mana, health, inventory, quest, dungeon, guild.
";

string transcription = await agent.TranscribeAsync(audioClip);
```

### Character Names

```csharp
// Help recognize specific names
agent.Settings.Transcription.Prompt = @"
Character names in this game:
Aldric, Seraphina, Thorgar, Elara, Grimwald
";
```

## Subtitle Generation

### Real-Time Subtitles

```csharp
public class SubtitleGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text subtitleText;
    [SerializeField] private float displayDuration = 3.0f;
    
    void Start()
    {
        agent.onTranscriptionCompleted.AddListener(ShowSubtitle);
    }
    
    async void ShowSubtitle(string text)
    {
        subtitleText.text = text;
        subtitleText.gameObject.SetActive(true);
        
        await UniTask.Delay((int)(displayDuration * 1000));
        
        subtitleText.gameObject.SetActive(false);
    }
}
```

### NPC Dialogue Transcription

```csharp
public async void TranscribeNPCDialogue(AudioClip npcAudio)
{
    // Transcribe with NPC context
    agent.Settings.Transcription.Prompt = "Fantasy RPG NPC dialogue";
    
    string dialogue = await agent.TranscribeAsync(npcAudio);
    
    // Display as subtitle
    ShowSubtitle(dialogue);
    
    // Save to dialogue log
    SaveToLog(dialogue);
}
```

## Audio Processing

### Batch Transcription

```csharp
public async UniTask<List<string>> TranscribeBatch(AudioClip[] clips)
{
    List<string> transcriptions = new();
    
    foreach (var clip in clips)
    {
        string transcription = await agent.TranscribeAsync(clip);
        transcriptions.Add(transcription);
        
        await UniTask.Delay(100); // Rate limiting
    }
    
    return transcriptions;
}

// Usage
AudioClip[] recordings = GetPlayerRecordings();
var transcriptions = await TranscribeBatch(recordings);
```

### Save Transcriptions

```csharp
public async void TranscribeAndSave(AudioClip clip, string fileName)
{
    string transcription = await agent.TranscribeAsync(clip);
    
    // Save to file
    string path = Path.Combine(Application.persistentDataPath, $"{fileName}.txt");
    File.WriteAllText(path, transcription);
    
    Debug.Log($"Saved transcription: {path}");
}
```

## Voice Commands

### Command Recognition

```csharp
public class VoiceCommands : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, System.Action> commands = new()
    {
        { "open inventory", OpenInventory },
        { "show map", ShowMap },
        { "use potion", UsePotion },
        { "attack", Attack },
        { "defend", Defend }
    };
    
    void Start()
    {
        agent.onTranscriptionCompleted.AddListener(ProcessCommand);
    }
    
    void ProcessCommand(string transcription)
    {
        string normalized = transcription.ToLower().Trim();
        
        foreach (var command in commands)
        {
            if (normalized.Contains(command.Key))
            {
                Debug.Log($"Executing: {command.Key}");
                command.Value?.Invoke();
                return;
            }
        }
        
        Debug.Log($"Unknown command: {transcription}");
    }
    
    void OpenInventory() => Debug.Log("Opening inventory");
    void ShowMap() => Debug.Log("Showing map");
    void UsePotion() => Debug.Log("Using potion");
    void Attack() => Debug.Log("Attacking");
    void Defend() => Debug.Log("Defending");
}
```

### Natural Language Commands

```csharp
public async void ProcessNaturalCommand(string transcription)
{
    // Let AI agent interpret the command
    await agent.SendAsync($@"
User voice command: '{transcription}'
Parse this as a game command and execute the appropriate function.
");
}

// The agent can use function calling to execute commands
agent.AddFunction("open_inventory", () => OpenInventory());
agent.AddFunction("use_item", (string itemName) => UseItem(itemName));
```

## UI Integration

### Recording Indicator

```csharp
public class RecordingUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject recordingIndicator;
    [SerializeField] private Image micIcon;
    
    void Start()
    {
        agent.onRecordingStarted.AddListener(() =>
        {
            recordingIndicator.SetActive(true);
            StartCoroutine(PulseIcon());
        });
        
        agent.onRecordingStopped.AddListener(() =>
        {
            recordingIndicator.SetActive(false);
            StopAllCoroutines();
        });
    }
    
    IEnumerator PulseIcon()
    {
        while (true)
        {
            micIcon.color = Color.red;
            yield return new WaitForSeconds(0.5f);
            micIcon.color = Color.white;
            yield return new WaitForSeconds(0.5f);
        }
    }
}
```

### Transcription Display

```csharp
public class TranscriptionDisplay : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text transcriptionText;
    [SerializeField] private ScrollRect scrollRect;
    
    void Start()
    {
        agent.onTranscriptionCompleted.AddListener(AddTranscription);
    }
    
    void AddTranscription(string text)
    {
        string timestamp = DateTime.Now.ToString("HH:mm:ss");
        transcriptionText.text += $"\n[{timestamp}] {text}";
        
        // Scroll to bottom
        Canvas.ForceUpdateCanvases();
        scrollRect.verticalNormalizedPosition = 0;
    }
}
```

## Error Handling

### Handle Transcription Errors

```csharp
agent.onTranscriptionError.AddListener(error =>
{
    Debug.LogError($"Transcription failed: {error}");
    
    if (error.Contains("no_audio"))
    {
        ShowMessage("No audio detected. Please try again.");
    }
    else if (error.Contains("too_short"))
    {
        ShowMessage("Audio too short. Speak longer.");
    }
    else if (error.Contains("rate_limit"))
    {
        ShowMessage("Too many requests. Please wait.");
    }
    else
    {
        ShowMessage("Transcription failed.");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class SpeechTranscriber : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text transcriptionDisplay;
    [SerializeField] private Button recordButton;
    
    [Header("Settings")]
    [SerializeField] private string language = "en";
    [SerializeField] private float temperature = 0.0f;
    
    private bool isRecording;
    
    async void Start()
    {
        await SetupTranscription();
        
        recordButton.onClick.AddListener(ToggleRecording);
    }
    
    async UniTask SetupTranscription()
    {
        // Configure settings
        agent.Settings.Transcription = new TranscriptionSettings
        {
            Provider = TranscriptionProvider.OpenAI,
            Language = language,
            Temperature = temperature,
            Prompt = "Game voice commands and dialogue"
        };
        
        // Add tool
        agent.AddLocalTool(LocalToolType.SpeechTranscription);
        
        // Listen for events
        agent.onTranscriptionCompleted.AddListener(OnTranscriptionCompleted);
        agent.onTranscriptionError.AddListener(OnTranscriptionError);
        agent.onRecordingStarted.AddListener(() =>
        {
            Debug.Log("🎤 Recording started");
            UpdateRecordButton(true);
        });
        agent.onRecordingStopped.AddListener(() =>
        {
            Debug.Log("⏹️ Recording stopped");
            UpdateRecordButton(false);
        });
        
        Debug.Log("✓ Speech transcription ready");
    }
    
    async void ToggleRecording()
    {
        if (!isRecording)
        {
            StartRecording();
        }
        else
        {
            await StopRecording();
        }
    }
    
    void StartRecording()
    {
        agent.StartRecording();
        isRecording = true;
    }
    
    async UniTask StopRecording()
    {
        isRecording = false;
        
        try
        {
            string transcription = await agent.StopRecordingAndTranscribeAsync();
            
            if (!string.IsNullOrEmpty(transcription))
            {
                Debug.Log($"✓ Transcribed: {transcription}");
                
                // Send to agent for processing
                await agent.SendAsync(transcription);
            }
        }
        catch (Exception ex)
        {
            Debug.LogError($"Transcription failed: {ex.Message}");
        }
    }
    
    void OnTranscriptionCompleted(string text)
    {
        Debug.Log($"✓ Transcription: {text}");
        
        // Display
        string timestamp = DateTime.Now.ToString("HH:mm:ss");
        transcriptionDisplay.text += $"\n[{timestamp}] {text}";
    }
    
    void OnTranscriptionError(string error)
    {
        Debug.LogError($"Transcription error: {error}");
        ShowErrorMessage(error);
    }
    
    void UpdateRecordButton(bool recording)
    {
        var buttonText = recordButton.GetComponentInChildren<TMP_Text>();
        buttonText.text = recording ? "Stop Recording" : "Start Recording";
        
        var buttonImage = recordButton.GetComponent<Image>();
        buttonImage.color = recording ? Color.red : Color.white;
    }
    
    void ShowErrorMessage(string error)
    {
        transcriptionDisplay.text += $"\n<color=red>Error: {error}</color>";
    }
}
```

## Next Steps

- [Audio Input](../../responses/audio-input.md)
- [Speech Generation](speech-generation.md)
- [Voice Settings](../../parameters/voice.md)
