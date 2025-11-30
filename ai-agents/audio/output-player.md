# Audio Output Player

Play audio responses from AI agents.

## Overview

Audio Output Player provides:
- Audio playback control
- Volume and speed adjustment
- Playlist management
- Audio effects
- Spatial audio support

## Basic Setup

### Initialize Player

```csharp
var audioPlayer = agent.AudioController.OutputPlayer;

// Configure settings
audioPlayer.Volume = 1.0f;
audioPlayer.Speed = 1.0f;
audioPlayer.EnableEffects = true;
```

## Playback Controls

### Play Audio

```csharp
public class AudioPlayer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private AudioSource audioSource;
    
    private AudioOutputPlayer player;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        player.AudioSource = audioSource;
    }
    
    public void PlayAudio(AudioClip clip)
    {
        player.Play(clip);
        Debug.Log($"▶️ Playing: {clip.length}s");
    }
    
    public void Pause()
    {
        player.Pause();
        Debug.Log("⏸️ Paused");
    }
    
    public void Resume()
    {
        player.Resume();
        Debug.Log("▶️ Resumed");
    }
    
    public void Stop()
    {
        player.Stop();
        Debug.Log("⏹️ Stopped");
    }
}
```

### Auto-Play Agent Responses

```csharp
public class AutoPlayResponses : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onAudioResponseReceived.AddListener(PlayResponse);
    }
    
    void PlayResponse(AudioClip clip)
    {
        Debug.Log($"🔊 Playing agent response: {clip.length}s");
        
        var player = agent.AudioController.OutputPlayer;
        player.Play(clip);
    }
}
```

## Volume Control

### Set Volume

```csharp
public class VolumeController : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Slider volumeSlider;
    
    private AudioOutputPlayer player;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        
        volumeSlider.onValueChanged.AddListener(SetVolume);
        volumeSlider.value = player.Volume;
    }
    
    void SetVolume(float volume)
    {
        player.Volume = volume;
        Debug.Log($"🔊 Volume: {volume:P0}");
    }
}
```

### Fade In/Out

```csharp
public async UniTask FadeIn(float duration)
{
    float startVolume = 0;
    float targetVolume = player.Volume;
    float elapsed = 0;
    
    player.Volume = startVolume;
    
    while (elapsed < duration)
    {
        elapsed += Time.deltaTime;
        player.Volume = Mathf.Lerp(startVolume, targetVolume, elapsed / duration);
        await UniTask.Yield();
    }
    
    player.Volume = targetVolume;
}

public async UniTask FadeOut(float duration)
{
    float startVolume = player.Volume;
    float elapsed = 0;
    
    while (elapsed < duration)
    {
        elapsed += Time.deltaTime;
        player.Volume = Mathf.Lerp(startVolume, 0, elapsed / duration);
        await UniTask.Yield();
    }
    
    player.Volume = 0;
    player.Stop();
}
```

## Speed Control

### Adjust Playback Speed

```csharp
public class SpeedController : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Slider speedSlider;
    [SerializeField] private TMP_Text speedText;
    
    private AudioOutputPlayer player;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        
        speedSlider.minValue = 0.5f;
        speedSlider.maxValue = 2.0f;
        speedSlider.value = 1.0f;
        
        speedSlider.onValueChanged.AddListener(SetSpeed);
    }
    
    void SetSpeed(float speed)
    {
        player.Speed = speed;
        speedText.text = $"{speed:F1}x";
        
        Debug.Log($"⚡ Speed: {speed}x");
    }
}
```

## Playlist Management

### Audio Queue

```csharp
public class AudioPlaylist : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Queue<AudioClip> playlist = new();
    private AudioOutputPlayer player;
    private bool isPlaying;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        player.onPlaybackCompleted.AddListener(OnClipFinished);
    }
    
    public void AddToQueue(AudioClip clip)
    {
        playlist.Enqueue(clip);
        Debug.Log($"➕ Added to queue: {playlist.Count} clips");
        
        if (!isPlaying)
        {
            PlayNext();
        }
    }
    
    void PlayNext()
    {
        if (playlist.Count == 0)
        {
            isPlaying = false;
            Debug.Log("✓ Playlist finished");
            return;
        }
        
        AudioClip clip = playlist.Dequeue();
        player.Play(clip);
        isPlaying = true;
        
        Debug.Log($"▶️ Playing: {clip.name} ({playlist.Count} remaining)");
    }
    
    void OnClipFinished()
    {
        PlayNext();
    }
    
    public void ClearQueue()
    {
        playlist.Clear();
        player.Stop();
        isPlaying = false;
        
        Debug.Log("🗑️ Queue cleared");
    }
}
```

### Skip Controls

