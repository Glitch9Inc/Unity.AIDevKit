---
icon: microphone
---

# Speech to Text

Transcribe audio to text using `.GENTranscription()` or `.SpeechToText()`.

## Basic Usage

```csharp
AudioClip recording = Microphone.Start(null, false, 10, 44100);
await UniTask.Delay(5000);
Microphone.End(null);

string transcript = await recording
    .GENTranscription()
    .ExecuteAsync();

Debug.Log($"You said: {transcript}");
```

## Input Types

### AudioClip Input

```csharp
AudioClip audio = Resources.Load<AudioClip>("Recording");
string text = await audio
    .GENTranscription()
    .ExecuteAsync();
```

### File Input

```csharp
var audioFile = new File<AudioClip>(audioClip, "recording.wav");
string text = await audioFile
    .GENTranscription()
    .ExecuteAsync();
```

### Alias Method

```csharp
// SpeechToText() is an alias for GENTranscription()
string text = await audioClip
    .SpeechToText()
    .ExecuteAsync();
```

## Configuration

### Model Selection

```csharp
// OpenAI Whisper
string text = await audioClip
    .GENTranscription()
    .SetModel(OpenAIModel.Whisper1)
    .ExecuteAsync();

// Google Chirp
string text = await audioClip
    .GENTranscription()
    .SetModel(GoogleModel.ChirpV2)
    .ExecuteAsync();
```

### Language Hint

```csharp
string text = await audioClip
    .GENTranscription()
    .SetSpokenLanguage(SystemLanguage.English)
    .ExecuteAsync();
```

**Supported languages:**

- English, Spanish, French, German, Italian, Portuguese
- Chinese, Japanese, Korean
- Arabic, Russian, Turkish
- And many more...

### Timestamp Granularities

Populate timestamps in the transcription output:

```csharp
string text = await audioClip
    .GENTranscription()
    .SetTimestampGranularities("word", "segment")
    .ExecuteAsync();
```

**Available granularities:**

- `"word"` - Word-level timestamps (adds latency)
- `"segment"` - Segment-level timestamps (no added latency)

## Unity Integration Examples

### Example 1: Voice Command System

```csharp
public class VoiceCommandSystem : MonoBehaviour
{
    private bool isListening = false;
    
    public async UniTask StartListening()
    {
        if (isListening) return;
        isListening = true;
        
        // Start recording
        AudioClip recording = Microphone.Start(null, false, 10, 44100);
        
        // Wait for user to finish speaking
        await UniTask.Delay(5000);
        
        // Stop recording
        Microphone.End(null);
        isListening = false;
        
        // Transcribe
        string command = await recording
            .GENTranscription()
            .SetSpokenLanguage(Application.systemLanguage)
            .ExecuteAsync();
        
        // Process command
        ProcessCommand(command);
    }
    
    void ProcessCommand(string command)
    {
        if (command.Contains("jump"))
            player.Jump();
        else if (command.Contains("shoot"))
            player.Shoot();
    }
}
```

### Example 2: Real-time Subtitles

```csharp
public class RealtimeSubtitles : MonoBehaviour
{
    [SerializeField] private TMPro.TextMeshProUGUI subtitleText;
    private Queue<AudioClip> audioQueue = new();
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
            StartRecording().Forget();
    }
    
    async UniTaskVoid StartRecording()
    {
        AudioClip recording = Microphone.Start(null, false, 5, 44100);
        await UniTask.Delay(3000);
        Microphone.End(null);
        
        string text = await recording
            .GENTranscription()
            .ExecuteAsync();
        
        subtitleText.text = text;
    }
}
```

### Example 3: Dictation System

```csharp
public class DictationSystem : MonoBehaviour
{
    [SerializeField] private TMPro.TMP_InputField inputField;
    private List<string> transcripts = new();
    
    public async UniTask RecordDictation()
    {
        Debug.Log("Recording... Press Space to stop");
        
        AudioClip recording = Microphone.Start(null, false, 60, 44100);
        
        await UniTask.WaitUntil(() => Input.GetKeyDown(KeyCode.Space));
        
        Microphone.End(null);
        
        string text = await recording
            .GENTranscription()
            .SetSpokenLanguage(Application.systemLanguage)
            .ExecuteAsync();
        
        transcripts.Add(text);
        inputField.text = string.Join(" ", transcripts);
    }
}
```

### Example 4: Audio File Transcriber

```csharp
public class AudioFileTranscriber : MonoBehaviour
{
    public async UniTask<string> TranscribeAudioFile(string filePath)
    {
        // Load audio file
        AudioClip audio = await LoadAudioFile(filePath);
        
        // Transcribe
        string transcript = await audio
            .GENTranscription()
            .SetModel(OpenAIModel.Whisper1)
            .SetResponseFormat(TranscriptFormat.VerboseJson)
            .ExecuteAsync();
        
        // Save to text file
        string outputPath = Path.ChangeExtension(filePath, ".txt");
        File.WriteAllText(outputPath, transcript);
        
        return transcript;
    }
    
    async UniTask<AudioClip> LoadAudioFile(string path)
    {
        // Implementation depends on audio format
        // Use UnityWebRequest for runtime loading
        return null;
    }
}
```

### Example 5: Meeting Transcriber

