# Audio Input

Send audio messages to your agent using voice input.

## Overview

Audio input enables:

- **Voice conversations** - Speak to your agent
- **Audio transcription** - Convert speech to text
- **Multi-modal interaction** - Combine voice and text

## Basic Audio Input

### Send Audio Clip

```csharp
// Load or record audio clip
AudioClip audioClip = GetAudioClip();

// Send to agent
Response response = await agent.SendAsync(audioClip);

Debug.Log($"Transcription: {response.Text}");
```

### Using Input Audio Recorder

```csharp
[SerializeField] private AgentBehaviour agent;
[SerializeField] private InputAudioRecorder recorder;

void Start()
{
    agent.InputAudioRecorder = recorder;
}

public async void RecordAndSend()
{
    // Agent handles recording automatically
    await agent.SendAudioAsync();
}
```

## Recording Audio

### Manual Recording

```csharp
[SerializeField] private InputAudioRecorder recorder;

public async void ManualRecord()
{
    // Start recording
    recorder.StartRecording();
    
    // Wait for user to finish (e.g., button release)
    await UniTask.WaitUntil(() => Input.GetKeyUp(KeyCode.Space));
    
    // Stop and get clip
    AudioClip clip = recorder.StopRecording();
    
    // Send to agent
    await agent.SendAsync(clip);
}
```

### Push-to-Talk

```csharp
public class PushToTalk : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private InputAudioRecorder recorder;
    
    private bool isRecording = false;
    
    void Update()
    {
        // Hold Space to record
        if (Input.GetKeyDown(KeyCode.Space) && !isRecording)
        {
            StartRecording();
        }
        else if (Input.GetKeyUp(KeyCode.Space) && isRecording)
        {
            StopRecording();
        }
    }
    
    async void StartRecording()
    {
        isRecording = true;
        recorder.StartRecording();
        Debug.Log("🎤 Recording...");
    }
    
    async void StopRecording()
    {
        isRecording = false;
        AudioClip clip = recorder.StopRecording();
        
        Debug.Log("⏹ Stopped recording");
        
        // Send to agent
        await agent.SendAsync(clip);
    }
}
```

### Voice Activity Detection

```csharp
public class VoiceActivation : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private InputAudioRecorder recorder;
    
    [SerializeField] private float threshold = 0.02f;
    [SerializeField] private float silenceDuration = 2f;
    
    private bool isRecording = false;
    private float silenceTimer = 0f;
    
    void Update()
    {
        float volume = GetMicrophoneVolume();
        
        if (volume > threshold)
        {
            if (!isRecording)
            {
                StartRecording();
            }
            silenceTimer = 0f;
        }
        else if (isRecording)
        {
            silenceTimer += Time.deltaTime;
            
            if (silenceTimer >= silenceDuration)
            {
                StopRecording();
            }
        }
    }
    
    float GetMicrophoneVolume()
    {
        // Get volume from recorder
        return recorder.GetCurrentVolume();
    }
    
    async void StartRecording()
    {
        isRecording = true;
        recorder.StartRecording();
        Debug.Log("🎤 Voice detected, recording...");
    }
    
    async void StopRecording()
    {
        isRecording = false;
        silenceTimer = 0f;
        
        AudioClip clip = recorder.StopRecording();
        Debug.Log("⏹ Silence detected, sending...");
        
        await agent.SendAsync(clip);
    }
}
```

## Audio Events

### Recording Lifecycle

```csharp
void Start()
{
    agent.onInputAudioStarted.AddListener(OnRecordingStarted);
    agent.onInputAudioCompleted.AddListener(OnRecordingCompleted);
    agent.onInputAudioTranscribed.AddListener(OnTranscribed);
}

void OnRecordingStarted()
{
    Debug.Log("🎤 Recording started");
    ShowRecordingIndicator();
}

void OnRecordingCompleted(AudioClip clip)
{
    Debug.Log($"⏹ Recording completed: {clip.length}s");
    HideRecordingIndicator();
    ShowTranscribingIndicator();
}

void OnTranscribed(string text)
{
    Debug.Log($"📝 Transcribed: {text}");
    HideTranscribingIndicator();
    DisplayTranscription(text);
}
```

### UnityEvents (AgentBehaviour)

