# Audio Setup

Configure voice input and output for your agent.

## Overview

AI Dev Kit supports:

- **Input Audio** - Record and transcribe user speech
- **Output Audio** - Generate speech from agent responses
- **Realtime Audio** - Low-latency bidirectional voice (Realtime API)

## Enabling Audio

### In AgentSettings

```csharp
// Enable input (speech-to-text)
settings.EnableInputAudio = true;
settings.InputAudioParameters = new TranscriptionParameters
{
    Model = "whisper-1",
    SpokenLanguage = SystemLanguage.English
};

// Enable output (text-to-speech)
settings.EnableOutputAudio = true;
settings.OutputAudioParameters = new SpeechParameters
{
    Model = "tts-1",
    Voice = "alloy",
    Speed = 1.0f
};
```

### In AgentBehaviour Inspector

1. Select GameObject with AgentBehaviour
2. Reference AgentSettings with audio enabled
3. Assign InputAudioRecorder component
4. Assign OutputAudioPlayer component

## Input Audio (Speech-to-Text)

### Transcription Parameters

```csharp
public class TranscriptionParameters
{
    public string Model { get; set; }              // "whisper-1"
    public SystemLanguage SpokenLanguage { get; set; }  // English, Spanish, etc.
    public string Prompt { get; set; }              // Optional context
    public float Temperature { get; set; }          // 0.0 - 1.0
}
```

Example:

```csharp
settings.InputAudioParameters = new TranscriptionParameters
{
    Model = "whisper-1",
    SpokenLanguage = SystemLanguage.English,
    Prompt = "This is a technical support conversation",
    Temperature = 0.2f
};
```

### Input Audio Recorder

Provide a recorder component:

```csharp
[SerializeField] private InputAudioRecorder recorder;

void Start()
{
    agentBehaviour.InputAudioRecorder = recorder;
}
```

Or implement custom recorder:

```csharp
public class CustomRecorder : MonoBehaviour, IInputAudioRecorder
{
    public async UniTask<AudioClip> RecordAsync(CancellationToken ct)
    {
        // Custom recording logic
        return recordedClip;
    }
}
```

## Output Audio (Text-to-Speech)

### Speech Parameters

```csharp
public class SpeechParameters
{
    public string Model { get; set; }       // "tts-1", "tts-1-hd"
    public string Voice { get; set; }       // "alloy", "echo", "fable", etc.
    public float Speed { get; set; }        // 0.25 - 4.0
    public AudioFormat Format { get; set; } // mp3, opus, aac, flac
}
```

Example:

```csharp
settings.OutputAudioParameters = new SpeechParameters
{
    Model = "tts-1-hd",
    Voice = "nova",
    Speed = 1.1f,
    Format = AudioFormat.mp3
};
```

### Available Voices

| Voice | Description |
|-------|-------------|
| **alloy** | Neutral, balanced |
| **echo** | Warm, engaging |
| **fable** | British accent |
| **onyx** | Deep, authoritative |
| **nova** | Friendly, conversational |
| **shimmer** | Upbeat, energetic |

### Output Audio Player

Provide a player component:

```csharp
[SerializeField] private OutputAudioPlayer player;

void Start()
{
    agentBehaviour.OutputAudioPlayer = player;
}
```

Or implement custom player:

```csharp
public class CustomPlayer : MonoBehaviour, IOutputAudioPlayer
{
    public async UniTask PlayAsync(AudioClip clip, CancellationToken ct)
    {
        // Custom playback logic
    }
    
    public void Stop()
    {
        // Stop playback
    }
}
```

## Realtime Audio (WebSocket)

For low-latency voice conversations:

```csharp
// In AgentSettings
settings.ChatServiceApi = ChatService.RealtimeApi;
settings.Model = "gpt-4o-realtime-preview";
settings.EnableInputAudio = true;
settings.EnableOutputAudio = true;

// Audio is handled natively by Realtime API
// No separate transcription/synthesis needed
```