```csharp
public void SkipCurrent()
{
    player.Stop();
    Debug.Log("⏭️ Skipped");
}

public void SkipToEnd()
{
    playlist.Clear();
    player.Stop();
    isPlaying = false;
    
    Debug.Log("⏭️ Skipped to end");
}
```

## Audio Effects

### Apply Effects

```csharp
public class AudioEffects : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private bool enableReverb = true;
    [SerializeField] private bool enableEcho = false;
    
    void Start()
    {
        if (enableReverb)
        {
            var reverb = audioSource.gameObject.AddComponent<AudioReverbFilter>();
            reverb.reverbPreset = AudioReverbPreset.Room;
        }
        
        if (enableEcho)
        {
            var echo = audioSource.gameObject.AddComponent<AudioEchoFilter>();
            echo.delay = 500f;
            echo.decayRatio = 0.5f;
        }
    }
}
```

### Equalizer

```csharp
public class AudioEqualizer : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    
    [Header("EQ Bands")]
    [SerializeField] private float lowFreqGain = 1.0f;
    [SerializeField] private float midFreqGain = 1.0f;
    [SerializeField] private float highFreqGain = 1.0f;
    
    private AudioLowPassFilter lowPass;
    private AudioHighPassFilter highPass;
    
    void Start()
    {
        lowPass = audioSource.gameObject.AddComponent<AudioLowPassFilter>();
        highPass = audioSource.gameObject.AddComponent<AudioHighPassFilter>();
        
        ApplyEQ();
    }
    
    void ApplyEQ()
    {
        lowPass.cutoffFrequency = 5000f * lowFreqGain;
        highPass.cutoffFrequency = 1000f / highFreqGain;
    }
    
    public void SetBand(int band, float gain)
    {
        switch (band)
        {
            case 0: lowFreqGain = gain; break;
            case 1: midFreqGain = gain; break;
            case 2: highFreqGain = gain; break;
        }
        
        ApplyEQ();
    }
}
```

## Spatial Audio

### 3D Audio Setup

```csharp
public class SpatialAudio : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Transform listenerTransform;
    
    private AudioSource audioSource;
    
    void Start()
    {
        audioSource = agent.AudioController.OutputPlayer.AudioSource;
        
        // Configure for 3D audio
        audioSource.spatialBlend = 1.0f; // Fully 3D
        audioSource.rolloffMode = AudioRolloffMode.Linear;
        audioSource.minDistance = 1f;
        audioSource.maxDistance = 20f;
        
        // Set audio listener
        AudioListener.transform = listenerTransform;
    }
    
    public void SetPosition(Vector3 position)
    {
        audioSource.transform.position = position;
    }
}
```

### Directional Audio

```csharp
public void ConfigureDirectionalAudio(float spread)
{
    audioSource.spatialBlend = 1.0f;
    audioSource.spread = spread; // 0-360 degrees
    audioSource.dopplerLevel = 1.0f;
}
```

## UI Integration

### Playback Controls UI

```csharp
public class PlaybackControlsUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button playButton;
    [SerializeField] private Button pauseButton;
    [SerializeField] private Button stopButton;
    [SerializeField] private Slider progressSlider;
    [SerializeField] private TMP_Text timeText;
    
    private AudioOutputPlayer player;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        
        playButton.onClick.AddListener(() => player.Resume());
        pauseButton.onClick.AddListener(() => player.Pause());
        stopButton.onClick.AddListener(() => player.Stop());
        
        progressSlider.onValueChanged.AddListener(Seek);
    }
    
    void Update()
    {
        if (player.IsPlaying)
        {
            float progress = player.PlaybackPosition / player.Duration;
            progressSlider.SetValueWithoutNotify(progress);
            
            timeText.text = $"{FormatTime(player.PlaybackPosition)} / {FormatTime(player.Duration)}";
        }
    }
    
    void Seek(float progress)
    {
        player.Seek(progress * player.Duration);
    }
    
    string FormatTime(float seconds)
    {
        int minutes = Mathf.FloorToInt(seconds / 60);
        int secs = Mathf.FloorToInt(seconds % 60);
        return $"{minutes:00}:{secs:00}";
    }
}
```

### Volume Mixer UI

```csharp
public class VolumeMixerUI : MonoBehaviour
{
    [SerializeField] private Slider masterSlider;
    [SerializeField] private Slider voiceSlider;
    [SerializeField] private Slider effectsSlider;
    
    void Start()
    {
        masterSlider.onValueChanged.AddListener(vol => 
            AudioListener.volume = vol);
        
        voiceSlider.onValueChanged.AddListener(vol => 
            agent.AudioController.OutputPlayer.Volume = vol);
    }
}
```

## Audio Visualization

### Waveform Display