```csharp
// In Inspector, bind to UI elements
public class AudioUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject recordingIndicator;
    [SerializeField] private TMP_Text transcriptionText;
    
    void Start()
    {
        // Bind in Inspector or code
        agent.onInputAudioStarted.AddListener(() =>
        {
            recordingIndicator.SetActive(true);
        });
        
        agent.onInputAudioCompleted.AddListener((clip) =>
        {
            recordingIndicator.SetActive(false);
        });
        
        agent.onInputAudioTranscribed.AddListener((text) =>
        {
            transcriptionText.text = text;
        });
    }
}
```

## Transcription Settings

### Configure Language

```csharp
// Set in AgentSettings or at runtime
agent.InputAudioLanguage = SystemLanguage.English;

// Or Japanese
agent.InputAudioLanguage = SystemLanguage.Japanese;
```

### Transcription Parameters

```csharp
// In AgentSettings.InputAudioParameters
settings.InputAudioParameters = new TranscriptionParameters
{
    Model = "whisper-1",
    SpokenLanguage = SystemLanguage.English,
    Prompt = "Technical conversation about Unity and game development",
    Temperature = 0.2f  // Lower = more accurate
};
```

### Custom Prompt for Context

```csharp
// Improve accuracy with context
agent.Settings.InputAudioParameters.Prompt = @"
This is a conversation about Unity game development.
Common terms: GameObject, MonoBehaviour, coroutine, prefab, shader.
";
```

## Audio Quality

### Check Microphone

```csharp
void Start()
{
    if (Microphone.devices.Length == 0)
    {
        Debug.LogError("No microphone detected!");
        return;
    }
    
    Debug.Log($"Using microphone: {Microphone.devices[0]}");
}
```

### Validate Audio Clip

```csharp
bool IsAudioClipValid(AudioClip clip)
{
    if (clip == null)
    {
        Debug.LogError("Audio clip is null");
        return false;
    }
    
    if (clip.length < 0.1f)
    {
        Debug.LogWarning("Audio clip too short");
        return false;
    }
    
    if (clip.length > 60f)
    {
        Debug.LogWarning("Audio clip too long");
        return false;
    }
    
    return true;
}
```

### Noise Reduction

```csharp
public AudioClip ReduceNoise(AudioClip original)
{
    // Get samples
    float[] samples = new float[original.samples * original.channels];
    original.GetData(samples, 0);
    
    // Apply simple noise gate
    float threshold = 0.01f;
    for (int i = 0; i < samples.Length; i++)
    {
        if (Mathf.Abs(samples[i]) < threshold)
        {
            samples[i] = 0f;
        }
    }
    
    // Create new clip
    AudioClip filtered = AudioClip.Create(
        "Filtered",
        original.samples,
        original.channels,
        original.frequency,
        false
    );
    filtered.SetData(samples, 0);
    
    return filtered;
}
```

## Combining Audio and Text

### Send Both

```csharp
// Send audio for transcription, but also include text context
public async UniTask SendAudioWithContext(AudioClip audio, string context)
{
    // First send context
    await agent.SendAsync(context);
    
    // Then send audio
    await agent.SendAsync(audio);
    
    // Agent has context from previous message
}
```

### Pre-transcribe and Edit

```csharp
public async UniTask SendAudioEditable(AudioClip audio)
{
    // Transcribe first
    string transcription = await agent.TranscribeAsync(audio);
    
    // Let user edit transcription
    string edited = await ShowEditDialog(transcription);
    
    // Send edited text
    await agent.SendAsync(edited);
}
```

## Error Handling

### Recording Errors

```csharp
try
{
    AudioClip clip = await RecordAudioAsync();
    await agent.SendAsync(clip);
}
catch (MicrophoneException ex)
{
    Debug.LogError($"Microphone error: {ex.Message}");
    ShowError("Microphone access denied");
}
catch (Exception ex)
{
    Debug.LogError($"Recording failed: {ex.Message}");
}
```

### Transcription Errors

```csharp
agent.onError.AddListener(OnError);

void OnError(string error)
{
    if (error.Contains("transcription"))
    {
        Debug.LogError("Transcription failed");
        ShowError("Could not understand audio. Please try again.");
    }
}
```

## Performance Tips

### 1. Limit Recording Duration

