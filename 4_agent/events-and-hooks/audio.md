# Audio Events

Monitor audio recording and playback events.

## Overview

Audio Events provide:

- Recording state monitoring
- Playback control events
- Transcription updates
- TTS generation tracking
- Audio error handling

## Basic Setup

### Subscribe to Audio Events

```csharp
public class AudioEventListener : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        var audioController = agent.AudioController;
        
        // Recording events
        audioController.onRecordingStarted.AddListener(OnRecordingStarted);
        audioController.onRecordingStopped.AddListener(OnRecordingStopped);
        
        // Playback events
        audioController.onPlaybackStarted.AddListener(OnPlaybackStarted);
        audioController.onPlaybackCompleted.AddListener(OnPlaybackCompleted);
        
        // Transcription events
        audioController.onTranscriptionStarted.AddListener(OnTranscriptionStarted);
        audioController.onTranscriptionCompleted.AddListener(OnTranscriptionCompleted);
        
        // TTS events
        audioController.onTTSStarted.AddListener(OnTTSStarted);
        audioController.onTTSCompleted.AddListener(OnTTSCompleted);
    }
    
    void OnRecordingStarted()
    {
        Debug.Log("🎤 Recording started");
    }
    
    void OnRecordingStopped(AudioClip clip)
    {
        Debug.Log($"⏹️ Recording stopped: {clip.length}s");
    }
    
    void OnPlaybackStarted(AudioClip clip)
    {
        Debug.Log($"▶️ Playback started: {clip.name}");
    }
    
    void OnPlaybackCompleted()
    {
        Debug.Log("⏹️ Playback completed");
    }
    
    void OnTranscriptionStarted()
    {
        Debug.Log("📝 Transcription started");
    }
    
    void OnTranscriptionCompleted(string text)
    {
        Debug.Log($"✓ Transcription: {text}");
    }
    
    void OnTTSStarted(string text)
    {
        Debug.Log($"🔊 Generating speech: {text}");
    }
    
    void OnTTSCompleted(AudioClip clip)
    {
        Debug.Log($"✓ Speech generated: {clip.length}s");
    }
}
```

## Recording Events

### Monitor Recording State

```csharp
public class RecordingMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Image recordButton;
    [SerializeField] private TMP_Text statusText;
    
    private AudioInputRecorder recorder;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
        
        recorder.onRecordingStarted.AddListener(OnRecordingStarted);
        recorder.onRecordingStopped.AddListener(OnRecordingStopped);
        recorder.onRecordingError.AddListener(OnRecordingError);
        recorder.onLevelChanged.AddListener(OnLevelChanged);
    }
    
    void OnRecordingStarted()
    {
        recordButton.color = Color.red;
        statusText.text = "🎤 Recording...";
        
        Debug.Log("Recording started");
    }
    
    void OnRecordingStopped(AudioClip clip)
    {
        recordButton.color = Color.white;
        statusText.text = "Ready";
        
        Debug.Log($"Recording stopped: {clip.length}s, {clip.samples} samples");
    }
    
    void OnRecordingError(string error)
    {
        Debug.LogError($"Recording error: {error}");
        
        recordButton.color = Color.gray;
        statusText.text = $"<color=red>Error: {error}</color>";
    }
    
    void OnLevelChanged(float level)
    {
        // Update microphone level indicator
        UpdateLevelIndicator(level);
    }
    
    void UpdateLevelIndicator(float level)
    {
        // Visualize microphone input level
    }
}
```

### Voice Activity Detection

```csharp
public class VADEvents : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        var recorder = agent.AudioController.InputRecorder;
        
        recorder.onVoiceDetected.AddListener(OnVoiceDetected);
        recorder.onSilenceDetected.AddListener(OnSilenceDetected);
        recorder.onVADLevelChanged.AddListener(OnVADLevelChanged);
    }
    
    void OnVoiceDetected()
    {
        Debug.Log("🗣️ Voice detected");
        ShowVoiceIndicator(true);
    }
    
    void OnSilenceDetected()
    {
        Debug.Log("🤫 Silence detected");
        ShowVoiceIndicator(false);
    }
    
    void OnVADLevelChanged(float level)
    {
        // Update voice activity level
        UpdateVADIndicator(level);
    }
    
    void ShowVoiceIndicator(bool show)
    {
        // Update UI
    }
    
    void UpdateVADIndicator(float level)
    {
        // Update VAD level display
    }
}
```

## Playback Events

### Monitor Playback State

