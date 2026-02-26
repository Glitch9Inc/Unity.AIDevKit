---
icon: microphone
---

# Audio

Configure and manage voice input/output for your agent.

## Overview

AI Dev Kit supports:

- **Input Audio Recorder** - Record user speech
- **Output Audio Player** - Play agent speech
- **Transcription** - Speech-to-text
- **Text-to-Speech** - Generate speech from text

## Input Audio

### Recording

```csharp
[SerializeField] private InputAudioRecorder recorder;

// Start recording
recorder.StartRecording();

// Stop and get clip
AudioClip clip = recorder.StopRecording();

// Send to agent
await agent.SendAsync(clip);
```

### Automatic Recording

```csharp
// Agent handles recording automatically
await agent.SendAudioAsync();
```

### Events

```csharp
agent.onInputAudioStarted.AddListener(() => {
    ShowRecordingIndicator();
});

agent.onInputAudioCompleted.AddListener((clip) => {
    HideRecordingIndicator();
    Debug.Log($"Recorded: {clip.length}s");
});

agent.onInputAudioTranscribed.AddListener((text) => {
    Debug.Log($"You said: {text}");
});
```

## Output Audio

### Playback

```csharp
[SerializeField] private OutputAudioPlayer player;

// Agent generates and plays audio automatically
await agent.SendAsync("Hello!");
// Speech plays automatically if EnableOutputAudio = true
```

### Manual Playback

```csharp
// Generate audio
AudioClip clip = await agent.GenerateSpeechAsync("Hello!");

// Play manually
player.Play(clip);
```

### Events

```csharp
agent.onOutputAudioStarted.AddListener(() => {
    ShowSpeakerIndicator();
});

agent.onOutputAudioCompleted.AddListener(() => {
    HideSpeakerIndicator();
});

agent.onOutputAudioProgress.AddListener((progress) => {
    UpdateProgressBar(progress);
});
```

## Transcription

### Configure

```csharp
settings.InputAudioParameters = new TranscriptionParameters
{
    Model = "whisper-1",
    SpokenLanguage = SystemLanguage.English,
    Prompt = "Technical support conversation",
    Temperature = 0.2f
};
```

### Manual Transcription

```csharp
AudioClip clip = GetAudioClip();
string text = await agent.TranscribeAsync(clip);
Debug.Log($"Transcribed: {text}");
```

## Text-to-Speech

### Configure

```csharp
settings.OutputAudioParameters = new SpeechParameters
{
    Model = "tts-1-hd",
    Voice = "nova",
    Speed = 1.0f,
    Format = AudioFormat.mp3
};
```

### Manual TTS

```csharp
AudioClip clip = await agent.GenerateSpeechAsync("Hello!");
audioSource.PlayOneShot(clip);
```

## Volume Control

```csharp
// Set volume
agent.OutputAudioVolume = 0.8f;

// UI slider
public void OnVolumeChanged(float value)
{
    agent.OutputAudioVolume = value;
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class VoiceChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private InputAudioRecorder recorder;
    [SerializeField] private OutputAudioPlayer player;
    
    void Start()
    {
        // Setup
        agent.InputAudioRecorder = recorder;
        agent.OutputAudioPlayer = player;
        
        // Subscribe to events
        agent.onInputAudioStarted.AddListener(OnRecordingStarted);
        agent.onInputAudioCompleted.AddListener(OnRecordingCompleted);
        agent.onInputAudioTranscribed.AddListener(OnTranscribed);
        agent.onOutputAudioStarted.AddListener(OnPlaybackStarted);
        agent.onOutputAudioCompleted.AddListener(OnPlaybackCompleted);
    }
    
    public async void OnMicButtonPressed()
    {
        // Record and send
        await agent.SendAudioAsync();
    }
    
    void OnRecordingStarted()
    {
        Debug.Log("Recording...");
    }
    
    void OnRecordingCompleted(AudioClip clip)
    {
        Debug.Log($"Recorded {clip.length}s");
    }
    
    void OnTranscribed(string text)
    {
        Debug.Log($"You: {text}");
    }
    
    void OnPlaybackStarted()
    {
        Debug.Log("Playing response...");
    }
    
    void OnPlaybackCompleted()
    {
        Debug.Log("Playback complete");
    }
}
```

## Next Steps

- [Input Audio Recorder](input-recorder.md)
- [Output Audio Player](output-player.md)
- [Transcription](transcription.md)
- [Text-to-Speech](text-to-speech.md)
- [Audio Setup](../configuration/audio-setup.md)
