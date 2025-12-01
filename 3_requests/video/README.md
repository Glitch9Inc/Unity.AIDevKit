---
icon: video
---

# Video Generation

AI Dev Kit provides video generation capabilities from text or images using `.GENVideo()`.

## Available Methods

### 1. Text to Video

Generate videos from text descriptions:

```csharp
VideoClip video = await "Ocean waves at sunset"
    .GENVideo()
    .ExecuteAsync();
```

### 2. Image to Video

Animate existing images:

```csharp
Texture2D image = Resources.Load<Texture2D>("Frame");
VideoClip video = await image
    .GENVideo()
    .ExecuteAsync();
```

## Basic Examples

### Example 1: Text to Video

```csharp
VideoClip video = await "A cat playing with a ball"
    .GENVideo()
    .SetDuration(5f)
    .ExecuteAsync();
```

### Example 2: Image Animation

```csharp
Texture2D stillImage = screenshot;
VideoClip animated = await stillImage
    .GENVideo()
    .ExecuteAsync();
```

## Configuration

### Duration

```csharp
VideoClip video = await "Dancing character"
    .GENVideo()
    .SetDuration(10f)  // 10 seconds
    .ExecuteAsync();
```

### Resolution

```csharp
VideoClip video = await "Landscape"
    .GENVideo()
    .SetResolution(1920, 1080)
    .ExecuteAsync();
```

### Frame Rate

```csharp
VideoClip video = await "Action scene"
    .GENVideo()
    .SetFrameRate(30)
    .ExecuteAsync();
```

## Provider Support

Video generation is available through select providers:

- **OpenAI**: Limited support
- **Google**: Experimental features
- **Specialized providers**: Coming soon

**Note:** Video generation is an emerging feature. Check provider documentation for current capabilities.

## Next Steps

- [Text to Video](text-to-video.md) - Detailed text-to-video guide
- [Image to Video](image-to-video.md) - Image animation guide
