---
icon: clapperboard
---

# Text to Video

Generate videos from text descriptions using `.GENVideo()`.

## Basic Usage

```csharp
VideoClip video = await "A serene lake with mountains in the background"
    .GENVideo()
    .ExecuteAsync();
```

## Input Types

### String Input

```csharp
VideoClip video = await "Fireworks over city skyline"
    .GENVideo()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("A {subject} {action} in {location}");
VideoClip video = await prompt
    .GENVideo()
    .ExecuteAsync();
```

## Configuration

```csharp
VideoClip video = await "Wildlife documentary scene"
    .GENVideo()
    .SetDuration(10f)
    .SetResolution(1920, 1080)
    .SetFrameRate(30)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Cutscene Generator

```csharp
public class CutsceneGenerator : MonoBehaviour
{
    public async UniTask<VideoClip> GenerateCutscene(string description)
    {
        return await description
            .GENVideo()
            .SetDuration(15f)
            .ExecuteAsync();
    }
}
```

### Example 2: Background Video

```csharp
public class BackgroundVideoGenerator : MonoBehaviour
{
    public async UniTask SetBackground(string environment)
    {
        VideoClip bg = await $"Looping {environment} background"
            .GENVideo()
            .SetDuration(30f)
            .ExecuteAsync();
        
        // Apply to video player
        videoPlayer.clip = bg;
        videoPlayer.isLooping = true;
        videoPlayer.Play();
    }
}
```

## Prompt Tips

### ✅ Good Prompts

```csharp
// ✅ Specific and descriptive
"Smooth camera pan across medieval castle at golden hour"

// ✅ Include motion details
"Slow zoom into glowing crystal, particles floating around"

// ✅ Describe timing
"Quick cut between city scenes, fast-paced montage"
```

### ❌ Bad Prompts

```csharp
// ❌ Too vague
"video"

// ❌ Too complex
"entire movie plot with multiple scenes and characters"
```

## Limitations

1. **Duration**: Typically limited to 10-30 seconds
2. **Quality**: May not match professional video
3. **Consistency**: Character/object consistency across frames
4. **Cost**: Expensive compared to image generation

## Next Steps

- [Image to Video](image-to-video.md) - Animate images