## Complete Setup Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class VoiceAssistantSetup : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private InputAudioRecorder recorder;
    [SerializeField] private OutputAudioPlayer player;
    
    void Start()
    {
        // Configure audio
        agent.InputAudioRecorder = recorder;
        agent.OutputAudioPlayer = player;
        agent.OutputAudioVolume = 0.8f;
        
        // Subscribe to audio events
        agent.onInputAudioStarted.AddListener(OnRecordingStarted);
        agent.onInputAudioCompleted.AddListener(OnRecordingCompleted);
        agent.onOutputAudioStarted.AddListener(OnPlaybackStarted);
        agent.onOutputAudioCompleted.AddListener(OnPlaybackCompleted);
    }
    
    public async void OnMicButtonPressed()
    {
        // Record and send audio
        await agent.SendAudioAsync();
    }
    
    void OnRecordingStarted()
    {
        ShowRecordingIndicator();
    }
    
    void OnRecordingCompleted(AudioClip clip)
    {
        HideRecordingIndicator();
        Debug.Log($"Recorded {clip.length} seconds");
    }
    
    void OnPlaybackStarted()
    {
        ShowSpeakerIndicator();
    }
    
    void OnPlaybackCompleted()
    {
        HideSpeakerIndicator();
    }
}
```

## Audio Events

### Input Events

```csharp
agent.onInputAudioStarted.AddListener(() => {
    Debug.Log("Started recording");
});

agent.onInputAudioCompleted.AddListener((clip) => {
    Debug.Log($"Recorded: {clip.length}s");
});

agent.onInputAudioTranscribed.AddListener((text) => {
    Debug.Log($"Transcribed: {text}");
});
```

### Output Events

```csharp
agent.onOutputAudioStarted.AddListener(() => {
    Debug.Log("Started playback");
});

agent.onOutputAudioCompleted.AddListener(() => {
    Debug.Log("Playback finished");
});

agent.onOutputAudioProgress.AddListener((progress) => {
    Debug.Log($"Progress: {progress:P}");
});
```

## Volume Control

```csharp
// Set output volume
agent.OutputAudioVolume = 0.7f; // 0.0 - 1.0

// Adjust at runtime
public void OnVolumeSliderChanged(float value)
{
    agent.OutputAudioVolume = value;
}
```

## Language Support

### Input Language

```csharp
settings.InputAudioParameters.SpokenLanguage = SystemLanguage.English;
// Or: Spanish, French, German, Chinese, Japanese, etc.
```

### Output Language

Output language is determined by the response text language. The TTS model automatically detects and speaks in the appropriate language.

## Audio Quality

### Input Quality

Use appropriate microphone settings:

```csharp
// 16kHz, 16-bit, mono recommended
recorder.SampleRate = 16000;
recorder.Channels = 1;
```

### Output Quality

```csharp
// Standard quality (faster, cheaper)
settings.OutputAudioParameters.Model = "tts-1";

// HD quality (slower, more expensive)
settings.OutputAudioParameters.Model = "tts-1-hd";
```

## Platform Considerations

### Mobile

```csharp
#if UNITY_IOS || UNITY_ANDROID
    // Request microphone permission
    if (!Application.HasUserAuthorization(UserAuthorization.Microphone))
    {
        await Application.RequestUserAuthorization(UserAuthorization.Microphone);
    }
#endif
```

### WebGL

WebGL requires browser microphone permissions. Handle in UI:

```csharp
if (Application.platform == RuntimePlatform.WebGLPlayer)
{
    ShowMicrophonePermissionDialog();
}
```

## Troubleshooting

### No audio recorded

- Check microphone permissions
- Verify InputAudioRecorder is assigned
- Check microphone device is available

### No audio output

- Verify OutputAudioPlayer is assigned
- Check audio volume settings
- Ensure EnableOutputAudio is true

### Poor transcription quality

- Reduce background noise
- Use higher quality microphone
- Add context in Prompt parameter

### Audio lag

- Use Realtime API for lowest latency
- Reduce audio quality if needed
- Check network connection

## Next Steps

- [Input Audio Recorder](../audio/input-recorder.md)
- [Output Audio Player](../audio/output-player.md)
- [Transcription](../audio/transcription.md)
- [Text-to-Speech](../audio/text-to-speech.md)
