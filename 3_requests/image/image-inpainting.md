---
icon: fill-drip
---

# Image Inpainting

Edit specific parts of an existing image using `.GENInpaint()`.

## Basic Usage

```csharp
Texture2D edited = await sourceTexture
    .GENInpaint("Add a red car in the center")
    .ExecuteAsync();
```

## Input Types

### Texture2D + Instruction

```csharp
Texture2D texture = Resources.Load<Texture2D>("Scene");
Texture2D edited = await texture
    .GENInpaint("Replace sky with sunset")
    .ExecuteAsync();
```

### Sprite + Instruction

```csharp
Sprite sprite = Resources.Load<Sprite>("Character");
Texture2D edited = await sprite
    .GENInpaint("Add a sword in right hand")
    .ExecuteAsync();
```

### ImagePrompt

```csharp
var prompt = new ImagePrompt
{
    Image = sourceTexture,
    Instruction = "Remove background"
};

Texture2D edited = await prompt
    .GENInpaint()
    .ExecuteAsync();
```

### File Input

```csharp
var file = new File<Texture2D>(texture, "scene.png");
Texture2D edited = await file
    .GENInpaint("Add clouds")
    .ExecuteAsync();
```

## Configuration

### Model Selection

```csharp
Texture2D edited = await texture
    .GENInpaint("Edit instruction")
    .SetModel(OpenAIModel.DallE2) // Only DALL-E 2 supports inpainting
    .ExecuteAsync();
```

### With Mask

Specify which areas to edit (provider-dependent):

```csharp
Texture2D mask = CreateMask(); // White = edit, Black = keep

var prompt = new ImagePrompt
{
    Image = sourceTexture,
    Mask = mask,
    Instruction = "Fill masked area with grass"
};

Texture2D edited = await prompt.GENInpaint().ExecuteAsync();
```

## Common Use Cases

### 1. Object Removal

```csharp
async UniTask<Texture2D> RemoveObject(Texture2D source, string object)
{
    return await source
        .GENInpaint($"Remove the {object}")
        .ExecuteAsync();
}

// Usage
Texture2D clean = await RemoveObject(screenshot, "UI elements");
```

### 2. Object Addition

```csharp
async UniTask<Texture2D> AddObject(Texture2D source, string object, string position)
{
    return await source
        .GENInpaint($"Add {object} {position}")
        .ExecuteAsync();
}

// Usage
Texture2D edited = await AddObject(scene, "a tree", "on the left side");
```

### 3. Background Replacement

```csharp
async UniTask<Texture2D> ReplaceBackground(Texture2D source, string newBg)
{
    return await source
        .GENInpaint($"Replace background with {newBg}")
        .ExecuteAsync();
}

// Usage
Texture2D studio = await ReplaceBackground(portrait, "professional studio background");
```

### 4. Style Transfer (Selective)

```csharp
async UniTask<Texture2D> StyleTransferArea(Texture2D source, string area, string style)
{
    return await source
        .GENInpaint($"Make the {area} {style}")
        .ExecuteAsync();
}

// Usage
Texture2D edited = await StyleTransferArea(scene, "sky", "watercolor style");
```

### 5. Repair/Fix

```csharp
async UniTask<Texture2D> RepairImage(Texture2D damaged, string issue)
{
    return await damaged
        .GENInpaint($"Fix {issue}")
        .ExecuteAsync();
}

// Usage
Texture2D fixed = await RepairImage(texture, "the corrupted area in the corner");
```

## Unity Integration Examples

### Example 1: Screenshot Editor

```csharp
public class ScreenshotEditor : MonoBehaviour
{
    async void EditLastScreenshot()
    {
        Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();
        
        Texture2D edited = await screenshot
            .GENInpaint("Remove all UI elements")
            .ExecuteAsync();
        
        // Save edited version
        byte[] bytes = edited.EncodeToPNG();
        File.WriteAllBytes("edited_screenshot.png", bytes);
    }
}
```

### Example 2: Texture Touchup Tool

```csharp
public class TextureTouchup : MonoBehaviour
{
    [SerializeField] private Texture2D sourceTexture;
    [SerializeField] private string editInstruction;
    
    public async UniTask<Texture2D> ApplyEdit()
    {
        return await sourceTexture
            .GENInpaint(editInstruction)
            .SetModel(OpenAIModel.DallE2)
            .ExecuteAsync();
    }
}
```