```csharp
public class MeetingTranscriber : MonoBehaviour
{
    private List<string> transcripts = new();
    private bool isRecording = false;
    
    public async UniTask StartMeeting()
    {
        isRecording = true;
        
        while (isRecording)
        {
            // Record 30-second segments
            AudioClip segment = Microphone.Start(null, false, 30, 44100);
            await UniTask.Delay(30000);
            Microphone.End(null);
            
            // Transcribe segment
            string text = await segment
                .GENTranscription()
                .ExecuteAsync();
            
            transcripts.Add($"[{DateTime.Now:HH:mm:ss}] {text}");
        }
    }
    
    public void StopMeeting()
    {
        isRecording = false;
        
        // Save full transcript
        string fullTranscript = string.Join("\n\n", transcripts);
        File.WriteAllText("meeting_transcript.txt", fullTranscript);
    }
}
```

### Example 6: Multi-Language Support

```csharp
public class MultiLanguageTranscriber : MonoBehaviour
{
    public async UniTask<string> TranscribeWithLanguageDetection(AudioClip audio)
    {
        // Try transcribing without language hint (auto-detect)
        string transcript = await audio
            .GENTranscription()
            .ExecuteAsync();
        
        return transcript;
    }
    
    public async UniTask<string> TranscribeSpecificLanguage(
        AudioClip audio, 
        SystemLanguage language)
    {
        return await audio
            .GENTranscription()
            .SetSpokenLanguage(language)
            .ExecuteAsync();
    }
}
```

## Provider Support

### OpenAI Whisper

```csharp
string text = await audioClip
    .GENTranscription()
    .SetModel(OpenAIModel.Whisper1)
    .SetSpokenLanguage(SystemLanguage.English)
    .SetTimestampGranularities("word", "segment")
    .ExecuteAsync();
```

**Features:**

- ??99+ languages
- ??High accuracy
- ??Speaker diarization (in verbose mode)
- ??Timestamp support

### Google Chirp

```csharp
string text = await audioClip
    .GENTranscription()
    .SetModel(GoogleModel.ChirpV2)
    .SetSpokenLanguage(SystemLanguage.English)
    .ExecuteAsync();
```

**Features:**

- ??Multiple languages
- ??Real-time streaming
- ??Punctuation
- ??Word-level timestamps

## Audio Requirements

### Format Requirements

**Supported formats:**

- WAV
- MP3
- M4A
- FLAC
- OGG

**Recommended settings:**

- Sample rate: 16kHz or higher
- Channels: Mono or Stereo
- Bit depth: 16-bit or higher

### Size Limits

**OpenAI:**

- Max file size: 25 MB
- Max duration: ~2 hours (at standard quality)

**Google:**

- Max file size: 10 MB (for sync)
- Max duration: 1 minute (for sync)

### Best Practices for Audio

```csharp
// ??Good - Use appropriate sample rate
AudioClip recording = Microphone.Start(null, false, 10, 16000);

// ??Good - Remove silence at start/end
AudioClip trimmed = TrimSilence(recording);

// ??Good - Normalize volume
AudioClip normalized = NormalizeVolume(recording);

// ??Bad - Very low sample rate
AudioClip lowQuality = Microphone.Start(null, false, 10, 8000);
```

## Best Practices

### ??Good Practices

```csharp
// ✅ Provide a language hint to improve accuracy
await audio.GENTranscription()
    .SetSpokenLanguage(SystemLanguage.English)
    .ExecuteAsync();

// ✅ Use appropriate language hint
await audio.GENTranscription()
    .SetSpokenLanguage(Application.systemLanguage)
    .ExecuteAsync();

// ??Handle long audio by chunking
async UniTask<string> TranscribeLongAudio(AudioClip longAudio)
{
    var chunks = SplitAudio(longAudio, maxSeconds: 30);
    var tasks = chunks.Select(chunk => chunk.GENTranscription().ExecuteAsync());
    string[] results = await UniTask.WhenAll(tasks);
    return string.Join(" ", results);
}

// ??Cache common phrases
Dictionary<AudioClip, string> cache = new();
```

### ??Bad Practices

```csharp
// ??Don't transcribe empty/silent audio
if (audioClip.length == 0) return;  // Check first!

// ??Don't use very low quality audio
// Use at least 16kHz sample rate

// ??Don't forget error handling
try {
    string text = await audio.GENTranscription().ExecuteAsync();
} catch (Exception ex) {
    Debug.LogError($"Transcription failed: {ex.Message}");
}

// ??Don't block main thread
string text = audio.GENTranscription().ExecuteAsync().Result;  // NO!
```

## Error Handling

```csharp
try
{
    string transcript = await audioClip
        .GENTranscription()
        .ExecuteAsync();
    
    if (string.IsNullOrEmpty(transcript))
        Debug.LogWarning("Empty transcript - audio may be silent");
    
    return transcript;
}
catch (AIApiException ex)
{
    Debug.LogError($"API Error: {ex.Message}");
    // Handle: invalid audio format, file too large, etc.
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Performance Tips

```csharp
// ??Good - parallel processing
var tasks = audioClips.Select(clip => 
    clip.GENTranscription().ExecuteAsync()
);
await UniTask.WhenAll(tasks);

// ??Good - use lower quality for non-critical transcription
await audio.GENTranscription()
    .SetModel(OpenAIModel.Whisper1)  // Faster
    .ExecuteAsync();

// ??Bad - sequential processing
foreach (var clip in audioClips)
{
    await clip.GENTranscription().ExecuteAsync();  // Slow!
}
```

## Next Steps

- [Speech Translation](speech-translation.md) - Translate speech to English
- [Text to Speech](text-to-speech.md) - Generate speech from text
- [Voice Change](voice-change.md) - Modify voice characteristics
