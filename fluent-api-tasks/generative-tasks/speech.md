---
description: Generate a speech using a text-to-speech model
icon: message-music
---

# Speech

returns [`GeneratedAudio`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedAudio.html)

Convert written text into natural-sounding speech using AI voices.

Perfect for dialogue, narration, accessibility, or dynamic voice-over content — powered by **ElevenLabs** and other TTS providers.

**Basic Usage**

```csharp
AudioClip voice = await "Welcome to the future of AI voices."
    .GENSpeech()
    .SetModel(ElevenLabsModel.Eleven_Flash_V2) // optional if you have set the default tts model in the settings
    .SetVoice(ElevenLabsVoice.Rachel) // optional if you have set the default voice in the settings
    .SetSpeed(1.0f) // optional
    .ExecuteAsync();
```

**Streaming**

Play audio in real-time as it’s generated — ideal for chat avatars or quick feedback loops. It requires `StreamAudioPlayer` from the Glitch9 core library.

```csharp
await "Loading complete."
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Adam)
    .StreamAsync(myStreamAudioPlayer);

// StreamAudioPlayer is a component that plays incoming audio chunks as they arrive.
```

