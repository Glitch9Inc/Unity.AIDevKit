# Audio Input Recorder

Record audio input from microphone for voice interactions.

## Overview

Audio Input Recorder enables:

- Microphone recording in Unity
- Push-to-talk functionality
- Voice activation detection (VAD)
- Audio processing and filtering
- Recording management

## Basic Setup

### Initialize Recorder

```csharp
var audioRecorder = agent.AudioController.InputRecorder;

// Configure settings
audioRecorder.SampleRate = 44100;
audioRecorder.MaxRecordingSeconds = 60;
audioRecorder.EnableNoiseReduction = true;
```

## Recording Controls

### Start/Stop Recording

```csharp
public class VoiceRecorder : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private AudioInputRecorder recorder;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
    }
    
    public void StartRecording()
    {
        recorder.StartRecording();
        Debug.Log("🎤 Recording started");
    }
    
    public void StopRecording()
    {
        AudioClip clip = recorder.StopRecording();
        Debug.Log($"⏹️ Recording stopped: {clip.length}s");
        
        // Use the recorded clip
        ProcessRecording(clip);
    }
    
    void ProcessRecording(AudioClip clip)
    {
        // Send to transcription or analysis
    }
}
```

### Push-to-Talk

```csharp
public class PushToTalk : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private KeyCode recordKey = KeyCode.Space;
    
    private AudioInputRecorder recorder;
    private bool isRecording;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
    }
    
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
        recorder.StartRecording();
        isRecording = true;
        
        Debug.Log("🎤 Recording...");
        ShowRecordingUI(true);
    }
    
    async void StopRecording()
    {
        if (!isRecording) return;
        
        AudioClip clip = recorder.StopRecording();
        isRecording = false;
        
        Debug.Log("⏹️ Processing...");
        ShowRecordingUI(false);
        
        // Transcribe
        string text = await agent.TranscribeAsync(clip);
        
        if (!string.IsNullOrEmpty(text))
        {
            Debug.Log($"You said: {text}");
            await agent.SendAsync(text);
        }
    }
    
    void ShowRecordingUI(bool show)
    {
        // Update UI to show recording state
    }
}
```

## Voice Activation Detection

### VAD Implementation

```csharp
public class VoiceActivationDetector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("VAD Settings")]
    [SerializeField] private float noiseThreshold = 0.02f;
    [SerializeField] private float silenceTimeout = 2.0f;
    [SerializeField] private float minRecordingLength = 0.5f;
    
    private AudioInputRecorder recorder;
    private bool isRecording;
    private float silenceTimer;
    private float recordingTimer;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
        recorder.EnableVAD = true;
        recorder.VADThreshold = noiseThreshold;
    }
    
    void Update()
    {
        float micLevel = GetMicrophoneLevel();
        
        if (!isRecording)
        {
            // Detect voice activity
            if (micLevel > noiseThreshold)
            {
                StartAutoRecording();
            }
        }
        else
        {
            recordingTimer += Time.deltaTime;
            
            // Detect silence
            if (micLevel < noiseThreshold)
            {
                silenceTimer += Time.deltaTime;
                
                if (silenceTimer >= silenceTimeout && 
                    recordingTimer >= minRecordingLength)
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
    
    float GetMicrophoneLevel()
    {
        return recorder.GetCurrentLevel();
    }
    
    void StartAutoRecording()
    {
        recorder.StartRecording();
        isRecording = true;
        silenceTimer = 0;
        recordingTimer = 0;
        
        Debug.Log("🎤 Voice detected, recording...");
    }
    
    async void StopAutoRecording()
    {
        AudioClip clip = recorder.StopRecording();
        isRecording = false;
        
        Debug.Log("⏹️ Silence detected, processing...");
        
        string text = await agent.TranscribeAsync(clip);
        
        if (!string.IsNullOrEmpty(text))
        {
            Debug.Log($"Transcribed: {text}");
            await agent.SendAsync(text);
        }
    }
}
```

## Audio Processing

### Noise Reduction

```csharp
public class NoiseReduction : MonoBehaviour
{
    [SerializeField] private bool enableNoiseReduction = true;
    [SerializeField] private float reductionAmount = 0.5f;
    
    public AudioClip ProcessAudio(AudioClip input)
    {
        if (!enableNoiseReduction)
            return input;
        
        float[] samples = new float[input.samples * input.channels];
        input.GetData(samples, 0);
        
        // Apply noise reduction
        for (int i = 0; i < samples.Length; i++)
        {
            // Simple noise gate
            if (Mathf.Abs(samples[i]) < reductionAmount)
            {
                samples[i] = 0;
            }
        }
        
        // Create processed clip
        AudioClip processed = AudioClip.Create(
            "Processed",
            input.samples,
            input.channels,
            input.frequency,
            false
        );
        processed.SetData(samples, 0);
        
        return processed;
    }
}
```

