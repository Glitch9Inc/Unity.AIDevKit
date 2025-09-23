---
description: Generate a video using a diffusion transformer model
icon: play
---

# Video

returns [`GeneratedVideo`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedVideo.html)

This task generates a short video clip from a **text prompt or image**, using models such as **Veo** (currently the only supported model).

The result is returned as a `RawFile`, pointing to the generated video file.

{% hint style="info" %}
#### Unity Limitation: `VideoClip` Cannot Be Created at Runtime

> **Unity does not support creation of a `VideoClip` at runtime.**

This means you cannot dynamically generate a `VideoClip` in memory using code (e.g., `ScriptableObject.CreateInstance<VideoClip>()`).\
Instead, videos must be handled as files—either streamed or played using `VideoPlayer.url`.
{% endhint %}

**Generating a Video**

```csharp
// Generate from Text Prompt
RawFile videoFile = await "A cat surfing a wave"
    .GENVideo() 
    .ExecuteAsync();

// Generate from Image
Texture2D myImage = MyImage;
RawFile videoFileFromImage = await myImage
    .GENVideo() 
    .ExecuteAsync();
```

{% hint style="warning" %}
Results are saved to a temporary location unless you call `.SetOutputPath()`.
{% endhint %}