### Example 3: Dynamic Scene Editing

```csharp
public class SceneEditor : MonoBehaviour
{
    private Texture2D currentScene;
    
    public async UniTask EditScene(string instruction)
    {
        // Capture current scene
        currentScene = ScreenCapture.CaptureScreenshotAsTexture();
        
        // Apply edit
        Texture2D edited = await currentScene
            .GENInpaint(instruction)
            .ExecuteAsync();
        
        // Apply to skybox or material
        ApplyToScene(edited);
    }
    
    void ApplyToScene(Texture2D texture)
    {
        RenderSettings.skybox.SetTexture("_MainTex", texture);
    }
}
```

### Example 4: Character Customization

```csharp
public class CharacterCustomizer : MonoBehaviour
{
    [SerializeField] private Image characterPortrait;
    
    public async UniTask CustomizeCharacter(string customization)
    {
        Texture2D current = characterPortrait.sprite.texture;
        
        Texture2D customized = await current
            .GENInpaint(customization)
            .ExecuteAsync();
        
        Sprite newSprite = Sprite.Create(
            customized,
            new Rect(0, 0, customized.width, customized.height),
            new Vector2(0.5f, 0.5f)
        );
        
        characterPortrait.sprite = newSprite;
    }
}

// Usage
await customizer.CustomizeCharacter("Add a wizard hat");
await customizer.CustomizeCharacter("Change hair color to blue");
```

### Example 5: Batch Processing

```csharp
async UniTask<List<Texture2D>> BatchEdit(List<Texture2D> textures, string instruction)
{
    var tasks = textures.Select(t =>
        t.GENInpaint(instruction).ExecuteAsync()
    );
    
    Texture2D[] results = await UniTask.WhenAll(tasks);
    return results.ToList();
}

// Usage
var edited = await BatchEdit(
    new List<Texture2D> { tex1, tex2, tex3 },
    "Remove watermarks"
);
```

## Instruction Tips

### ✅ Good Instructions

```csharp
// ✅ Specific and clear
"Add a red sports car in the center of the road"

// ✅ Describe desired outcome
"Replace the cloudy sky with a clear blue sky"

// ✅ Include style/quality hints
"Remove the person on the left, maintain photo-realistic quality"

// ✅ Specify area clearly
"Fill the top-right corner with autumn leaves"
```

### ❌ Bad Instructions

```csharp
// ❌ Too vague
"fix it"

// ❌ Multiple conflicting edits
"add trees and remove trees and change sky"

// ❌ Unclear location
"put something there"

// ❌ Impossible requests
"remove the main subject but keep the image interesting"
```

## Provider Support

### OpenAI DALL-E 2

```csharp
// Only DALL-E 2 supports inpainting
Texture2D edited = await texture
    .GENInpaint("Edit instruction")
    .SetModel(OpenAIModel.DallE2)
    .ExecuteAsync();
```

**Note:** DALL-E 3 does not support inpainting. Use DALL-E 2 or image-to-image instead.

## Limitations

1. **Square Images Only**: Most providers require square input images
2. **Size Limits**: Input images may be resized to fit model requirements
3. **Quality**: Edits may not perfectly match original style
4. **Provider-Specific**: Different providers have different capabilities

## Best Practices

### ✅ Do

- Use clear, specific instructions
- Prepare images in supported formats
- Start with smaller edits
- Test instructions with sample images
- Cache results when appropriate

### ❌ Don't

- Use extremely high-resolution images (they'll be resized)
- Make multiple major edits in one instruction
- Expect pixel-perfect results
- Forget to validate output

## Error Handling

```csharp
try
{
    Texture2D edited = await texture
        .GENInpaint("Edit instruction")
        .ExecuteAsync();
    
    if (edited == null || edited.width == 0)
        throw new Exception("Invalid edited image");
    
    // Use edited image
}
catch (AIApiException ex)
{
    Debug.LogError($"Inpainting failed: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Next Steps

- [Image to Image](image-to-image.md) - Full image transformation
- [Text to Image](text-to-image.md) - Generate new images
