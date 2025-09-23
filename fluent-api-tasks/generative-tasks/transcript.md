---
description: Transcribe a recording using a speech to text model
icon: microphone-stand
---

# Transcript

returns [`Transcript`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.Transcript.html)

Convert spoken words from an `AudioClip` into text using powerful AI transcription models. Ideal for voice commands, user feedback, subtitles, or audio-driven gameplay systems.

**Basic Usage**

```csharp
AudioClip recording = MicrophoneCapture.GetLastClip();

string result = await recording
    .GENTranscript()
    .SetModel(OpenAIModel.Whisper)
    .SetLanguage(SystemLanguage.Korean)
    .ExecuteAsync();

Debug.Log("Transcript: " + result);
```

***

## GENTranslation

returns [`Transcript`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.Transcript.html)

You can also translate speech into English using `GENTranslation.`

```csharp
string english = await recording
    .GENTranslation()
    .SetModel(OpenAIModel.Whisper)
    .ExecuteAsync();
```
