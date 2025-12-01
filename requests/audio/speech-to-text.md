# Speech to Text

## Overview

The Speech to Text feature enables the conversion of spoken language into written text. This is useful for transcribing audio recordings, enabling voice commands, and more.

## Getting Started

To use the Speech to Text feature, you need to have an audio clip in a supported format (e.g., WAV, MP3). You can then use the `GENTranscript` method to transcribe the audio to text.

### Basic Usage

```csharp
AudioClip audio = Resources.Load<AudioClip>("YourAudioClip");
string transcribedText = await audio.GENTranscript().ExecuteAsync();
```

## Next Steps

- [Speech Translation](speech-translation.md) - Translate speech to English
- [Text to Speech](text-to-speech.md) - Generate speech from text
- [Voice Change](voice-change.md) - Modify voice characteristics

---

## Speech Translation

In addition to transcribing audio in its original language, you can also translate speech directly to English using `.GENTranslation()`.

### Quick Translation

```csharp
AudioClip foreignAudio = Resources.Load<AudioClip>("Spanish");
string english = await foreignAudio
    .GENTranslation()
    .ExecuteAsync();
```

### When to Use Translation vs Transcription

| Use Case                | Method                | Output            |
|-------------------------|-----------------------|-------------------|
| Same language text      | `.GENTranscript()`    | Original language |
| Convert to English      | `.GENTranslation()`   | Always English    |

### Translation Example

```csharp
public class MultiLanguageSupport : MonoBehaviour
{
    public async UniTask<string> GetEnglishText(AudioClip anyLanguage)
    {
        // Automatically translates to English
        return await anyLanguage
            .GENTranslation()
            .ExecuteAsync();
    }
}
```

For detailed translation features and examples, see [Speech Translation](speech-translation.md).
