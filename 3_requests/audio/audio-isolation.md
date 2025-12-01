---
icon: filter
---

# Audio Isolation

Isolate or enhance audio elements using `.GENAudioIsolation()`.

## Basic Usage

```csharp
AudioClip noisyAudio = Resources.Load<AudioClip>("NoisyRecording");
AudioClip clean = await noisyAudio
    .GENAudioIsolation()
    .ExecuteAsync();
```

## Input Types

### AudioClip Input

```csharp
AudioClip noisy = Resources.Load<AudioClip>("Audio");
AudioClip isolated = await noisy
    .GENAudioIsolation()
    .ExecuteAsync();
```

### File Input

```csharp
var audioFile = new File<AudioClip>(audioClip, "recording.wav");
AudioClip isolated = await audioFile
    .GENAudioIsolation()
    .ExecuteAsync();
```

## Common Use Cases

### 1. Voice Isolation

Remove background noise and isolate speech:

```csharp
async UniTask<AudioClip> IsolateVoice(AudioClip noisyRecording)
{
    return await noisyRecording
        .GENAudioIsolation()
        .ExecuteAsync();
}
```

### 2. Noise Reduction

Clean up audio recordings:

```csharp
async UniTask<AudioClip> ReduceNoise(AudioClip audio)
{
    return await audio
        .GENAudioIsolation()
        .ExecuteAsync();
}
```

### 3. Audio Enhancement

Improve audio clarity:

```csharp
async UniTask<AudioClip> EnhanceAudio(AudioClip lowQuality)
{
    return await lowQuality
        .GENAudioIsolation()
        .ExecuteAsync();
}
```

## Unity Integration Examples

### Example 1: Voice Chat Cleaner

```csharp
public class VoiceChatCleaner : MonoBehaviour
{
    public async UniTask<AudioClip> CleanVoiceChat(AudioClip rawRecording)
    {
        // Remove background noise from voice chat
        AudioClip clean = await rawRecording
            .GENAudioIsolation()
            .ExecuteAsync();
        
        return clean;
    }
}
```

### Example 2: Recording Cleanup

```csharp
public class RecordingCleanup : MonoBehaviour
{
    public async UniTask ProcessRecording(AudioClip recording)
    {
        Debug.Log("Cleaning up recording...");
        
        AudioClip cleaned = await recording
            .GENAudioIsolation()
            .ExecuteAsync();
        
        // Save cleaned version
        SaveAudioClip(cleaned, "cleaned_recording.wav");
    }
    
    void SaveAudioClip(AudioClip clip, string filename)
    {
        // Implementation
    }
}
```

### Example 3: Batch Audio Processor

```csharp
public class BatchAudioProcessor : MonoBehaviour
{
    public async UniTask<List<AudioClip>> ProcessBatch(List<AudioClip> audioClips)
    {
        var tasks = audioClips.Select(clip =>
            clip.GENAudioIsolation().ExecuteAsync()
        );
        
        AudioClip[] results = await UniTask.WhenAll(tasks);
        return results.ToList();
    }
}
```

### Example 4: Real-time Audio Filter

```csharp
public class RealtimeAudioFilter : MonoBehaviour
{
    private Queue<AudioClip> processingQueue = new();
    
    public void OnMicrophoneInput(AudioClip recording)
    {
        ProcessAudioAsync(recording).Forget();
    }
    
    async UniTaskVoid ProcessAudioAsync(AudioClip recording)
    {
        AudioClip cleaned = await recording
            .GENAudioIsolation()
            .ExecuteAsync();
        
        // Play cleaned audio
        AudioSource.PlayClipAtPoint(cleaned, transform.position);
    }
}
```

### Example 5: Podcast Editor

```csharp
public class PodcastEditor : MonoBehaviour
{
    public async UniTask<AudioClip> PrepareForPublish(AudioClip rawPodcast)
    {
        // Clean up podcast audio
        AudioClip isolated = await rawPodcast
            .GENAudioIsolation()
            .ExecuteAsync();
        
        Debug.Log("Podcast audio cleaned and ready");
        return isolated;
    }
}
```

### Example 6: Voice Command Preprocessor

```csharp
public class VoiceCommandPreprocessor : MonoBehaviour
{
    public async UniTask<string> ProcessVoiceCommand(AudioClip rawCommand)
    {
        // Clean audio first
        AudioClip clean = await rawCommand
            .GENAudioIsolation()
            .ExecuteAsync();
        
        // Then transcribe
        string command = await clean
            .GENTranscript()
            .ExecuteAsync();
        
        return command;
    }
}
```

