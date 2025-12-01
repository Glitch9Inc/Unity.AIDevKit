---
icon: repeat
---

# Image to Image

Transform entire images while preserving structure using `.ImageToImage()` or `.GENInpaint()`.

## Basic Usage

```csharp
Texture2D transformed = await sourceTexture
    .ImageToImage("Convert to watercolor painting")
    .ExecuteAsync();
```

## Input Types

### Texture2D + Instruction

```csharp
Texture2D texture = Resources.Load<Texture2D>("Photo");
Texture2D artistic = await texture
    .ImageToImage("Make it look like an oil painting")
    .ExecuteAsync();
```

### Sprite + Instruction

```csharp
Sprite sprite = Resources.Load<Sprite>("Character");
Texture2D pixelated = await sprite
    .ImageToImage("Convert to pixel art style")
    .ExecuteAsync();
```

### ImagePrompt

```csharp
var prompt = new ImagePrompt
{
    Image = sourceTexture,
    Instruction = "Apply anime art style"
};

Texture2D result = await prompt
    .ImageToImage()
    .ExecuteAsync();
```

### File Input

```csharp
var file = new File<Texture2D>(texture, "source.png");
Texture2D result = await file
    .ImageToImage("Make it look hand-drawn")
    .ExecuteAsync();
```

## Common Use Cases

### 1. Style Transfer

```csharp
async UniTask<Texture2D> ApplyStyle(Texture2D source, string style)
{
    return await source
        .ImageToImage($"Convert to {style} style")
        .ExecuteAsync();
}

// Usage
Texture2D watercolor = await ApplyStyle(photo, "watercolor painting");
Texture2D sketch = await ApplyStyle(photo, "pencil sketch");
Texture2D anime = await ApplyStyle(photo, "anime");
```

### 2. Art Direction Changes

```csharp
async UniTask<Texture2D> ChangeArtDirection(Texture2D concept, string direction)
{
    return await concept
        .ImageToImage($"Redesign with {direction}")
        .ExecuteAsync();
}

// Usage
Texture2D realistic = await ChangeArtDirection(cartoon, "realistic style");
Texture2D stylized = await ChangeArtDirection(photo, "stylized low-poly");
```

### 3. Quality Enhancement

```csharp
async UniTask<Texture2D> EnhanceQuality(Texture2D lowRes)
{
    return await lowRes
        .ImageToImage("Enhance to high quality, sharpen details")
        .ExecuteAsync();
}
```

### 4. Color Grading

```csharp
async UniTask<Texture2D> ApplyColorGrade(Texture2D source, string grade)
{
    return await source
        .ImageToImage($"Apply {grade} color grading")
        .ExecuteAsync();
}

// Usage
Texture2D warm = await ApplyColorGrade(image, "warm sunset");
Texture2D cool = await ApplyColorGrade(image, "cool blue");
Texture2D vintage = await ApplyColorGrade(image, "vintage film");
```

### 5. Environment Changes

```csharp
async UniTask<Texture2D> ChangeEnvironment(Texture2D scene, string environment)
{
    return await scene
        .ImageToImage($"Change setting to {environment}")
        .ExecuteAsync();
}

// Usage
Texture2D night = await ChangeEnvironment(dayScene, "nighttime with moonlight");
Texture2D winter = await ChangeEnvironment(summer, "winter with snow");
```

## Unity Integration Examples

### Example 1: Material Style Switcher

```csharp
public class MaterialStyleSwitcher : MonoBehaviour
{
    [SerializeField] private Texture2D baseTexture;
    [SerializeField] private Material targetMaterial;
    
    public async UniTask ApplyStyle(string style)
    {
        Texture2D styled = await baseTexture
            .ImageToImage($"Convert to {style} style")
            .ExecuteAsync();
        
        targetMaterial.mainTexture = styled;
    }
}

// Usage
await switcher.ApplyStyle("cartoon");
await switcher.ApplyStyle("realistic");
await switcher.ApplyStyle("pixel art");
```

### Example 2: Screenshot Filter

```csharp
public class ScreenshotFilter : MonoBehaviour
{
    public async UniTask<Texture2D> CaptureAndFilter(string filter)
    {
        // Capture screenshot
        Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();
        
        // Apply filter
        Texture2D filtered = await screenshot
            .ImageToImage($"Apply {filter} effect")
            .ExecuteAsync();
        
        return filtered;
    }
}

// Usage
Texture2D sepia = await filter.CaptureAndFilter("sepia tone");
Texture2D blackwhite = await filter.CaptureAndFilter("black and white");
```

### Example 3: Texture Variation Generator

```csharp
public class TextureVariationGenerator : MonoBehaviour
{
    public async UniTask<List<Texture2D>> GenerateVariations(
        Texture2D baseTexture, 
        int count)
    {
        var variations = new List<Texture2D>();
        
        for (int i = 0; i < count; i++)
        {
            Texture2D variation = await baseTexture
                .ImageToImage($"Create artistic variation {i+1}")
                .ExecuteAsync();
            
            variations.Add(variation);
        }
        
        return variations;
    }
}
```