```csharp
public class PlaybackMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button playButton;
    [SerializeField] private Button pauseButton;
    [SerializeField] private Slider progressSlider;
    [SerializeField] private TMP_Text timeText;
    
    private AudioOutputPlayer player;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        
        player.onPlaybackStarted.AddListener(OnPlaybackStarted);
        player.onPlaybackPaused.AddListener(OnPlaybackPaused);
        player.onPlaybackResumed.AddListener(OnPlaybackResumed);
        player.onPlaybackCompleted.AddListener(OnPlaybackCompleted);
        player.onPlaybackError.AddListener(OnPlaybackError);
        player.onPlaybackProgress.AddListener(OnPlaybackProgress);
    }
    
    void OnPlaybackStarted(AudioClip clip)
    {
        Debug.Log($"▶️ Playing: {clip.name}");
        
        playButton.interactable = false;
        pauseButton.interactable = true;
        
        progressSlider.maxValue = clip.length;
    }
    
    void OnPlaybackPaused()
    {
        Debug.Log("⏸️ Paused");
        
        playButton.interactable = true;
        pauseButton.interactable = false;
    }
    
    void OnPlaybackResumed()
    {
        Debug.Log("▶️ Resumed");
        
        playButton.interactable = false;
        pauseButton.interactable = true;
    }
    
    void OnPlaybackCompleted()
    {
        Debug.Log("⏹️ Playback completed");
        
        playButton.interactable = true;
        pauseButton.interactable = false;
        
        progressSlider.value = 0;
        timeText.text = "00:00";
    }
    
    void OnPlaybackError(string error)
    {
        Debug.LogError($"Playback error: {error}");
        
        playButton.interactable = true;
        pauseButton.interactable = false;
    }
    
    void OnPlaybackProgress(float position, float duration)
    {
        progressSlider.value = position;
        
        timeText.text = $"{FormatTime(position)} / {FormatTime(duration)}";
    }
    
    string FormatTime(float seconds)
    {
        int minutes = Mathf.FloorToInt(seconds / 60);
        int secs = Mathf.FloorToInt(seconds % 60);
        return $"{minutes:00}:{secs:00}";
    }
}
```

## Transcription Events

### Monitor Transcription Progress

```csharp
public class TranscriptionMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text transcriptionText;
    [SerializeField] private GameObject loadingIndicator;
    
    private SpeechTranscriber transcriber;
    
    void Start()
    {
        transcriber = agent.AudioController.Transcriber;
        
        transcriber.onTranscriptionStarted.AddListener(OnTranscriptionStarted);
        transcriber.onTranscriptionProgress.AddListener(OnTranscriptionProgress);
        transcriber.onTranscriptionCompleted.AddListener(OnTranscriptionCompleted);
        transcriber.onTranscriptionError.AddListener(OnTranscriptionError);
    }
    
    void OnTranscriptionStarted()
    {
        Debug.Log("📝 Transcription started");
        
        transcriptionText.text = "Transcribing...";
        loadingIndicator.SetActive(true);
    }
    
    void OnTranscriptionProgress(float progress)
    {
        Debug.Log($"Progress: {progress:P0}");
        
        transcriptionText.text = $"Transcribing... {progress:P0}";
    }
    
    void OnTranscriptionCompleted(string text)
    {
        Debug.Log($"✓ Transcription completed: {text}");
        
        transcriptionText.text = text;
        loadingIndicator.SetActive(false);
    }
    
    void OnTranscriptionError(string error)
    {
        Debug.LogError($"Transcription error: {error}");
        
        transcriptionText.text = $"<color=red>Error: {error}</color>";
        loadingIndicator.SetActive(false);
    }
}
```

## TTS Events

### Monitor Speech Generation

```csharp
public class TTSMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text statusText;
    [SerializeField] private Slider progressBar;
    
    void Start()
    {
        var tts = agent.AudioController.TextToSpeech;
        
        tts.onTTSStarted.AddListener(OnTTSStarted);
        tts.onTTSProgress.AddListener(OnTTSProgress);
        tts.onTTSCompleted.AddListener(OnTTSCompleted);
        tts.onTTSError.AddListener(OnTTSError);
    }
    
    void OnTTSStarted(string text)
    {
        Debug.Log($"🔊 Generating speech: {text.Substring(0, Math.Min(50, text.Length))}...");
        
        statusText.text = "Generating speech...";
        progressBar.value = 0;
    }
    
    void OnTTSProgress(float progress)
    {
        progressBar.value = progress;
        statusText.text = $"Generating... {progress:P0}";
    }
    
    void OnTTSCompleted(AudioClip clip)
    {
        Debug.Log($"✓ Speech generated: {clip.length}s");
        
        statusText.text = "Ready";
        progressBar.value = 1f;
        
        // Auto-play
        agent.AudioController.OutputPlayer.Play(clip);
    }
    
    void OnTTSError(string error)
    {
        Debug.LogError($"TTS error: {error}");
        
        statusText.text = $"<color=red>Error: {error}</color>";
        progressBar.value = 0;
    }
}
```