```csharp
public class WaveformVisualizer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Image waveformImage;
    [SerializeField] private int resolution = 256;
    
    private AudioOutputPlayer player;
    
    void Start()
    {
        player = agent.AudioController.OutputPlayer;
        player.onPlaybackStarted.AddListener(GenerateWaveform);
    }
    
    void GenerateWaveform(AudioClip clip)
    {
        float[] samples = new float[clip.samples * clip.channels];
        clip.GetData(samples, 0);
        
        Texture2D texture = new Texture2D(resolution, 100);
        
        int samplesPerPixel = samples.Length / resolution;
        
        for (int x = 0; x < resolution; x++)
        {
            float sum = 0;
            for (int i = 0; i < samplesPerPixel; i++)
            {
                sum += Mathf.Abs(samples[x * samplesPerPixel + i]);
            }
            float average = sum / samplesPerPixel;
            
            int height = Mathf.RoundToInt(average * 100);
            
            for (int y = 0; y < 100; y++)
            {
                texture.SetPixel(x, y, y < height ? Color.white : Color.black);
            }
        }
        
        texture.Apply();
        waveformImage.sprite = Sprite.Create(
            texture,
            new Rect(0, 0, resolution, 100),
            new Vector2(0.5f, 0.5f)
        );
    }
}
```

## Error Handling

### Handle Playback Errors

```csharp
player.onPlaybackError.AddListener(error =>
{
    Debug.LogError($"Playback error: {error}");
    
    if (error.Contains("audio_source"))
    {
        ShowMessage("Audio source not configured.");
    }
    else if (error.Contains("invalid_clip"))
    {
        ShowMessage("Invalid audio clip.");
    }
    else
    {
        ShowMessage("Playback failed.");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AudioPlayerManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private AudioSource audioSource;
    
    [Header("UI")]
    [SerializeField] private Button playPauseButton;
    [SerializeField] private Button stopButton;
    [SerializeField] private Slider volumeSlider;
    [SerializeField] private Slider speedSlider;
    [SerializeField] private TMP_Text statusText;
    
    private AudioOutputPlayer player;
    private Queue<AudioClip> playlist = new();
    private bool isPlaying;
    
    async void Start()
    {
        await SetupPlayer();
        
        playPauseButton.onClick.AddListener(TogglePlayPause);
        stopButton.onClick.AddListener(Stop);
        volumeSlider.onValueChanged.AddListener(SetVolume);
        speedSlider.onValueChanged.AddListener(SetSpeed);
    }
    
    async UniTask SetupPlayer()
    {
        player = agent.AudioController.OutputPlayer;
        player.AudioSource = audioSource;
        
        // Configure
        player.Volume = 1.0f;
        player.Speed = 1.0f;
        
        // Listen for events
        player.onPlaybackStarted.AddListener(OnPlaybackStarted);
        player.onPlaybackCompleted.AddListener(OnPlaybackCompleted);
        player.onPlaybackError.AddListener(OnPlaybackError);
        
        // Listen for agent audio responses
        agent.onAudioResponseReceived.AddListener(QueueAudio);
        
        Debug.Log("✓ Audio player ready");
    }
    
    void QueueAudio(AudioClip clip)
    {
        playlist.Enqueue(clip);
        Debug.Log($"➕ Queued: {playlist.Count} clips");
        
        if (!isPlaying)
        {
            PlayNext();
        }
    }
    
    void PlayNext()
    {
        if (playlist.Count == 0)
        {
            isPlaying = false;
            statusText.text = "Ready";
            return;
        }
        
        AudioClip clip = playlist.Dequeue();
        player.Play(clip);
        isPlaying = true;
    }
    
    void TogglePlayPause()
    {
        if (player.IsPlaying)
        {
            player.Pause();
        }
        else
        {
            player.Resume();
        }
    }
    
    void Stop()
    {
        player.Stop();
        playlist.Clear();
        isPlaying = false;
    }
    
    void SetVolume(float volume)
    {
        player.Volume = volume;
    }
    
    void SetSpeed(float speed)
    {
        player.Speed = speed;
    }
    
    void OnPlaybackStarted(AudioClip clip)
    {
        Debug.Log($"▶️ Playing: {clip.name}");
        statusText.text = $"Playing: {clip.name}";
        playPauseButton.GetComponentInChildren<TMP_Text>().text = "Pause";
    }
    
    void OnPlaybackCompleted()
    {
        Debug.Log("✓ Playback completed");
        PlayNext();
    }
    
    void OnPlaybackError(string error)
    {
        Debug.LogError($"Playback error: {error}");
        statusText.text = $"<color=red>Error: {error}</color>";
    }
}
```

## Next Steps

- [Audio Input Recorder](input-recorder.md)
- [Speech Transcription](transcription.md)
- [Text-to-Speech](text-to-speech.md)
