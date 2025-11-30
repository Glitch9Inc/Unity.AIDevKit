# Speech Transcription

Convert speech to text using Whisper API.

## Overview

Speech Transcription enables:

- Speech-to-text conversion
- Multi-language support
- Timestamp generation
- Speaker detection
- Real-time transcription

## Basic Setup

### Transcribe Audio

```csharp
string text = await agent.TranscribeAsync(audioClip);
Debug.Log($"Transcribed: {text}");
```

## Transcription Options

### Configure Settings

```csharp
public class TranscriptionSettings : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        var transcriber = agent.AudioController.Transcriber;
        
        // Configure
        transcriber.Language = "en";
        transcriber.Model = "whisper-1";
        transcriber.Temperature = 0f;
        transcriber.EnableTimestamps = false;
        
        Debug.Log("✓ Transcription configured");
    }
}
```

### Language Support

```csharp
public async UniTask TranscribeWithLanguage(AudioClip clip, string language)
{
    var transcriber = agent.AudioController.Transcriber;
    transcriber.Language = language;
    
    string text = await agent.TranscribeAsync(clip);
    Debug.Log($"Transcribed ({language}): {text}");
}

// Examples
await TranscribeWithLanguage(clip, "en"); // English
await TranscribeWithLanguage(clip, "es"); // Spanish
await TranscribeWithLanguage(clip, "ja"); // Japanese
await TranscribeWithLanguage(clip, "ko"); // Korean
await TranscribeWithLanguage(clip, "zh"); // Chinese
```

## Timestamped Transcription

### Get Timestamps

```csharp
public class TimestampedTranscription : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async UniTask<TranscriptionResult> TranscribeWithTimestamps(AudioClip clip)
    {
        var transcriber = agent.AudioController.Transcriber;
        transcriber.EnableTimestamps = true;
        
        var result = await transcriber.TranscribeAsync(clip);
        
        Debug.Log($"Transcription: {result.Text}");
        
        foreach (var segment in result.Segments)
        {
            Debug.Log($"[{segment.Start:F2}s - {segment.End:F2}s]: {segment.Text}");
        }
        
        return result;
    }
}

public class TranscriptionResult
{
    public string Text { get; set; }
    public List<TranscriptionSegment> Segments { get; set; }
}

public class TranscriptionSegment
{
    public float Start { get; set; }
    public float End { get; set; }
    public string Text { get; set; }
}
```

### Subtitle Generation

```csharp
public class SubtitleGenerator : MonoBehaviour
{
    [SerializeField] private TMP_Text subtitleText;
    [SerializeField] private float displayDuration = 3f;
    
    public async UniTask ShowSubtitles(TranscriptionResult result, AudioClip clip)
    {
        var player = agent.AudioController.OutputPlayer;
        player.Play(clip);
        
        foreach (var segment in result.Segments)
        {
            // Wait until segment start time
            await UniTask.WaitUntil(() => 
                player.PlaybackPosition >= segment.Start);
            
            // Show subtitle
            subtitleText.text = segment.Text;
            
            // Hide after duration
            float duration = segment.End - segment.Start;
            await UniTask.Delay(TimeSpan.FromSeconds(duration));
            
            subtitleText.text = "";
        }
    }
}
```

## Real-Time Transcription

### Streaming Transcription

```csharp
public class StreamingTranscription : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private float chunkDuration = 2f;
    
    private AudioInputRecorder recorder;
    private List<string> transcriptionBuffer = new();
    private bool isStreaming;
    
    void Start()
    {
        recorder = agent.AudioController.InputRecorder;
    }
    
    public void StartStreaming()
    {
        isStreaming = true;
        transcriptionBuffer.Clear();
        
        StartCoroutine(StreamingLoop());
        Debug.Log("🎤 Streaming transcription started");
    }
    
    public void StopStreaming()
    {
        isStreaming = false;
        
        string fullTranscription = string.Join(" ", transcriptionBuffer);
        Debug.Log($"Full transcription: {fullTranscription}");
    }
    
    IEnumerator StreamingLoop()
    {
        while (isStreaming)
        {
            // Record chunk
            recorder.StartRecording();
            yield return new WaitForSeconds(chunkDuration);
            AudioClip chunk = recorder.StopRecording();
            
            // Transcribe chunk
            TranscribeChunk(chunk).Forget();
        }
    }
    
    async UniTaskVoid TranscribeChunk(AudioClip chunk)
    {
        string text = await agent.TranscribeAsync(chunk);
        
        if (!string.IsNullOrEmpty(text))
        {
            transcriptionBuffer.Add(text);
            Debug.Log($"Chunk: {text}");
            
            OnChunkTranscribed?.Invoke(text);
        }
    }
    
    public event Action<string> OnChunkTranscribed;
}
```

### Live Transcription Display

```csharp
public class LiveTranscriptionUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text transcriptionText;
    [SerializeField] private ScrollRect scrollView;
    
    private StreamingTranscription streaming;
    
    void Start()
    {
        streaming = GetComponent<StreamingTranscription>();
        streaming.OnChunkTranscribed += AppendTranscription;
    }
    
    void AppendTranscription(string text)
    {
        transcriptionText.text += text + " ";
        
        // Auto-scroll to bottom
        Canvas.ForceUpdateCanvases();
        scrollView.verticalNormalizedPosition = 0;
    }
}
```