```csharp
private const float maxRecordingDuration = 30f;
private float recordingStartTime;

void StartRecording()
{
    recorder.StartRecording();
    recordingStartTime = Time.time;
}

void Update()
{
    if (isRecording)
    {
        if (Time.time - recordingStartTime >= maxRecordingDuration)
        {
            StopRecording();
            Debug.Log("Max recording duration reached");
        }
    }
}
```

### 2. Compress Audio

```csharp
public AudioClip CompressAudio(AudioClip original)
{
    // Downsample to 16kHz (Whisper's native rate)
    int targetFrequency = 16000;
    
    if (original.frequency <= targetFrequency)
    {
        return original;
    }
    
    // Resample audio
    // (Implementation depends on audio library)
    return ResampleAudio(original, targetFrequency);
}
```

### 3. Cache Microphone Access

```csharp
private static string cachedMicrophone;

string GetMicrophone()
{
    if (cachedMicrophone == null)
    {
        if (Microphone.devices.Length > 0)
        {
            cachedMicrophone = Microphone.devices[0];
        }
    }
    
    return cachedMicrophone;
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using TMPro;

public class VoiceInput : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private InputAudioRecorder recorder;
    
    [Header("UI")]
    [SerializeField] private GameObject recordButton;
    [SerializeField] private GameObject stopButton;
    [SerializeField] private GameObject recordingIndicator;
    [SerializeField] private TMP_Text transcriptionText;
    [SerializeField] private TMP_Text statusText;
    
    [Header("Settings")]
    [SerializeField] private float maxRecordingDuration = 30f;
    
    private bool isRecording = false;
    private float recordingStartTime;
    
    void Start()
    {
        // Setup agent
        agent.InputAudioRecorder = recorder;
        agent.InputAudioLanguage = SystemLanguage.English;
        
        // Setup events
        agent.onInputAudioStarted.AddListener(OnRecordingStarted);
        agent.onInputAudioCompleted.AddListener(OnRecordingCompleted);
        agent.onInputAudioTranscribed.AddListener(OnTranscribed);
        agent.onError.AddListener(OnError);
        
        // Check microphone
        if (Microphone.devices.Length == 0)
        {
            statusText.text = "No microphone detected";
            recordButton.SetActive(false);
        }
    }
    
    void Update()
    {
        // Check max duration
        if (isRecording)
        {
            float duration = Time.time - recordingStartTime;
            statusText.text = $"Recording: {duration:F1}s / {maxRecordingDuration}s";
            
            if (duration >= maxRecordingDuration)
            {
                StopRecording();
            }
        }
    }
    
    public void StartRecording()
    {
        isRecording = true;
        recordingStartTime = Time.time;
        
        recorder.StartRecording();
        
        recordButton.SetActive(false);
        stopButton.SetActive(true);
        recordingIndicator.SetActive(true);
        transcriptionText.text = "";
    }
    
    public async void StopRecording()
    {
        if (!isRecording) return;
        
        isRecording = false;
        
        AudioClip clip = recorder.StopRecording();
        
        recordButton.SetActive(true);
        stopButton.SetActive(false);
        recordingIndicator.SetActive(false);
        statusText.text = "Processing...";
        
        try
        {
            await agent.SendAsync(clip);
        }
        catch (Exception ex)
        {
            Debug.LogError($"Error: {ex.Message}");
            statusText.text = "Error: " + ex.Message;
        }
    }
    
    void OnRecordingStarted()
    {
        Debug.Log("🎤 Recording started");
    }
    
    void OnRecordingCompleted(AudioClip clip)
    {
        Debug.Log($"⏹ Recording completed: {clip.length}s");
        statusText.text = "Transcribing...";
    }
    
    void OnTranscribed(string text)
    {
        Debug.Log($"📝 Transcribed: {text}");
        transcriptionText.text = $"You: {text}";
        statusText.text = "Ready";
    }
    
    void OnError(string error)
    {
        Debug.LogError($"Error: {error}");
        statusText.text = $"Error: {error}";
        
        isRecording = false;
        recordButton.SetActive(true);
        stopButton.SetActive(false);
        recordingIndicator.SetActive(false);
    }
}
```

## Next Steps

- [Text Messages](text-messages.md)
- [File Attachments](file-attachments.md)
- [Input Audio Recorder](../audio/input-recorder.md)
- [Transcription](../audio/transcription.md)
