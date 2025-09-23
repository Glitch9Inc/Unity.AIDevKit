---
description: Isolate vocals from a song using an audio separation model
icon: magnifying-glass-waveform
---

# Audio Isolation

returns [`GeneratedAudio`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.GeneratedAudio.html)

Use AI to extract clean vocals or speech from noisy audio.

This is useful for podcast cleanup, speech recognition preprocessing, or removing background noise in recorded clips. Powered by **ElevenLabs Audio Isolation API**.

**Basic Usage**

```csharp
AudioClip rawRecording = Resources.Load<AudioClip>("NoisyInput");

AudioClip cleanSpeech = await rawRecording
    .GENAudioIsolation()
    .SetOutputFormat(OutputFormat.MPEG)
    .ExecuteAsync();
```