## Voice Commands

### Command Recognition

```csharp
public class VoiceCommandRecognizer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Commands")]
    [SerializeField] private string[] commands = new[]
    {
        "help",
        "inventory",
        "map",
        "status",
        "settings"
    };
    
    public async UniTask<string> RecognizeCommand(AudioClip clip)
    {
        string text = await agent.TranscribeAsync(clip);
        text = text.ToLower().Trim();
        
        Debug.Log($"Recognized: {text}");
        
        // Find matching command
        foreach (string cmd in commands)
        {
            if (text.Contains(cmd))
            {
                Debug.Log($"✓ Command matched: {cmd}");
                return cmd;
            }
        }
        
        Debug.Log("No command matched");
        return null;
    }
}
```

### Keyword Spotting

```csharp
public class KeywordSpotter : MonoBehaviour
{
    [SerializeField] private Dictionary<string, Action> keywords = new()
    {
        { "attack", () => Debug.Log("Attack command") },
        { "defend", () => Debug.Log("Defend command") },
        { "heal", () => Debug.Log("Heal command") },
        { "run", () => Debug.Log("Run command") }
    };
    
    public async UniTask ProcessVoiceInput(AudioClip clip)
    {
        string text = await agent.TranscribeAsync(clip);
        text = text.ToLower();
        
        foreach (var keyword in keywords)
        {
            if (text.Contains(keyword.Key))
            {
                Debug.Log($"Keyword detected: {keyword.Key}");
                keyword.Value?.Invoke();
            }
        }
    }
}
```

## Translation

### Transcribe and Translate

```csharp
public async UniTask<string> TranscribeAndTranslate(
    AudioClip clip, 
    string sourceLanguage, 
    string targetLanguage)
{
    var transcriber = agent.AudioController.Transcriber;
    transcriber.Language = sourceLanguage;
    
    // Transcribe in source language
    string sourceText = await agent.TranscribeAsync(clip);
    Debug.Log($"Source ({sourceLanguage}): {sourceText}");
    
    // Translate to target language
    string translatedText = await TranslateText(sourceText, targetLanguage);
    Debug.Log($"Translated ({targetLanguage}): {translatedText}");
    
    return translatedText;
}

async UniTask<string> TranslateText(string text, string targetLanguage)
{
    string prompt = $"Translate to {targetLanguage}: {text}";
    
    var conversation = agent.ConversationController.CreateConversation();
    await conversation.AddMessageAsync(prompt);
    
    var response = await agent.SendAsync();
    return response.TextContent;
}
```

## Accuracy Optimization

### Prompt Optimization

```csharp
public class TranscriptionOptimizer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string context;
    
    public async UniTask<string> TranscribeWithContext(AudioClip clip)
    {
        var transcriber = agent.AudioController.Transcriber;
        
        // Provide context for better accuracy
        transcriber.Prompt = context;
        
        string text = await agent.TranscribeAsync(clip);
        return text;
    }
}

// Example usage
var optimizer = new TranscriptionOptimizer();
optimizer.context = "This is a conversation about game development, " +
                   "including terms like Unity, GameObject, Component, etc.";

string text = await optimizer.TranscribeWithContext(audioClip);
```

### Temperature Control

```csharp
// Lower temperature for more deterministic output
transcriber.Temperature = 0f;

// Higher temperature for more creative transcription
transcriber.Temperature = 0.5f;
```

## Audio Pre-processing

### Improve Audio Quality

```csharp
public AudioClip PreprocessAudio(AudioClip input)
{
    float[] samples = new float[input.samples * input.channels];
    input.GetData(samples, 0);
    
    // Noise reduction
    for (int i = 0; i < samples.Length; i++)
    {
        if (Mathf.Abs(samples[i]) < 0.01f)
        {
            samples[i] = 0;
        }
    }
    
    // Normalization
    float max = 0;
    foreach (float sample in samples)
    {
        max = Mathf.Max(max, Mathf.Abs(sample));
    }
    
    if (max > 0)
    {
        for (int i = 0; i < samples.Length; i++)
        {
            samples[i] /= max;
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
```

## Batch Transcription

### Process Multiple Files

```csharp
public class BatchTranscriber : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async UniTask<List<string>> TranscribeBatch(List<AudioClip> clips)
    {
        List<string> transcriptions = new();
        
        for (int i = 0; i < clips.Count; i++)
        {
            Debug.Log($"Transcribing {i + 1}/{clips.Count}...");
            
            string text = await agent.TranscribeAsync(clips[i]);
            transcriptions.Add(text);
            
            // Rate limiting
            await UniTask.Delay(TimeSpan.FromSeconds(1));
        }
        
        Debug.Log($"✓ Batch transcription complete: {clips.Count} files");
        return transcriptions;
    }
}
```

## Export Transcriptions

### Save to File

