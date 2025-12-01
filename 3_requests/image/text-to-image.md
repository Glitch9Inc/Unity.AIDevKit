---
icon: wand-magic-sparkles
---

# Text to Image

Generate images from text descriptions using `.GENImage()` or `.TextToImage()`.

## Basic Usage

```csharp
Texture2D image = await "A serene mountain landscape at sunset"
    .GENImage()
    .ExecuteAsync();
```

## Input Types

### String Input

```csharp
Texture2D image = await "Cyberpunk city at night"
    .GENImage()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("A {adjective} {subject} in {style} style");
Texture2D image = await prompt
    .GENImage()
    .ExecuteAsync();
```

### Alias Method

```csharp
// TextToImage() is an alias for GENImage()
Texture2D image = await "Space station"
    .TextToImage()
    .ExecuteAsync();
```

## Configuration

### Model Selection

```csharp
// DALL-E 3 (highest quality)
Texture2D image = await "High quality portrait"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .ExecuteAsync();

// DALL-E 2 (faster, cheaper)
Texture2D image = await "Simple icon"
    .GENImage()
    .SetModel(OpenAIModel.DallE2)
    .ExecuteAsync();
```

### Image Size

```csharp
// Square formats
Texture2D square = await "Logo design"
    .GENImage()
    .SetSize(ImageSize._1024x1024)
    .ExecuteAsync();

// Landscape
Texture2D landscape = await "Wide banner"
    .GENImage()
    .SetSize(ImageSize._1792x1024)
    .ExecuteAsync();

// Portrait
Texture2D portrait = await "Vertical poster"
    .GENImage()
    .SetSize(ImageSize._1024x1792)
    .ExecuteAsync();
```

**Available sizes:**

- `ImageSize._256x256` (DALL-E 2 only)
- `ImageSize._512x512` (DALL-E 2 only)
- `ImageSize._1024x1024` (All models)
- `ImageSize._1792x1024` (DALL-E 3 only)
- `ImageSize._1024x1792` (DALL-E 3 only)

### Quality

```csharp
// HD quality (DALL-E 3 only)
Texture2D hd = await "Detailed artwork"
    .GENImage()
    .SetQuality(ImageQuality.HD)
    .ExecuteAsync();

// Standard quality (faster, cheaper)
Texture2D standard = await "Concept sketch"
    .GENImage()
    .SetQuality(ImageQuality.Standard)
    .ExecuteAsync();
```

### Style

```csharp
// Vivid - hyper-real and dramatic
Texture2D vivid = await "Fantasy dragon"
    .GENImage()
    .SetStyle(ImageStyle.Vivid)
    .ExecuteAsync();

// Natural - more natural, less hyper-real
Texture2D natural = await "Portrait photo"
    .GENImage()
    .SetStyle(ImageStyle.Natural)
    .ExecuteAsync();
```

### Number of Images

```csharp
// Generate multiple variations
List<Texture2D> images = await "Character concept"
    .GENImage()
    .SetN(4)
    .ExecuteAsync();

foreach (var img in images)
{
    Debug.Log($"Generated: {img.width}x{img.height}");
}
```

## Complete Example

```csharp
Texture2D image = await "A majestic phoenix rising from flames"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024)
    .SetQuality(ImageQuality.HD)
    .SetStyle(ImageStyle.Vivid)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: UI Sprite Generation

```csharp
public class SpriteGenerator : MonoBehaviour
{
    [SerializeField] private Image uiImage;
    
    async void Start()
    {
        Texture2D texture = await "Fantasy game icon"
            .GENImage()
            .SetSize(ImageSize._512x512)
            .ExecuteAsync();
        
        Sprite sprite = Sprite.Create(
            texture,
            new Rect(0, 0, texture.width, texture.height),
            new Vector2(0.5f, 0.5f)
        );
        
        uiImage.sprite = sprite;
    }
}
```

### Example 2: Material Texture

```csharp
public class TextureGenerator : MonoBehaviour
{
    async void Start()
    {
        Texture2D texture = await "Brick wall texture, seamless"
            .GENImage()
            .SetSize(ImageSize._1024x1024)
            .ExecuteAsync();
        
        GetComponent<Renderer>().material.mainTexture = texture;
    }
}
```

### Example 3: Skybox Generation

```csharp
async UniTask<Texture2D> GenerateSkybox(string description)
{
    return await description
        .GENImage()
        .SetModel(OpenAIModel.DallE3)
        .SetSize(ImageSize._1792x1024)
        .SetQuality(ImageQuality.HD)
        .ExecuteAsync();
}

