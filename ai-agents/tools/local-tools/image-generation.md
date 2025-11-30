# Image Generation (Local Tool)

Generate images locally in Unity using AI models.

## Overview

Local Image Generation allows agents to:

- Generate images within Unity
- Use local AI models
- Create game assets without API calls
- Generate textures and sprites
- Produce procedural art

## Basic Setup

### Install Local Model

```csharp
// Using Stable Diffusion in Unity
// Install via Package Manager or Unity ML-Agents
```

### Configure Agent

```csharp
agent.AddLocalTool(LocalToolType.ImageGeneration);
```

## Generate Images

### Simple Generation

```csharp
await agent.SendAsync("Generate a fantasy sword");
await agent.SendAsync("Create a fire effect texture");
```

### With Parameters

```csharp
var settings = new LocalImageSettings
{
    Width = 512,
    Height = 512,
    Steps = 20,
    Seed = 42,
    GuidanceScale = 7.5f
};

await agent.GenerateImageLocallyAsync("A medieval castle", settings);
```

## Access Generated Images

### Get Texture

```csharp
agent.onLocalImageGenerated.AddListener((texture, prompt) =>
{
    Debug.Log($"Generated: {texture.width}x{texture.height}");
    
    // Display in UI
    rawImage.texture = texture;
    
    // Save to file
    SaveTexture(texture, "generated.png");
});
```

### Save to Disk

```csharp
void SaveTexture(Texture2D texture, string fileName)
{
    byte[] bytes = texture.EncodeToPNG();
    string path = Path.Combine(Application.persistentDataPath, fileName);
    File.WriteAllBytes(path, bytes);
    
    Debug.Log($"Saved: {path}");
}
```

## Integration with Unity

### Generate Sprite

```csharp
public async UniTask<Sprite> GenerateSprite(string description)
{
    var texture = await agent.GenerateImageLocallyAsync(description);
    
    Sprite sprite = Sprite.Create(
        texture,
        new Rect(0, 0, texture.width, texture.height),
        new Vector2(0.5f, 0.5f)
    );
    
    return sprite;
}

// Usage
var weaponSprite = await GenerateSprite("A flaming sword icon");
itemImage.sprite = weaponSprite;
```

### Generate Material Texture

```csharp
public async UniTask<Material> GenerateMaterial(string description)
{
    var texture = await agent.GenerateImageLocallyAsync(description);
    
    Material material = new Material(Shader.Find("Standard"));
    material.mainTexture = texture;
    
    return material;
}

// Usage
var stoneMaterial = await GenerateMaterial("Stone wall texture, seamless");
GetComponent<Renderer>().material = stoneMaterial;
```

## Model Configuration

### Stable Diffusion Settings

```csharp
agent.LocalImageSettings = new LocalImageSettings
{
    ModelPath = "Models/stable-diffusion-v1-5",
    Width = 512,
    Height = 512,
    Steps = 20,                  // Inference steps (higher = better quality)
    Seed = -1,                   // -1 for random
    GuidanceScale = 7.5f,        // How closely to follow prompt
    NegativePrompt = "blurry, low quality, distorted"
};
```

### Performance Options

```csharp
// Fast generation (lower quality)
agent.LocalImageSettings.Steps = 10;
agent.LocalImageSettings.Width = 256;
agent.LocalImageSettings.Height = 256;

// High quality (slower)
agent.LocalImageSettings.Steps = 50;
agent.LocalImageSettings.Width = 1024;
agent.LocalImageSettings.Height = 1024;
```

## Game-Specific Use Cases

### Character Portrait Generator

```csharp
public class PortraitGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private RawImage portraitDisplay;
    
    public async void GenerateCharacterPortrait(CharacterData character)
    {
        string prompt = $@"
Portrait of {character.Name}:
{character.Race} {character.Class}
{character.Appearance}
Style: fantasy RPG character portrait
Quality: high detail, professional game art
";
        
        var texture = await agent.GenerateImageLocallyAsync(prompt);
        portraitDisplay.texture = texture;
        
        // Save to character data
        character.Portrait = texture;
    }
}
```

### Procedural Terrain Textures

```csharp
public class TerrainTextureGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Terrain terrain;
    
    public async void GenerateTerrainTexture(string terrainType)
    {
        string prompt = $"{terrainType} ground texture, seamless, top-down view";
        
        var texture = await agent.GenerateImageLocallyAsync(prompt);
        
        // Apply to terrain
        TerrainLayer layer = new TerrainLayer
        {
            diffuseTexture = texture,
            tileSize = new Vector2(10, 10)
        };
        
        terrain.terrainData.terrainLayers = new[] { layer };
    }
}

// Usage
GenerateTerrainTexture("grass");
GenerateTerrainTexture("rocky desert");
```