```csharp
public class TranscriptionExporter : MonoBehaviour
{
    [SerializeField] private string exportPath = "Transcriptions";
    
    public void SaveTranscription(string text, string filename)
    {
        string fullPath = Path.Combine(
            Application.persistentDataPath,
            exportPath,
            $"{filename}.txt"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(fullPath));
        File.WriteAllText(fullPath, text);
        
        Debug.Log($"💾 Saved: {fullPath}");
    }
    
    public void SaveSRT(TranscriptionResult result, string filename)
    {
        StringBuilder srt = new();
        
        for (int i = 0; i < result.Segments.Count; i++)
        {
            var segment = result.Segments[i];
            
            srt.AppendLine($"{i + 1}");
            srt.AppendLine($"{FormatSRTTime(segment.Start)} --> {FormatSRTTime(segment.End)}");
            srt.AppendLine(segment.Text);
            srt.AppendLine();
        }
        
        string fullPath = Path.Combine(
            Application.persistentDataPath,
            exportPath,
            $"{filename}.srt"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(fullPath));
        File.WriteAllText(fullPath, srt.ToString());
        
        Debug.Log($"💾 Saved SRT: {fullPath}");
    }
    
    string FormatSRTTime(float seconds)
    {
        TimeSpan time = TimeSpan.FromSeconds(seconds);
        return $"{time.Hours:00}:{time.Minutes:00}:{time.Seconds:00},{time.Milliseconds:000}";
    }
}
```

## Error Handling

### Handle Transcription Errors

```csharp
try
{
    string text = await agent.TranscribeAsync(audioClip);
    Debug.Log($"Transcribed: {text}");
}
catch (Exception ex)
{
    Debug.LogError($"Transcription error: {ex.Message}");
    
    if (ex.Message.Contains("invalid_audio"))
    {
        ShowMessage("Invalid audio format.");
    }
    else if (ex.Message.Contains("too_long"))
    {
        ShowMessage("Audio too long (max 25MB).");
    }
    else if (ex.Message.Contains("language_not_supported"))
    {
        ShowMessage("Language not supported.");
    }
    else
    {
        ShowMessage("Transcription failed.");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class TranscriptionManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button recordButton;
    [SerializeField] private TMP_Text transcriptionText;
    
    [Header("Settings")]
    [SerializeField] private string language = "en";
    [SerializeField] private bool enableTimestamps = false;
    [SerializeField] private float temperature = 0f;
    
    private AudioInputRecorder recorder;
    private SpeechTranscriber transcriber;
    private bool isRecording;
    
    async void Start()
    {
        await SetupTranscription();
        
        recordButton.onClick.AddListener(ToggleRecording);
    }
    
    async UniTask SetupTranscription()
    {
        recorder = agent.AudioController.InputRecorder;
        transcriber = agent.AudioController.Transcriber;
        
        // Configure
        transcriber.Language = language;
        transcriber.EnableTimestamps = enableTimestamps;
        transcriber.Temperature = temperature;
        
        Debug.Log("✓ Transcription ready");
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
        
        transcriptionText.text = "🎤 Recording...";
        recordButton.GetComponentInChildren<TMP_Text>().text = "Stop";
        
        Debug.Log("🎤 Recording started");
    }
    
    async void StopRecording()
    {
        AudioClip clip = recorder.StopRecording();
        isRecording = false;
        
        transcriptionText.text = "⏳ Processing...";
        recordButton.interactable = false;
        
        Debug.Log("⏹️ Recording stopped, transcribing...");
        
        try
        {
            // Preprocess audio
            AudioClip processed = PreprocessAudio(clip);
            
            // Transcribe
            string text = await agent.TranscribeAsync(processed);
            
            if (!string.IsNullOrEmpty(text))
            {
                transcriptionText.text = text;
                Debug.Log($"✓ Transcribed: {text}");
                
                // Send to agent
                await agent.SendAsync(text);
            }
            else
            {
                transcriptionText.text = "No speech detected";
            }
        }
        catch (Exception ex)
        {
            Debug.LogError($"Transcription error: {ex.Message}");
            transcriptionText.text = $"<color=red>Error: {ex.Message}</color>";
        }
        finally
        {
            recordButton.GetComponentInChildren<TMP_Text>().text = "Record";
            recordButton.interactable = true;
        }
    }
    
    AudioClip PreprocessAudio(AudioClip input)
    {
        float[] samples = new float[input.samples * input.channels];
        input.GetData(samples, 0);
        
        // Noise reduction
        for (int i = 0; i < samples.Length; i++)
        {
            if (Mathf.Abs(samples[i]) < 0.01f)
            {
                samples[i] = 0;
            }
        }
        
        // Normalization
        float max = 0;
        foreach (float sample in samples)
        {
            max = Mathf.Max(max, Mathf.Abs(sample));
        }
        
        if (max > 0)
        {
            for (int i = 0; i < samples.Length; i++)
            {
                samples[i] /= max;
            }
        }
        
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

## Next Steps

- [Audio Input Recorder](input-recorder.md)
- [Audio Output Player](output-player.md)
- [Text-to-Speech](text-to-speech.md)