### Example 4: Environment Time-of-Day System

```csharp
public class TimeOfDayConverter : MonoBehaviour
{
    [SerializeField] private Texture2D baseSkybox;
    
    public async UniTask SetTimeOfDay(string timeOfDay)
    {
        Texture2D skybox = await baseSkybox
            .ImageToImage($"Convert to {timeOfDay} lighting")
            .ExecuteAsync();
        
        RenderSettings.skybox.SetTexture("_MainTex", skybox);
    }
}

// Usage
await timeOfDay.SetTimeOfDay("sunset");
await timeOfDay.SetTimeOfDay("midnight");
await timeOfDay.SetTimeOfDay("golden hour");
```

### Example 5: Batch Processing

```csharp
async UniTask<List<Texture2D>> BatchConvert(
    List<Texture2D> textures, 
    string transformation)
{
    var tasks = textures.Select(tex =>
        tex.ImageToImage(transformation).ExecuteAsync()
    );
    
    Texture2D[] results = await UniTask.WhenAll(tasks);
    return results.ToList();
}

// Usage
var stylized = await BatchConvert(
    originalTextures,
    "Convert to low-poly style"
);
```

## Instruction Tips

### ✅ Good Instructions

```csharp
// ✅ Specific style references
"Convert to Studio Ghibli anime art style"

// ✅ Clear transformation goal
"Make it look like a hand-drawn watercolor painting"

// ✅ Include technical details
"Apply cel-shading with bold outlines, comic book style"

// ✅ Describe desired mood
"Transform to dark, moody noir aesthetic with high contrast"
```

### ❌ Bad Instructions

```csharp
// ❌ Too vague
"make it better"

// ❌ Conflicting requirements
"realistic but also cartoon-like"

// ❌ Unclear target
"change it"

// ❌ Too complex
"make it anime and add dragons and change the background to space"
```

## Differences from Inpainting

| Feature | Image to Image | Inpainting |
|---------|---------------|------------|
| **Scope** | Full image | Selective areas |
| **Structure** | Preserved | Can change |
| **Use Case** | Style transfer | Object editing |
| **Result** | Global change | Local change |

## When to Use

### ✅ Use Image to Image for

- Style transfers (photo to painting)
- Art direction changes
- Filter effects
- Color grading
- Full scene transformations

### ❌ Use Inpainting for

- Object removal/addition
- Selective edits
- Background replacement
- Specific area fixes

## Provider Support

### OpenAI DALL-E

```csharp
// Works with both DALL-E 2 and DALL-E 3
Texture2D result = await texture
    .ImageToImage("transformation")
    .SetModel(OpenAIModel.DallE2)
    .ExecuteAsync();
```

### Google Imagen

```csharp
Texture2D result = await texture
    .ImageToImage("transformation")
    .SetModel(GoogleModel.Imagen3)
    .ExecuteAsync();
```

## Best Practices

### ✅ Do

```csharp
// ✅ Be specific about target style
await texture.ImageToImage("Convert to pixel art, 8-bit style");

// ✅ Include quality hints
await texture.ImageToImage("High-quality oil painting style");

// ✅ Preserve important elements
await texture.ImageToImage("Watercolor style, preserve faces");

// ✅ Test with small images first
Texture2D small = ScaleDown(original, 512, 512);
await small.ImageToImage("test transformation");
```

### ❌ Don't

```csharp
// ❌ Too vague
await texture.ImageToImage("different");

// ❌ Multiple transformations
await texture.ImageToImage("cartoon and realistic and painted");

// ❌ Unrealistic expectations
await texture.ImageToImage("make it look exactly like a Van Gogh");
```

## Configuration

```csharp
Texture2D result = await texture
    .ImageToImage("transformation")
    .SetModel(OpenAIModel.DallE3)
    .SetQuality(ImageQuality.HD)
    .ExecuteAsync();
```

## Error Handling

```csharp
try
{
    Texture2D result = await texture
        .ImageToImage("transformation")
        .ExecuteAsync();
    
    if (result == null || result.width == 0)
        throw new Exception("Invalid result");
    
    // Use result
}
catch (AIApiException ex)
{
    Debug.LogError($"Transformation failed: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Performance Tips

```csharp
// ✅ Good - parallel processing
var tasks = new[]
{
    tex1.ImageToImage("style A").ExecuteAsync(),
    tex2.ImageToImage("style B").ExecuteAsync()
};
await UniTask.WhenAll(tasks);

// ✅ Good - reuse configuration
var transformer = "placeholder"
    .GENImage()
    .SetModel(OpenAIModel.DallE3);

Texture2D t1 = await texture1.ImageToImage("watercolor").ExecuteAsync();
Texture2D t2 = await texture2.ImageToImage("anime").ExecuteAsync();

// ❌ Bad - sequential when parallel is better
foreach (var tex in textures)
{
    await tex.ImageToImage("style").ExecuteAsync();
}
```

## Next Steps

- [Text to Image](text-to-image.md) - Generate new images
- [Image Inpainting](image-inpainting.md) - Selective editing