## Voice Command Events

### Track Voice Commands

```csharp
public class VoiceCommandEvents : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AudioController.onVoiceCommandDetected.AddListener(OnVoiceCommandDetected);
        agent.AudioController.onVoiceCommandExecuted.AddListener(OnVoiceCommandExecuted);
        agent.AudioController.onVoiceCommandError.AddListener(OnVoiceCommandError);
    }
    
    void OnVoiceCommandDetected(string command)
    {
        Debug.Log($"🎙️ Voice command detected: {command}");
        
        ShowCommandFeedback(command);
    }
    
    void OnVoiceCommandExecuted(string command)
    {
        Debug.Log($"✓ Command executed: {command}");
        
        HideCommandFeedback();
    }
    
    void OnVoiceCommandError(string command, string error)
    {
        Debug.LogError($"❌ Command error: {command} - {error}");
        
        ShowErrorFeedback(error);
    }
    
    void ShowCommandFeedback(string command)
    {
        // Show UI feedback
    }
    
    void HideCommandFeedback()
    {
        // Hide UI feedback
    }
    
    void ShowErrorFeedback(string error)
    {
        // Show error message
    }
}
```

## Audio Queue Events

### Monitor Audio Queue

```csharp
public class AudioQueueMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text queueCountText;
    
    void Start()
    {
        var player = agent.AudioController.OutputPlayer;
        
        player.onAudioQueued.AddListener(OnAudioQueued);
        player.onQueueChanged.AddListener(OnQueueChanged);
        player.onQueueEmpty.AddListener(OnQueueEmpty);
    }
    
    void OnAudioQueued(AudioClip clip)
    {
        Debug.Log($"➕ Audio queued: {clip.name}");
    }
    
    void OnQueueChanged(int count)
    {
        Debug.Log($"Queue size: {count}");
        queueCountText.text = $"Queue: {count}";
    }
    
    void OnQueueEmpty()
    {
        Debug.Log("Queue empty");
        queueCountText.text = "Queue: 0";
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AudioEventManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("UI")]
    [SerializeField] private Image recordIndicator;
    [SerializeField] private Image playbackIndicator;
    [SerializeField] private TMP_Text statusText;
    [SerializeField] private Slider levelMeter;
    [SerializeField] private Slider progressBar;
    
    [Header("Settings")]
    [SerializeField] private bool enableLogging = true;
    
    private AudioController audioController;
    
    void Start()
    {
        audioController = agent.AudioController;
        RegisterAllEvents();
    }
    
    void RegisterAllEvents()
    {
        // Recording events
        audioController.InputRecorder.onRecordingStarted.AddListener(OnRecordingStarted);
        audioController.InputRecorder.onRecordingStopped.AddListener(OnRecordingStopped);
        audioController.InputRecorder.onRecordingError.AddListener(OnRecordingError);
        audioController.InputRecorder.onLevelChanged.AddListener(OnLevelChanged);
        
        // Playback events
        audioController.OutputPlayer.onPlaybackStarted.AddListener(OnPlaybackStarted);
        audioController.OutputPlayer.onPlaybackCompleted.AddListener(OnPlaybackCompleted);
        audioController.OutputPlayer.onPlaybackError.AddListener(OnPlaybackError);
        audioController.OutputPlayer.onPlaybackProgress.AddListener(OnPlaybackProgress);
        
        // Transcription events
        audioController.Transcriber.onTranscriptionStarted.AddListener(OnTranscriptionStarted);
        audioController.Transcriber.onTranscriptionCompleted.AddListener(OnTranscriptionCompleted);
        audioController.Transcriber.onTranscriptionError.AddListener(OnTranscriptionError);
        
        // TTS events
        audioController.TextToSpeech.onTTSStarted.AddListener(OnTTSStarted);
        audioController.TextToSpeech.onTTSCompleted.AddListener(OnTTSCompleted);
        audioController.TextToSpeech.onTTSError.AddListener(OnTTSError);
        
        if (enableLogging)
            Debug.Log("✓ Audio events registered");
    }
    
    // Recording Events
    
    void OnRecordingStarted()
    {
        if (enableLogging)
            Debug.Log("🎤 Recording started");
        
        recordIndicator.color = Color.red;
        statusText.text = "Recording...";
    }
    
    void OnRecordingStopped(AudioClip clip)
    {
        if (enableLogging)
            Debug.Log($"⏹️ Recording stopped: {clip.length}s");
        
        recordIndicator.color = Color.white;
        statusText.text = "Processing...";
    }
    
    void OnRecordingError(string error)
    {
        Debug.LogError($"Recording error: {error}");
        
        recordIndicator.color = Color.gray;
        statusText.text = $"<color=red>Error</color>";
    }
    
    void OnLevelChanged(float level)
    {
        levelMeter.value = level;
    }
    
    // Playback Events
    
    void OnPlaybackStarted(AudioClip clip)
    {
        if (enableLogging)
            Debug.Log($"▶️ Playing: {clip.name}");
        
        playbackIndicator.color = Color.green;
        statusText.text = $"Playing: {clip.name}";
    }
    
    void OnPlaybackCompleted()
    {
        if (enableLogging)
            Debug.Log("⏹️ Playback completed");
        
        playbackIndicator.color = Color.white;
        statusText.text = "Ready";
        progressBar.value = 0;
    }
    
    void OnPlaybackError(string error)
    {
        Debug.LogError($"Playback error: {error}");
        
        playbackIndicator.color = Color.gray;
        statusText.text = "<color=red>Playback error</color>";
    }
    
    void OnPlaybackProgress(float position, float duration)
    {
        progressBar.value = position / duration;
    }
    
    // Transcription Events
    
    void OnTranscriptionStarted()
    {
        if (enableLogging)
            Debug.Log("📝 Transcription started");
        
        statusText.text = "Transcribing...";
    }
    
    void OnTranscriptionCompleted(string text)
    {
        if (enableLogging)
            Debug.Log($"✓ Transcribed: {text}");
        
        statusText.text = "Ready";
    }
    
    void OnTranscriptionError(string error)
    {
        Debug.LogError($"Transcription error: {error}");
        
        statusText.text = "<color=red>Transcription failed</color>";
    }
    
    // TTS Events
    
    void OnTTSStarted(string text)
    {
        if (enableLogging)
            Debug.Log($"🔊 Generating speech: {text.Substring(0, Math.Min(50, text.Length))}...");
        
        statusText.text = "Generating speech...";
    }
    
    void OnTTSCompleted(AudioClip clip)
    {
        if (enableLogging)
            Debug.Log($"✓ Speech generated: {clip.length}s");
        
        statusText.text = "Ready";
    }
    
    void OnTTSError(string error)
    {
        Debug.LogError($"TTS error: {error}");
        
        statusText.text = "<color=red>TTS failed</color>";
    }
    
    void OnDestroy()
    {
        if (audioController != null)
        {
            // Cleanup all listeners
            audioController.InputRecorder.onRecordingStarted.RemoveListener(OnRecordingStarted);
            audioController.InputRecorder.onRecordingStopped.RemoveListener(OnRecordingStopped);
            audioController.InputRecorder.onRecordingError.RemoveListener(OnRecordingError);
            audioController.InputRecorder.onLevelChanged.RemoveListener(OnLevelChanged);
            
            audioController.OutputPlayer.onPlaybackStarted.RemoveListener(OnPlaybackStarted);
            audioController.OutputPlayer.onPlaybackCompleted.RemoveListener(OnPlaybackCompleted);
            audioController.OutputPlayer.onPlaybackError.RemoveListener(OnPlaybackError);
            audioController.OutputPlayer.onPlaybackProgress.RemoveListener(OnPlaybackProgress);
            
            audioController.Transcriber.onTranscriptionStarted.RemoveListener(OnTranscriptionStarted);
            audioController.Transcriber.onTranscriptionCompleted.RemoveListener(OnTranscriptionCompleted);
            audioController.Transcriber.onTranscriptionError.RemoveListener(OnTranscriptionError);
            
            audioController.TextToSpeech.onTTSStarted.RemoveListener(OnTTSStarted);
            audioController.TextToSpeech.onTTSCompleted.RemoveListener(OnTTSCompleted);
            audioController.TextToSpeech.onTTSError.RemoveListener(OnTTSError);
        }
    }
}
```

## Next Steps

- [Agent Lifecycle Hooks](agent-hooks.md)
- [Text Delta Events](text-delta.md)
- [Tool Call Events](tool-calls.md)
- [Status Events](status.md)
- [Error Handling Events](error-handling.md)
