# Image Generation (Hosted Tool)

Generate images using DALL-E through OpenAI's API.

## Overview

Image Generation allows agents to:

- Create images from text descriptions
- Generate variations of existing images
- Edit images with natural language
- Create game assets
- Produce concept art

## Basic Setup

### Enable Image Generation

```csharp
// Add image generation tool
agent.AddTool(ToolType.ImageGeneration);
```

## Generate Images

### Simple Generation

```csharp
await agent.SendAsync("Generate an image of a medieval castle at sunset");
await agent.SendAsync("Create a portrait of a futuristic robot");
await agent.SendAsync("Draw a fantasy forest with glowing mushrooms");
```

### With Parameters

```csharp
await agent.SendAsync(@"
Generate a high-quality image of:
A cyberpunk city street at night with neon signs,
flying cars, and rain-soaked pavement.
Style: photorealistic, high detail
");
```

## Access Generated Images

### Download Images

```csharp
agent.onResponseCompleted.AddListener(async response =>
{
    if (response.GeneratedImages != null && response.GeneratedImages.Count > 0)
    {
        Debug.Log($"Generated {response.GeneratedImages.Count} image(s)");
        
        foreach (var image in response.GeneratedImages)
        {
            Debug.Log($"Image URL: {image.Url}");
            
            // Download image
            byte[] imageData = await DownloadImage(image.Url);
            
            // Save to file
            string fileName = $"generated_{DateTime.Now:yyyyMMdd_HHmmss}.png";
            string savePath = Path.Combine(Application.persistentDataPath, fileName);
            File.WriteAllBytes(savePath, imageData);
            
            Debug.Log($"Saved: {savePath}");
        }
    }
});

async UniTask<byte[]> DownloadImage(string url)
{
    using var request = UnityWebRequest.Get(url);
    await request.SendWebRequest();
    return request.downloadHandler.data;
}
```

### Display in Unity

```csharp
public class ImageDisplay : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private RawImage displayImage;
    
    void Start()
    {
        agent.AddTool(ToolType.ImageGeneration);
        agent.onResponseCompleted.AddListener(DisplayGeneratedImages);
    }
    
    async void DisplayGeneratedImages(Response response)
    {
        if (response.GeneratedImages == null || response.GeneratedImages.Count == 0)
            return;
        
        var firstImage = response.GeneratedImages[0];
        
        // Download and display
        byte[] imageData = await DownloadImage(firstImage.Url);
        Texture2D texture = new Texture2D(2, 2);
        texture.LoadImage(imageData);
        
        displayImage.texture = texture;
        
        Debug.Log($"✓ Image displayed: {texture.width}x{texture.height}");
    }
    
    async UniTask<byte[]> DownloadImage(string url)
    {
        using var request = UnityWebRequest.Get(url);
        await request.SendWebRequest();
        return request.downloadHandler.data;
    }
}
```

## Image Settings

### Configure Generation

```csharp
agent.Settings.ImageGeneration = new ImageGenerationSettings
{
    Model = "dall-e-3",           // dall-e-2, dall-e-3
    Size = "1024x1024",           // 256x256, 512x512, 1024x1024, 1792x1024, 1024x1792
    Quality = "standard",         // standard, hd
    Style = "vivid",              // vivid, natural
    NumberOfImages = 1            // 1-10 for dall-e-2, 1 for dall-e-3
};
```

### Quality Options

```csharp
// Standard quality (faster, cheaper)
agent.Settings.ImageGeneration.Quality = "standard";

// HD quality (slower, more expensive, better detail)
agent.Settings.ImageGeneration.Quality = "hd";
```

### Style Options

```csharp
// Vivid: More imaginative and artistic
agent.Settings.ImageGeneration.Style = "vivid";

// Natural: More realistic and natural
agent.Settings.ImageGeneration.Style = "natural";
```

## Game Asset Generation

### Character Concepts

```csharp
public class CharacterArtGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool(ToolType.ImageGeneration);
    }
    
    public async void GenerateCharacter(string description)
    {
        await agent.SendAsync($@"
Generate a character concept art:
{description}

Style: detailed concept art, professional game character design
View: front view, full body
");
    }
    
    public async void GeneratePortrait(string characterName, string description)
    {
        await agent.SendAsync($@"
Generate a character portrait for {characterName}:
{description}

Style: high quality portrait, game character art
Composition: head and shoulders, focused on face
");
    }
}

// Usage
artGenerator.GenerateCharacter("A female elven warrior with silver hair and emerald eyes, wearing leather armor");
artGenerator.GeneratePortrait("Aria", "A confident mage with blue robes and glowing staff");
```

### Environment Art

