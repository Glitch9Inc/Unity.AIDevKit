---
icon: image
---

# Image Generation

AI Dev Kit provides multiple methods for generating and editing images using AI.

## Available Methods

### 1. Text to Image (`.GENImage()`)

Generate images from text descriptions:

```csharp
Texture2D image = await "A cyberpunk city at night"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024)
    .ExecuteAsync();
```

**Best for:**

- ✅ Creating new images from scratch
- ✅ Concept art
- ✅ Asset generation
- ✅ Prototyping

### 2. Image Inpainting (`.GENInpaint()`)

Edit specific parts of an existing image:

```csharp
Texture2D edited = await sourceTexture
    .GENInpaint("Add a red car in the center")
    .ExecuteAsync();
```

**Best for:**

- ✅ Selective editing
- ✅ Object removal/addition
- ✅ Background replacement
- ✅ Style transfer

### 3. Image to Image (`.ImageToImage()`)

Transform entire image while preserving structure:

```csharp
Texture2D watercolor = await sourceTexture
    .ImageToImage("Convert to watercolor painting")
    .ExecuteAsync();
```

**Best for:**

- ✅ Style conversion
- ✅ Art direction changes
- ✅ Filters and effects
- ✅ Variations

## Quick Comparison

| Method | Input | Use Case | Providers |
|--------|-------|----------|-----------|
| `GENImage()` | Text | New images | OpenAI, Google |
| `GENInpaint()` | Image + Text | Selective edits | OpenAI |
| `ImageToImage()` | Image + Text | Full transforms | OpenAI |

## Basic Examples

### Example 1: Generate Logo

```csharp
Texture2D logo = await "Modern tech company logo, minimalist"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024)
    .SetQuality(ImageQuality.HD)
    .ExecuteAsync();

// Use in UI
logoImage.texture = logo;
```

### Example 2: Edit Screenshot

```csharp
Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();

Texture2D edited = await screenshot
    .GENInpaint("Remove UI elements")
    .ExecuteAsync();
```

### Example 3: Style Transfer

```csharp
Texture2D concept = await Resources.Load<Texture2D>("ConceptArt");

Texture2D pixelArt = await concept
    .ImageToImage("Convert to pixel art style")
    .ExecuteAsync();
```

## Configuration Options

All image generation methods support:

```csharp
Texture2D image = await "Your prompt"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)         // Model selection
    .SetSize(ImageSize._1024x1024)        // Image dimensions
    .SetQuality(ImageQuality.HD)          // Quality level
    .SetStyle(ImageStyle.Vivid)           // Art style
    .SetN(1)                              // Number of images
    .ExecuteAsync();
```

## Provider Support

### OpenAI DALL-E

```csharp
// DALL-E 3 (highest quality)
Texture2D image = await "prompt"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetQuality(ImageQuality.HD)
    .ExecuteAsync();

// DALL-E 2 (faster, cheaper)
Texture2D image = await "prompt"
    .GENImage()
    .SetModel(OpenAIModel.DallE2)
    .ExecuteAsync();
```

### Google Imagen

```csharp
Texture2D image = await "prompt"
    .GENImage()
    .SetModel(GoogleModel.Imagen3)
    .ExecuteAsync();
```

## Common Workflows

### Workflow 1: Texture Generation

```csharp
async UniTask<Texture2D> GenerateTexture(string description)
{
    Texture2D texture = await description
        .GENImage()
        .SetModel(OpenAIModel.DallE3)
        .SetSize(ImageSize._1024x1024)
        .SetQuality(ImageQuality.Standard)
        .ExecuteAsync();
    
    // Apply to material
    GetComponent<Renderer>().material.mainTexture = texture;
    
    return texture;
}
```

### Workflow 2: Sprite Creation

```csharp
async UniTask<Sprite> GenerateSprite(string description)
{
    Texture2D texture = await description
        .GENImage()
        .SetSize(ImageSize._512x512)
        .ExecuteAsync();
    
    // Convert to sprite
    Sprite sprite = Sprite.Create(
        texture,
        new Rect(0, 0, texture.width, texture.height),
        new Vector2(0.5f, 0.5f)
    );
    
    return sprite;
}
```

### Workflow 3: Batch Generation

```csharp
async UniTask<List<Texture2D>> GenerateBatch(List<string> prompts)
{
    var tasks = prompts.Select(prompt =>
        prompt.GENImage().ExecuteAsync()
    );
    
    Texture2D[] results = await UniTask.WhenAll(tasks);
    return results.ToList();
}
```

## Performance Tips

### ✅ Good Practices

```csharp
// ✅ Reuse request configuration
var imageRequest = "placeholder"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024);

Texture2D logo = await imageRequest.SetPrompt("logo").ExecuteAsync();
Texture2D icon = await imageRequest.SetPrompt("icon").ExecuteAsync();

// ✅ Parallel generation
var tasks = new[]
{
    "image 1".GENImage().ExecuteAsync(),
    "image 2".GENImage().ExecuteAsync()
};
await UniTask.WhenAll(tasks);

// ✅ Cache results
Dictionary<string, Texture2D> imageCache = new();
```

### ❌ Bad Practices

```csharp
// ❌ Sequential when parallel is possible
for (int i = 0; i < 10; i++)
{
    await $"image {i}".GENImage().ExecuteAsync();
}

// ❌ Unnecessarily high quality
await "test".GENImage()
    .SetQuality(ImageQuality.HD)  // Expensive!
    .SetSize(ImageSize._1792x1024)
    .ExecuteAsync();
```

## Next Steps

- [Text to Image](text-to-image.md) - Detailed generation guide
- [Image Inpainting](image-inpainting.md) - Editing techniques
- [Image to Image](image-to-image.md) - Transformation guide