### Audio Normalization

```csharp
public AudioClip NormalizeAudio(AudioClip input)
{
    float[] samples = new float[input.samples * input.channels];
    input.GetData(samples, 0);
    
    // Find max amplitude
    float maxAmplitude = 0;
    foreach (float sample in samples)
    {
        maxAmplitude = Mathf.Max(maxAmplitude, Mathf.Abs(sample));
    }
    
    // Normalize
    if (maxAmplitude > 0)
    {
        float normalizeMultiplier = 1.0f / maxAmplitude;
        for (int i = 0; i < samples.Length; i++)
        {
            samples[i] *= normalizeMultiplier;
        }
    }
    
    // Create normalized clip
    AudioClip normalized = AudioClip.Create(
        "Normalized",
        input.samples,
        input.channels,
        input.frequency,
        false
    );
    normalized.SetData(samples, 0);
    
    return normalized;
}
```

## Recording Management

### Save Recordings

```csharp
public class RecordingManager : MonoBehaviour
{
    [SerializeField] private string savePath = "Recordings";
    
    public void SaveRecording(AudioClip clip, string filename)
    {
        byte[] wavData = ConvertToWav(clip);
        
        string fullPath = Path.Combine(
            Application.persistentDataPath,
            savePath,
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
        
        // WAV header
        byte[] wav = new byte[44 + samples.Length * 2];
        
        // RIFF header
        System.Text.Encoding.UTF8.GetBytes("RIFF").CopyTo(wav, 0);
        BitConverter.GetBytes(wav.Length - 8).CopyTo(wav, 4);
        System.Text.Encoding.UTF8.GetBytes("WAVE").CopyTo(wav, 8);
        
        // fmt chunk
        System.Text.Encoding.UTF8.GetBytes("fmt ").CopyTo(wav, 12);
        BitConverter.GetBytes(16).CopyTo(wav, 16); // Chunk size
        BitConverter.GetBytes((short)1).CopyTo(wav, 20); // Audio format (PCM)
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

### Load Recordings

```csharp
public AudioClip LoadRecording(string filename)
{
    string fullPath = Path.Combine(
        Application.persistentDataPath,
        savePath,
        $"{filename}.wav"
    );
    
    if (!File.Exists(fullPath))
    {
        Debug.LogError($"Recording not found: {fullPath}");
        return null;
    }
    
    byte[] wavData = File.ReadAllBytes(fullPath);
    
    // Parse WAV file
    int channels = BitConverter.ToInt16(wavData, 22);
    int frequency = BitConverter.ToInt32(wavData, 24);
    int dataSize = BitConverter.ToInt32(wavData, 40);
    
    float[] samples = new float[dataSize / 2];
    
    for (int i = 0; i < samples.Length; i++)
    {
        short sample = BitConverter.ToInt16(wavData, 44 + i * 2);
        samples[i] = sample / (float)short.MaxValue;
    }
    
    AudioClip clip = AudioClip.Create(
        filename,
        samples.Length / channels,
        channels,
        frequency,
        false
    );
    clip.SetData(samples, 0);
    
    return clip;
}
```

## UI Integration

### Recording Button

```csharp
public class RecordButton : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button recordButton;
    [SerializeField] private Image buttonImage;
    [SerializeField] private TMP_Text buttonText;
    
    private AudioInputRecorder recorder;
    private bool isRecording;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
        recordButton.onClick.AddListener(ToggleRecording);
    }
    
    void ToggleRecording()
    {
        if (!isRecording)
        {
            StartRecording();
        }
        else
        {
            StopRecording();
        }
    }
    
    void StartRecording()
    {
        recorder.StartRecording();
        isRecording = true;
        
        buttonImage.color = Color.red;
        buttonText.text = "Stop";
        
        StartCoroutine(AnimateRecording());
    }
    
    async void StopRecording()
    {
        AudioClip clip = recorder.StopRecording();
        isRecording = false;
        
        buttonImage.color = Color.white;
        buttonText.text = "Record";
        
        StopAllCoroutines();
        
        // Process recording
        buttonText.text = "Processing...";
        recordButton.interactable = false;
        
        string text = await agent.TranscribeAsync(clip);
        
        if (!string.IsNullOrEmpty(text))
        {
            await agent.SendAsync(text);
        }
        
        buttonText.text = "Record";
        recordButton.interactable = true;
    }
    
    IEnumerator AnimateRecording()
    {
        while (isRecording)
        {
            buttonImage.color = Color.Lerp(Color.red, Color.white, 
                Mathf.PingPong(Time.time * 2, 1));
            yield return null;
        }
    }
}
```

### Audio Visualizer

```csharp
public class AudioVisualizer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Image[] bars;
    [SerializeField] private int sampleCount = 64;
    
    private AudioInputRecorder recorder;
    private float[] samples;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
        samples = new float[sampleCount];
    }
    
    void Update()
    {
        if (!recorder.IsRecording) return;
        
        recorder.GetSpectrumData(samples, 0, FFTWindow.BlackmanHarris);
        
        for (int i = 0; i < bars.Length && i < samples.Length; i++)
        {
            float height = samples[i] * 10f;
            bars[i].fillAmount = Mathf.Lerp(bars[i].fillAmount, height, Time.deltaTime * 10);
        }
    }
}
```

## Error Handling

### Handle Recording Errors

```csharp
recorder.onRecordingError.AddListener(error =>
{
    Debug.LogError($"Recording error: {error}");
    
    if (error.Contains("permission"))
    {
        ShowMessage("Microphone permission denied. Please enable in settings.");
    }
    else if (error.Contains("not_found"))
    {
        ShowMessage("No microphone detected.");
    }
    else if (error.Contains("already_recording"))
    {
        ShowMessage("Already recording.");
    }
    else
    {
        ShowMessage("Recording failed. Please try again.");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AudioRecorderManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button recordButton;
    [SerializeField] private TMP_Text statusText;
    
    [Header("Settings")]
    [SerializeField] private int sampleRate = 44100;
    [SerializeField] private int maxRecordingSeconds = 60;
    [SerializeField] private bool enableNoiseReduction = true;
    
    private AudioInputRecorder recorder;
    private bool isRecording;
    
    async void Start()
    {
        await SetupRecorder();
        
        recordButton.onClick.AddListener(ToggleRecording);
    }
    
    async UniTask SetupRecorder()
    {
        recorder = agent.AudioController.InputRecorder;
        
        // Configure
        recorder.SampleRate = sampleRate;
        recorder.MaxRecordingSeconds = maxRecordingSeconds;
        recorder.EnableNoiseReduction = enableNoiseReduction;
        
        // Listen for events
        recorder.onRecordingStarted.AddListener(OnRecordingStarted);
        recorder.onRecordingStopped.AddListener(OnRecordingStopped);
        recorder.onRecordingError.AddListener(OnRecordingError);
        
        // Request microphone permission
        await RequestMicrophonePermission();
        
        Debug.Log("✓ Audio recorder ready");
    }
    
    async UniTask RequestMicrophonePermission()
    {
        if (!Application.HasUserAuthorization(UserAuthorization.Microphone))
        {
            await Application.RequestUserAuthorization(UserAuthorization.Microphone);
        }
    }
    
    void ToggleRecording()
    {
        if (!isRecording)
        {
            StartRecording();
        }
        else
        {
            StopRecording();
        }
    }
    
    void StartRecording()
    {
        recorder.StartRecording();
    }
    
    async void StopRecording()
    {
        AudioClip clip = recorder.StopRecording();
        
        if (clip != null)
        {
            statusText.text = "Processing...";
            recordButton.interactable = false;
            
            // Transcribe
            string text = await agent.TranscribeAsync(clip);
            
            if (!string.IsNullOrEmpty(text))
            {
                Debug.Log($"Transcribed: {text}");
                await agent.SendAsync(text);
            }
            
            statusText.text = "Ready";
            recordButton.interactable = true;
        }
    }
    
    void OnRecordingStarted()
    {
        isRecording = true;
        statusText.text = "🎤 Recording...";
        recordButton.GetComponentInChildren<TMP_Text>().text = "Stop";
        
        Debug.Log("🎤 Recording started");
    }
    
    void OnRecordingStopped(AudioClip clip)
    {
        isRecording = false;
        statusText.text = "Ready";
        recordButton.GetComponentInChildren<TMP_Text>().text = "Record";
        
        Debug.Log($"⏹️ Recording stopped: {clip.length}s");
    }
    
    void OnRecordingError(string error)
    {
        Debug.LogError($"Recording error: {error}");
        statusText.text = $"<color=red>Error: {error}</color>";
    }
}
```

## Next Steps

- [Audio Output Player](output-player.md)
- [Speech Transcription](transcription.md)
- [Text-to-Speech](text-to-speech.md)