```csharp
public async void GenerateEnvironment(string description)
{
    await agent.SendAsync($@"
Generate game environment concept:
{description}

Style: professional environment concept art
Lighting: atmospheric, mood-appropriate
Detail: high quality, suitable for game reference
");
}

// Usage
GenerateEnvironment("A mystical underground cavern with glowing crystals and ancient ruins");
GenerateEnvironment("A futuristic space station interior with holographic displays");
```

### Item/Weapon Art

```csharp
public async void GenerateItem(string itemName, string description)
{
    await agent.SendAsync($@"
Generate game item icon for {itemName}:
{description}

Style: game asset icon, clear silhouette
View: isometric or 3/4 view
Background: transparent or simple gradient
Quality: high detail, clean design
");
}

// Usage
GenerateItem("Flaming Sword", "A legendary sword with flames wrapping around the blade");
GenerateItem("Health Potion", "A red potion in an ornate glass bottle with magical glow");
```

## UI Integration

### Image Gallery

```csharp
public class ImageGallery : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject imagePrefab;
    [SerializeField] private Transform gallery Container;
    
    private List<Texture2D> generatedImages = new();
    
    void Start()
    {
        agent.AddTool(ToolType.ImageGeneration);
        agent.onResponseCompleted.AddListener(AddToGallery);
    }
    
    async void AddToGallery(Response response)
    {
        if (response.GeneratedImages == null) return;
        
        foreach (var image in response.GeneratedImages)
        {
            // Download
            byte[] imageData = await DownloadImage(image.Url);
            
            // Create texture
            Texture2D texture = new Texture2D(2, 2);
            texture.LoadImage(imageData);
            generatedImages.Add(texture);
            
            // Display in gallery
            var imageObj = Instantiate(imagePrefab, galleryContainer);
            var rawImage = imageObj.GetComponent<RawImage>();
            rawImage.texture = texture;
            
            // Add interaction
            var button = imageObj.GetComponent<Button>();
            button.onClick.AddListener(() => ShowFullSize(texture));
        }
    }
    
    void ShowFullSize(Texture2D texture)
    {
        // Display full-size image
        Debug.Log($"Showing image: {texture.width}x{texture.height}");
    }
    
    async UniTask<byte[]> DownloadImage(string url)
    {
        using var request = UnityWebRequest.Get(url);
        await request.SendWebRequest();
        return request.downloadHandler.data;
    }
}
```

### Save to Assets

```csharp
public class ImageAssetSaver : MonoBehaviour
{
    [SerializeField] private string savePath = "Assets/GeneratedArt";
    
    public async void SaveImage(string url, string fileName)
    {
        // Download
        byte[] imageData = await DownloadImage(url);
        
        // Ensure directory exists
        Directory.CreateDirectory(savePath);
        
        // Save
        string fullPath = Path.Combine(savePath, $"{fileName}.png");
        File.WriteAllBytes(fullPath, imageData);
        
#if UNITY_EDITOR
        // Refresh asset database in editor
        UnityEditor.AssetDatabase.Refresh();
#endif
        
        Debug.Log($"✓ Saved to: {fullPath}");
    }
    
    async UniTask<byte[]> DownloadImage(string url)
    {
        using var request = UnityWebRequest.Get(url);
        await request.SendWebRequest();
        return request.downloadHandler.data;
    }
}
```

## Advanced Usage

### Batch Generation

```csharp
public async void GenerateBatch(string[] prompts)
{
    Debug.Log($"Generating {prompts.Length} images...");
    
    foreach (var prompt in prompts)
    {
        await agent.SendAsync($"Generate: {prompt}");
        await UniTask.Delay(1000); // Rate limiting
    }
    
    Debug.Log("✓ Batch generation complete");
}

// Usage
string[] characterVariations = {
    "A warrior with red armor",
    "A warrior with blue armor",
    "A warrior with gold armor"
};
GenerateBatch(characterVariations);
```

### Iterative Refinement

```csharp
public async void RefineImage(string originalPrompt, string refinement)
{
    await agent.SendAsync($@"
Generate an image similar to: '{originalPrompt}'
But with this change: {refinement}

Keep the overall style and composition similar.
");
}

// Usage
RefineImage(
    "A fantasy castle on a hill",
    "Add a dragon flying above the castle"
);
```

## Error Handling

### Handle Generation Errors

```csharp
agent.onToolCallError.AddListener((toolCall, error) =>
{
    if (toolCall.Function.Name == "image_generation")
    {
        Debug.LogError($"Image generation failed: {error}");
        
        if (error.Contains("content_policy"))
        {
            ShowMessage("Image rejected: Content policy violation");
        }
        else if (error.Contains("rate_limit"))
        {
            ShowMessage("Too many requests. Please wait.");
        }
        else
        {
            ShowMessage("Image generation failed. Please try again.");
        }
    }
});
```

## Best Practices

### 1. Clear Descriptions

