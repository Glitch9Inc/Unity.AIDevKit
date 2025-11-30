---
description: Generate a sound effect using a generative audio model
icon: waveform
---

# SoundFX

returns [`GeneratedAudio`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.GeneratedAudio.html)

**GENSoundEffect** allows you to generate **short, high-quality sound effects** from text prompts using AI.\
It's especially useful for game developers, UI feedback sounds, ambient design, and more — powered by  **ElevenLabs SFX** models.

**Basic Usage**

```csharp
AudioClip sfx = await "Footsteps on snow"
    .GENSoundEffect()
    .SetDuration(2.5)
    .ExecuteAsync();
    
myAudioSource.Play(); // play immediately after the sfx is generation
```
