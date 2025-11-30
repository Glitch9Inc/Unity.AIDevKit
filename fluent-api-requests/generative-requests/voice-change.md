---
description: Morph your voice using a  voice conversion model
icon: microphone
---

# Voice Change

returns [`GeneratedAudio`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.GeneratedAudio.html)

Transform recorded voices using AI-powered voice conversion.

This can be used to change the speaker’s identity, create fictional character voices, or anonymize speech. Powered by **ElevenLabs**.

Basic Usage

```csharp
AudioClip originalVoice = Resources.Load<AudioClip>("UserVoice");

AudioClip newVoice = await originalVoice
    .GENVoiceChange()
    .SetVoice(ElevenLabsVoice.Rachel)
    .SetSeed(42)
    .ExecuteAsync();
```
