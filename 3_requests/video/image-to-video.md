---
icon: film
---

# Image to Video

Animate static images into videos using `.GENVideo()`.

## Basic Usage

```csharp
Texture2D image = Resources.Load<Texture2D>("Portrait");
VideoClip video = await image
    .GENVideo()
    .ExecuteAsync();
```

## Input Types

### Texture2D Input

```csharp
Texture2D stillImage = screenshot;
VideoClip animated = await stillImage
    .GENVideo()
    .ExecuteAsync();
```

### ImagePrompt Input

```csharp
var prompt = new ImagePrompt
{
    Image = texture,
    Instruction = "Add gentle wind movement"
};

VideoClip video = await prompt
    .GENVideo()
    .ExecuteAsync();
```

## Configuration

```csharp
VideoClip video = await texture
    .GENVideo()
    .SetDuration(5f)
    .SetMotion("subtle camera pan")
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Animate Screenshot

```csharp
public class ScreenshotAnimator : MonoBehaviour
{
    public async UniTask<VideoClip> AnimateScreenshot()
    {
        Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();
        
        return await screenshot
            .GENVideo()
            .SetDuration(3f)
            .ExecuteAsync();
    }
}
```

### Example 2: Photo Booth Effect

```csharp
public class PhotoBooth : MonoBehaviour
{
    public async UniTask<VideoClip> CreateBoomerang(Texture2D photo)
    {
        return await photo
            .GENVideo()
            .SetMotion("subtle zoom in and out")
            .SetDuration(2f)
            .ExecuteAsync();
    }
}
```

## Use Cases

| Use Case | Example |
|----------|---------|
| **Parallax** | Add depth to 2D images |
| **Cinemagraphs** | Subtle motion in photos |
| **Transitions** | Smooth scene changes |
| **Effects** | Camera moves on stills |

## Next Steps

- [Text to Video](text-to-video.md) - Generate from text