// Usage
Texture2D skybox = await GenerateSkybox("Sunset over ocean");
RenderSettings.skybox.SetTexture("_MainTex", skybox);
```

### Example 4: Batch Generation

```csharp
async UniTask<List<Texture2D>> GenerateCharacterConcepts(string basePrompt)
{
    var prompts = new[]
    {
        $"{basePrompt}, warrior class",
        $"{basePrompt}, mage class",
        $"{basePrompt}, rogue class",
        $"{basePrompt}, healer class"
    };
    
    var tasks = prompts.Select(p => 
        p.GENImage()
         .SetModel(OpenAIModel.DallE3)
         .ExecuteAsync()
    );
    
    Texture2D[] results = await UniTask.WhenAll(tasks);
    return results.ToList();
}
```

### Example 5: Save to Disk

```csharp
async UniTask GenerateAndSave(string prompt, string filename)
{
    Texture2D texture = await prompt
        .GENImage()
        .SetModel(OpenAIModel.DallE3)
        .ExecuteAsync();
    
    byte[] bytes = texture.EncodeToPNG();
    string path = Path.Combine(Application.dataPath, filename);
    File.WriteAllBytes(path, bytes);
    
    Debug.Log($"Saved to: {path}");
}
```

## Prompt Engineering Tips

### ✅ Good Prompts

```csharp
// ✅ Specific and detailed
"A mystical forest with glowing mushrooms, ancient trees, 
 ethereal mist, fantasy art style, detailed rendering"

// ✅ Include art style
"Low poly 3D render of a spaceship, vibrant colors, 
 clean geometry, game asset style"

// ✅ Specify mood and lighting
"Dark fantasy castle on a cliff, stormy weather, 
 dramatic lighting, ominous atmosphere"

// ✅ Technical details
"Seamless brick texture, 4K resolution, PBR ready, 
 high detail, realistic"
```

### ❌ Bad Prompts

```csharp
// ❌ Too vague
"castle"

// ❌ Conflicting instructions
"realistic cartoon photo"

// ❌ Too many unrelated elements
"castle, dragon, knight, wizard, princess, treasure, 
 sunset, ocean, mountains, all in one scene"
```

## Provider Support

### OpenAI DALL-E

```csharp
// DALL-E 3
Texture2D image = await "prompt"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetQuality(ImageQuality.HD)
    .SetSize(ImageSize._1024x1024)
    .ExecuteAsync();

// DALL-E 2
Texture2D image = await "prompt"
    .GENImage()
    .SetModel(OpenAIModel.DallE2)
    .SetSize(ImageSize._512x512)
    .ExecuteAsync();
```

### Google Imagen

```csharp
Texture2D image = await "prompt"
    .GENImage()
    .SetModel(GoogleModel.Imagen3)
    .ExecuteAsync();
```

## Error Handling

```csharp
try
{
    Texture2D image = await "Complex scene"
        .GENImage()
        .ExecuteAsync();
    
    if (image == null)
        throw new Exception("Generated image is null");
    
    // Use image
}
catch (AIApiException ex)
{
    Debug.LogError($"API Error: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected Error: {ex.Message}");
}
```

## Best Practices

### ✅ Do

- Be specific with prompts
- Specify art style and mood
- Use appropriate size for your use case
- Consider quality vs cost tradeoff
- Cache generated images
- Handle errors gracefully

### ❌ Don't

- Use vague prompts
- Generate unnecessarily large images
- Forget to dispose unused textures
- Ignore memory management
- Generate images in tight loops

## Performance Considerations

```csharp
// ✅ Good - reuse configuration
var imageGen = "placeholder"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024);

Texture2D img1 = await imageGen.SetPrompt("Image 1").ExecuteAsync();
Texture2D img2 = await imageGen.SetPrompt("Image 2").ExecuteAsync();

// ✅ Good - parallel generation
var tasks = new[]
{
    "Image 1".GENImage().ExecuteAsync(),
    "Image 2".GENImage().ExecuteAsync()
};
await UniTask.WhenAll(tasks);

// ❌ Bad - sequential when parallel is possible
for (int i = 0; i < 10; i++)
{
    await $"Image {i}".GENImage().ExecuteAsync();
}
```

## Next Steps

- [Image Inpainting](image-inpainting.md) - Edit existing images
- [Image to Image](image-to-image.md) - Transform images