```csharp
// ❌ Vague
await agent.SendAsync("Generate a character");

// ✅ Specific
await agent.SendAsync(@"
Generate a character:
- Female elf ranger
- Long silver hair, green eyes
- Leather armor with leaf patterns
- Holding a bow
- Forest background
- Concept art style
");
```

### 2. Include Style Direction

```csharp
await agent.SendAsync(@"
Generate: A fantasy sword

Style: game asset icon
Detail level: high
View: side view, 45 degree angle
Background: simple gradient
Lighting: dramatic, clear silhouette
");
```

### 3. Specify Use Case

```csharp
// For icons
"Style: clean icon, suitable for UI, 512x512"

// For concept art
"Style: professional concept art, detailed, painterly"

// For textures
"Style: seamless texture, tileable, top-down view"
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class GameArtGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private RawImage displayImage;
    [SerializeField] private string savePath = "GeneratedArt";
    
    private List<Texture2D> generatedImages = new();
    
    async void Start()
    {
        await SetupImageGeneration();
    }
    
    async UniTask SetupImageGeneration()
    {
        // Configure settings
        agent.Settings.ImageGeneration = new ImageGenerationSettings
        {
            Model = "dall-e-3",
            Size = "1024x1024",
            Quality = "hd",
            Style = "vivid"
        };
        
        // Add tool
        agent.AddTool(ToolType.ImageGeneration);
        
        // Listen for results
        agent.onResponseCompleted.AddListener(HandleGeneratedImages);
        
        Debug.Log("✓ Image generation ready");
    }
    
    public async void GenerateCharacterArt(string description)
    {
        Debug.Log($"🎨 Generating character: {description}");
        
        await agent.SendAsync($@"
Generate a professional game character concept art:

{description}

Style: detailed digital painting, game character design
View: full body, front view
Quality: high detail, professional artwork
Background: simple, neutral, with subtle atmosphere
");
    }
    
    public async void GenerateEnvironment(string description)
    {
        Debug.Log($"🎨 Generating environment: {description}");
        
        await agent.SendAsync($@"
Generate game environment concept art:

{description}

Style: professional environment design
Composition: wide angle, establishing shot
Lighting: atmospheric, mood-setting
Quality: high detail, game-ready reference
");
    }
    
    public async void GenerateItemIcon(string itemName, string description)
    {
        Debug.Log($"🎨 Generating item icon: {itemName}");
        
        await agent.SendAsync($@"
Generate a game item icon for '{itemName}':

{description}

Style: clean game icon, clear silhouette
View: isometric or 3/4 view
Background: simple gradient or transparent
Quality: high detail, suitable for game UI
Size: optimized for 512x512 icon
");
    }
    
    async void HandleGeneratedImages(Response response)
    {
        if (response.GeneratedImages == null || response.GeneratedImages.Count == 0)
            return;
        
        Debug.Log($"✓ {response.GeneratedImages.Count} image(s) generated");
        
        foreach (var image in response.GeneratedImages)
        {
            await ProcessImage(image);
        }
    }
    
    async UniTask ProcessImage(GeneratedImage image)
    {
        try
        {
            // Download
            byte[] imageData = await DownloadImage(image.Url);
            
            // Create texture
            Texture2D texture = new Texture2D(2, 2);
            texture.LoadImage(imageData);
            generatedImages.Add(texture);
            
            // Display
            displayImage.texture = texture;
            Debug.Log($"✓ Image loaded: {texture.width}x{texture.height}");
            
            // Save
            await SaveImage(imageData, $"generated_{DateTime.Now:yyyyMMdd_HHmmss}");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Failed to process image: {ex.Message}");
        }
    }
    
    async UniTask<byte[]> DownloadImage(string url)
    {
        using var request = UnityWebRequest.Get(url);
        await request.SendWebRequest();
        
        if (request.result != UnityWebRequest.Result.Success)
        {
            throw new Exception($"Download failed: {request.error}");
        }
        
        return request.downloadHandler.data;
    }
    
    async UniTask SaveImage(byte[] imageData, string fileName)
    {
        // Ensure directory exists
        string fullPath = Path.Combine(Application.persistentDataPath, savePath);
        Directory.CreateDirectory(fullPath);
        
        // Save file
        string filePath = Path.Combine(fullPath, $"{fileName}.png");
        await File.WriteAllBytesAsync(filePath, imageData);
        
        Debug.Log($"💾 Saved: {filePath}");
    }
    
    public void ClearGeneratedImages()
    {
        foreach (var texture in generatedImages)
        {
            Destroy(texture);
        }
        generatedImages.Clear();
        
        Debug.Log("✓ Cleared generated images");
    }
}
```

## Next Steps

- [Local Image Generation](../local-tools/image-generation.md)
- [File Search](file-search.md)
- [Code Interpreter](code-interpreter.md)