### Item Icon Generator

```csharp
public async UniTask<Sprite> GenerateItemIcon(ItemData item)
{
    string prompt = $@"
Game item icon: {item.Name}
{item.Description}
Style: clean icon, isometric view
Background: transparent or simple gradient
Quality: high detail, 512x512
";
    
    var texture = await agent.GenerateImageLocallyAsync(prompt);
    
    // Create sprite
    return Sprite.Create(
        texture,
        new Rect(0, 0, texture.width, texture.height),
        new Vector2(0.5f, 0.5f)
    );
}
```

## Batch Generation

### Generate Multiple Assets

```csharp
public async void GenerateAssetPack(string[] descriptions)
{
    Debug.Log($"Generating {descriptions.Length} assets...");
    
    List<Texture2D> generatedAssets = new();
    
    foreach (var description in descriptions)
    {
        var texture = await agent.GenerateImageLocallyAsync(description);
        generatedAssets.Add(texture);
        
        // Small delay to prevent overload
        await UniTask.Delay(100);
    }
    
    Debug.Log($"✓ Generated {generatedAssets.Count} assets");
    
    // Save pack
    SaveAssetPack(generatedAssets);
}

void SaveAssetPack(List<Texture2D> textures)
{
    string packPath = Path.Combine(Application.persistentDataPath, "AssetPack");
    Directory.CreateDirectory(packPath);
    
    for (int i = 0; i < textures.Count; i++)
    {
        byte[] bytes = textures[i].EncodeToPNG();
        File.WriteAllBytes(Path.Combine(packPath, $"asset_{i}.png"), bytes);
    }
}
```

## Style Control

### Art Style Presets

```csharp
public enum ArtStyle
{
    Realistic,
    Cartoon,
    PixelArt,
    Watercolor,
    OilPainting
}

public string GetStylePrompt(ArtStyle style)
{
    return style switch
    {
        ArtStyle.Realistic => "photorealistic, high detail, 3D render",
        ArtStyle.Cartoon => "cartoon style, cel-shaded, bold outlines",
        ArtStyle.PixelArt => "pixel art, 16-bit style, retro game graphics",
        ArtStyle.Watercolor => "watercolor painting, soft colors, artistic",
        ArtStyle.OilPainting => "oil painting, brush strokes, classical art",
        _ => ""
    };
}

public async UniTask<Texture2D> GenerateWithStyle(string subject, ArtStyle style)
{
    string stylePrompt = GetStylePrompt(style);
    string fullPrompt = $"{subject}, {stylePrompt}";
    
    return await agent.GenerateImageLocallyAsync(fullPrompt);
}
```

## Image Variations

### Generate Variations

```csharp
public async UniTask<List<Texture2D>> GenerateVariations(string basePrompt, int count)
{
    List<Texture2D> variations = new();
    
    for (int i = 0; i < count; i++)
    {
        // Use different seed for each variation
        agent.LocalImageSettings.Seed = Random.Range(0, 100000);
        
        var texture = await agent.GenerateImageLocallyAsync(basePrompt);
        variations.Add(texture);
    }
    
    return variations;
}

// Usage
var swordVariations = await GenerateVariations("Fantasy sword", 5);
```

## Image-to-Image

### Modify Existing Image

```csharp
public async UniTask<Texture2D> ModifyImage(Texture2D sourceImage, string modification)
{
    // Convert to base64 or file
    byte[] imageBytes = sourceImage.EncodeToPNG();
    
    // Generate based on source
    var result = await agent.GenerateImageToImageAsync(
        sourceImage: imageBytes,
        prompt: modification,
        strength: 0.75f  // How much to change (0-1)
    );
    
    return result;
}

// Usage
var modifiedTexture = await ModifyImage(
    originalTexture,
    "Add glowing magical effects"
);
```

## Performance Optimization

### Async Loading

```csharp
public class AsyncImageGenerator : MonoBehaviour
{
    private Queue<ImageRequest> requestQueue = new();
    private bool isGenerating;
    
    public async void RequestImage(string prompt, System.Action<Texture2D> callback)
    {
        requestQueue.Enqueue(new ImageRequest { Prompt = prompt, Callback = callback });
        
        if (!isGenerating)
        {
            await ProcessQueue();
        }
    }
    
    async UniTask ProcessQueue()
    {
        isGenerating = true;
        
        while (requestQueue.Count > 0)
        {
            var request = requestQueue.Dequeue();
            
            var texture = await agent.GenerateImageLocallyAsync(request.Prompt);
            request.Callback?.Invoke(texture);
            
            await UniTask.Yield(); // Prevent frame drops
        }
        
        isGenerating = false;
    }
    
    struct ImageRequest
    {
        public string Prompt;
        public System.Action<Texture2D> Callback;
    }
}
```