## Provider Support

### ElevenLabs

```csharp
AudioClip isolated = await audioClip
    .GENAudioIsolation()
    .ExecuteAsync();
```

**Features:**

- ✅ Voice isolation
- ✅ Noise reduction
- ✅ Audio enhancement
- ✅ Background removal

**Note:** Currently, ElevenLabs is the primary provider for audio isolation.

## What Gets Removed

Audio isolation typically removes:

- Background noise
- Environmental sounds
- Music (in voice isolation mode)
- Echo and reverb
- Static and hiss
- Wind noise

## What Gets Preserved

- Primary speech/voice
- Speech clarity
- Timing and rhythm
- Emotional tone
- Word pronunciation

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Process before transcription
AudioClip clean = await noisy.GENAudioIsolation().ExecuteAsync();
string text = await clean.GENTranscript().ExecuteAsync();

// ✅ Cache processed audio
Dictionary<string, AudioClip> cache = new();

// ✅ Check audio quality first
if (IsHighQuality(audio))
    return audio;  // Skip processing
else
    return await audio.GENAudioIsolation().ExecuteAsync();

// ✅ Clean up after processing
Destroy(noisyAudio);
```

### ❌ Bad Practices

```csharp
// ❌ Don't process already clean audio
// Unnecessary API calls and cost

// ❌ Don't process in Update()
void Update()
{
    await audio.GENAudioIsolation().ExecuteAsync();  // NO!
}

// ❌ Don't forget cleanup
// Memory leak if clips aren't destroyed

// ❌ Don't block main thread
AudioClip clip = audio.GENAudioIsolation().ExecuteAsync().Result;  // Blocks!
```

## Audio Requirements

**Input:**

- Any audio with speech
- Background noise is okay
- Any format (WAV, MP3, etc.)

**Quality:**

- Better input = better output
- Extreme noise may reduce quality
- Very short clips may not process well

**Limits:**

- Max file size: varies by provider
- Duration: varies by provider

## Use Cases

| Use Case | Example |
|----------|---------|
| **Voice Chat** | Clean real-time voice communication |
| **Podcasts** | Remove background noise |
| **Interviews** | Isolate speaker from environment |
| **Voice Commands** | Improve recognition accuracy |
| **Recordings** | Clean up low-quality recordings |
| **Transcription** | Pre-process for better accuracy |

## Error Handling

```csharp
try
{
    AudioClip cleaned = await noisyAudio
        .GENAudioIsolation()
        .ExecuteAsync();
    
    if (cleaned == null || cleaned.length == 0)
        throw new Exception("Audio isolation failed");
    
    // Use cleaned audio
    audioSource.clip = cleaned;
    audioSource.Play();
}
catch (AIApiException ex)
{
    Debug.LogError($"Isolation failed: {ex.Message}");
    // Fallback to original audio
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Performance Tips

```csharp
// ✅ Good - process once, use many times
AudioClip clean = await noisy.GENAudioIsolation().ExecuteAsync();
voiceCache["clean"] = clean;

// ✅ Good - parallel processing
var tasks = audioClips.Select(clip =>
    clip.GENAudioIsolation().ExecuteAsync()
);
await UniTask.WhenAll(tasks);

// ❌ Bad - reprocess every time
// Cache the result instead
```

## Workflow: Clean → Transcribe

Common pattern for voice recognition:

```csharp
async UniTask<string> TranscribeNoisy(AudioClip noisyAudio)
{
    // Step 1: Clean audio
    AudioClip clean = await noisyAudio
        .GENAudioIsolation()
        .ExecuteAsync();
    
    // Step 2: Transcribe
    string text = await clean
        .GENTranscript()
        .ExecuteAsync();
    
    return text;
}
```

## Limitations

1. **Speech Focus**: Optimized for speech, not music
2. **Cannot Restore**: Can't restore missing/corrupted audio
3. **Quality Dependent**: Very noisy input = lower quality output
4. **Processing Time**: Takes time depending on audio length

## Next Steps

- [Voice Change](voice-change.md) - Modify voice characteristics
- [Speech to Text](speech-to-text.md) - Transcribe audio
- [Text to Speech](text-to-speech.md) - Generate speech
- [Sound Effects](sound-effects.md) - Generate sound effects