### Texture Pooling

```csharp
public class TexturePool : MonoBehaviour
{
    private Dictionary<string, Texture2D> cache = new();
    
    public async UniTask<Texture2D> GetOrGenerate(string prompt)
    {
        if (cache.TryGetValue(prompt, out var cached))
        {
            Debug.Log("Using cached texture");
            return cached;
        }
        
        var texture = await agent.GenerateImageLocallyAsync(prompt);
        cache[prompt] = texture;
        
        return texture;
    }
    
    public void ClearCache()
    {
        foreach (var texture in cache.Values)
        {
            Destroy(texture);
        }
        cache.Clear();
    }
}
```

## Error Handling

### Handle Generation Errors

```csharp
agent.onLocalToolError.AddListener((tool, error) =>
{
    if (tool == LocalToolType.ImageGeneration)
    {
        Debug.LogError($"Image generation failed: {error}");
        
        if (error.Contains("out of memory"))
        {
            // Reduce resolution
            agent.LocalImageSettings.Width = 256;
            agent.LocalImageSettings.Height = 256;
            Debug.Log("Reduced resolution due to memory constraints");
        }
        else if (error.Contains("model not found"))
        {
            ShowMessage("AI model not installed. Please download model.");
        }
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class LocalImageGenerator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private RawImage displayImage;
    
    [Header("Settings")]
    [SerializeField] private int imageWidth = 512;
    [SerializeField] private int imageHeight = 512;
    [SerializeField] private int inferenceSteps = 20;
    
    private Dictionary<string, Texture2D> cache = new();
    
    async void Start()
    {
        await SetupLocalImageGeneration();
    }
    
    async UniTask SetupLocalImageGeneration()
    {
        // Configure settings
        agent.LocalImageSettings = new LocalImageSettings
        {
            Width = imageWidth,
            Height = imageHeight,
            Steps = inferenceSteps,
            Seed = -1,
            GuidanceScale = 7.5f,
            NegativePrompt = "blurry, low quality, distorted, ugly"
        };
        
        // Add tool
        agent.AddLocalTool(LocalToolType.ImageGeneration);
        
        // Listen for results
        agent.onLocalImageGenerated.AddListener(OnImageGenerated);
        agent.onLocalToolError.AddListener(OnToolError);
        
        Debug.Log("✓ Local image generation ready");
    }
    
    public async void GenerateImage(string prompt)
    {
        // Check cache
        if (cache.TryGetValue(prompt, out var cached))
        {
            displayImage.texture = cached;
            Debug.Log("Using cached image");
            return;
        }
        
        Debug.Log($"🎨 Generating: {prompt}");
        
        try
        {
            var texture = await agent.GenerateImageLocallyAsync(prompt);
            
            // Display
            displayImage.texture = texture;
            
            // Cache
            cache[prompt] = texture;
            
            // Save
            SaveTexture(texture, prompt);
        }
        catch (Exception ex)
        {
            Debug.LogError($"Generation failed: {ex.Message}");
        }
    }
    
    void OnImageGenerated(Texture2D texture, string prompt)
    {
        Debug.Log($"✓ Generated: {texture.width}x{texture.height}");
        displayImage.texture = texture;
    }
    
    void OnToolError(LocalToolType tool, string error)
    {
        if (tool == LocalToolType.ImageGeneration)
        {
            Debug.LogError($"Image generation error: {error}");
        }
    }
    
    void SaveTexture(Texture2D texture, string prompt)
    {
        byte[] bytes = texture.EncodeToPNG();
        
        string fileName = $"{prompt.Replace(" ", "_")}_{DateTime.Now:yyyyMMdd_HHmmss}.png";
        string path = Path.Combine(Application.persistentDataPath, "Generated", fileName);
        
        Directory.CreateDirectory(Path.GetDirectoryName(path));
        File.WriteAllBytes(path, bytes);
        
        Debug.Log($"💾 Saved: {path}");
    }
    
    public void ClearCache()
    {
        foreach (var texture in cache.Values)
        {
            Destroy(texture);
        }
        cache.Clear();
        
        Debug.Log("✓ Cache cleared");
    }
    
    void OnDestroy()
    {
        ClearCache();
    }
}
```

## Next Steps

- [Speech Generation](speech-generation.md)
- [Speech Transcription](speech-transcription.md)
- [Hosted Image Generation](../hosted-tools/image-generation.md)
