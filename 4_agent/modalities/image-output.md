---
icon: image
---

# Image Output (Generation)

Generate images from text descriptions using AI image generation.

## Overview

AI Dev Kit agents can generate images from text prompts:

- **Text-to-Image** - Create images from descriptions
- **Style Control** - Specify artistic styles
- **Size Options** - Multiple resolution options
- **Batch Generation** - Generate multiple variations
- **Provider Support** - OpenAI DALL-E, Stability AI, and more

## Basic Usage

### Generate Single Image

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ImageGeneration : MonoBehaviour
{
    private Agent agent;
    
    async void Start()
    {
        agent = new Agent(new AgentConfiguration 
        { 
            ImageGenerationProvider = AiProvider.OpenAI
        });
        
        await agent.InitializeAsync();
        
        // Generate an image
        var images = await agent.GenerateImagesAsync(
            "A serene mountain landscape at sunset, photorealistic"
        );
        
        // Use the generated image
        Texture2D texture = images[0].ToTexture2D();
        GetComponent<SpriteRenderer>().sprite = Sprite.Create(
            texture,
            new Rect(0, 0, texture.width, texture.height),
            new Vector2(0.5f, 0.5f)
        );
    }
}
```

### Generate Multiple Images

```csharp
// Generate multiple variations
var images = await agent.GenerateImagesAsync(
    "A cute robot character",
    count: 4
);

// Display all variations
foreach (var image in images)
{
    GameObject go = new GameObject("GeneratedImage");
    SpriteRenderer sr = go.AddComponent<SpriteRenderer>();
    Texture2D texture = image.ToTexture2D();
    sr.sprite = Sprite.Create(
        texture,
        new Rect(0, 0, texture.width, texture.height),
        new Vector2(0.5f, 0.5f)
    );
}
```

## Image Size Options

```csharp
// DALL-E 3 sizes
await agent.GenerateImagesAsync(
    prompt,
    size: ImageSize.Square_1024    // 1024x1024
);

await agent.GenerateImagesAsync(
    prompt,
    size: ImageSize.Landscape_1792 // 1792x1024
);

await agent.GenerateImagesAsync(
    prompt,
    size: ImageSize.Portrait_1024  // 1024x1792
);

// DALL-E 2 sizes
await agent.GenerateImagesAsync(
    prompt,
    size: ImageSize.Square_256,    // 256x256
    model: "dall-e-2"
);
```

## Quality Settings

```csharp
// Standard quality (faster, cheaper)
var images = await agent.GenerateImagesAsync(
    prompt,
    quality: ImageQuality.Standard
);

// HD quality (slower, more expensive, better detail)
var images = await agent.GenerateImagesAsync(
    prompt,
    quality: ImageQuality.HD
);
```

## Style Control

```csharp
// Natural style (photorealistic)
var images = await agent.GenerateImagesAsync(
    "A coffee shop interior",
    style: ImageStyle.Natural
);

// Vivid style (more dramatic, artistic)
var images = await agent.GenerateImagesAsync(
    "A futuristic cityscape",
    style: ImageStyle.Vivid
);
```

## Prompt Engineering

### Effective Prompts

```csharp
// ✓ Good - detailed, specific
await agent.GenerateImagesAsync(
    \"A photorealistic portrait of a female warrior, \" +\n    \"medieval plate armor, sunset lighting, \" +\n    \"cinematic composition, detailed textures\"\n);\n\n// ✗ Poor - too vague\nawait agent.GenerateImagesAsync(\"a person\");\n```\n\n### Prompt Components\n\n```csharp\nstring prompt = \n    // Subject\n    \"A majestic dragon \" +\n    \n    // Details\n    \"with scales, wings spread, breathing fire \" +\n    \n    // Environment\n    \"perched on a mountain peak, stormy sky \" +\n    \n    // Style\n    \"fantasy art style, dramatic lighting \" +\n    \n    // Quality\n    \"highly detailed, 8k\";\n\nvar images = await agent.GenerateImagesAsync(prompt);\n```\n\n## Use Cases\n\n### Concept Art\n\n```csharp\nvar characterConcepts = await agent.GenerateImagesAsync(\n    \"Character concept art: sci-fi space explorer, \" +\n    \"futuristic suit, helmet, technical details\",\n    count: 4\n);\n```\n\n### Asset Creation\n\n```csharp\nvar texture = await agent.GenerateImagesAsync(\n    \"Seamless stone wall texture, high resolution, \" +\n    \"weathered surface, neutral lighting\"\n);\n```\n\n### UI Mockups\n\n```csharp\nvar uiMockup = await agent.GenerateImagesAsync(\n    \"Mobile app UI mockup for a fitness tracker, \" +\n    \"clean design, modern interface, dashboard view\"\n);\n```\n\n### Icons and Sprites\n\n```csharp\nvar icon = await agent.GenerateImagesAsync(\n    \"Game icon: magical sword, glowing blue, \" +\n    \"simple design, transparent background style\",\n    size: ImageSize.Square_256\n);\n```\n\n## Events\n\n```csharp\n// Generation started\nagent.OnImageGenerationStarted += (prompt) => {\n    Debug.Log($\"Generating: {prompt}\");\n    ShowLoadingIndicator();\n};\n\n// Generation progress\nagent.OnImageGenerationProgress += (progress) => {\n    UpdateProgressBar(progress);\n};\n\n// Generation completed\nagent.OnImageGenerated += (images) => {\n    Debug.Log($\"Generated {images.Count} images\");\n    HideLoadingIndicator();\n    DisplayImages(images);\n};\n\n// Generation failed\nagent.OnImageGenerationFailed += (error) => {\n    Debug.LogError($\"Generation failed: {error.Message}\");\n    ShowErrorMessage(error.Message);\n};\n```\n\n## Save Generated Images\n\n### Save to File\n\n```csharp\nasync void SaveGeneratedImage()\n{\n    var images = await agent.GenerateImagesAsync(\"a beautiful sunset\");\n    \n    Texture2D texture = images[0].ToTexture2D();\n    byte[] bytes = texture.EncodeToPNG();\n    \n    string path = Path.Combine(\n        Application.persistentDataPath,\n        \"generated_image.png\"\n    );\n    \n    File.WriteAllBytes(path, bytes);\n    Debug.Log($\"Saved to: {path}\");\n}\n```\n\n### Save to Sprite\n\n```csharp\npublic Sprite CreateSpriteFromGenerated(Texture2D texture)\n{\n    return Sprite.Create(\n        texture,\n        new Rect(0, 0, texture.width, texture.height),\n        new Vector2(0.5f, 0.5f),\n        pixelsPerUnit: 100\n    );\n}\n```\n\n## Provider Options\n\n### OpenAI DALL-E\n\n```csharp\nvar config = new AgentConfiguration\n{\n    ImageGenerationProvider = AiProvider.OpenAI,\n    ImageModel = \"dall-e-3\"\n};\n\nAgent agent = new Agent(config);\n```\n\n**Features:**\n- High quality results\n- Good prompt following\n- Multiple sizes\n- Style control\n\n### Stability AI\n\n```csharp\nvar config = new AgentConfiguration\n{\n    ImageGenerationProvider = AiProvider.StabilityAI,\n    ImageModel = \"stable-diffusion-xl-1024-v1-0\"\n};\n```\n\n**Features:**\n- More artistic styles\n- Lower cost\n- Faster generation\n- More control options\n\n## Cost Optimization\n\n```csharp\n// Monitor costs\nagent.OnImageGenerated += (images) => {\n    float cost = CalculateImageCost(images);\n    Debug.Log($\"Generation cost: ${cost}\");\n};\n\n// Use smaller sizes when possible\nvar images = await agent.GenerateImagesAsync(\n    prompt,\n    size: ImageSize.Square_1024  // Cheaper than 1792\n);\n\n// Use standard quality\nvar images = await agent.GenerateImagesAsync(\n    prompt,\n    quality: ImageQuality.Standard  // Cheaper than HD\n);\n```\n\n## Best Practices\n\n### 1. Be Descriptive\n\n```csharp\n// ✓ Good\nawait agent.GenerateImagesAsync(\n    \"A cozy coffee shop interior, warm lighting, \" +\n    \"wooden furniture, plants, morning atmosphere\"\n);\n\n// ✗ Poor\nawait agent.GenerateImagesAsync(\"coffee shop\");\n```\n\n### 2. Specify Style\n\n```csharp\n// For game assets\nawait agent.GenerateImagesAsync(\n    \"Forest background, 2D game art style, flat colors\"\n);\n\n// For realistic scenes\nawait agent.GenerateImagesAsync(\n    \"City street, photorealistic, natural lighting\"\n);\n```\n\n### 3. Use Appropriate Size\n\n```csharp\n// For UI elements - smaller is fine\nvar icon = await agent.GenerateImagesAsync(\n    prompt,\n    size: ImageSize.Square_256\n);\n\n// For detailed artwork - use larger\nvar artwork = await agent.GenerateImagesAsync(\n    prompt,\n    size: ImageSize.Square_1024\n);\n```\n\n### 4. Iterate and Refine\n\n```csharp\n// Generate variations\nvar initialImages = await agent.GenerateImagesAsync(\n    basePrompt,\n    count: 4\n);\n\n// Select best and refine\nvar refined = await agent.GenerateImagesAsync(\n    basePrompt + \", more detailed, better composition\"\n);\n```\n\n## Limitations\n\n- **Content Policy**: No inappropriate or harmful content\n- **Generation Time**: Can take 10-30 seconds\n- **Rate Limits**: Check provider limits\n- **Cost**: Each generation consumes credits\n- **Consistency**: Hard to generate exact same image twice\n\n## Troubleshooting\n\n### Content Policy Violation\n\n```csharp\ntry\n{\n    var images = await agent.GenerateImagesAsync(prompt);\n}\ncatch (ContentPolicyException ex)\n{\n    Debug.LogError(\"Prompt violated content policy\");\n    // Revise prompt\n}\n```\n\n### Poor Quality Results\n\n```csharp\n// Use HD quality\nquality: ImageQuality.HD\n\n// Be more specific in prompt\nprompt += \", highly detailed, professional quality\";\n\n// Try different style\nstyle: ImageStyle.Vivid\n```\n\n### Generation Timeout\n\n```csharp\n// Increase timeout\nagent.Configuration.ImageGenerationTimeout = TimeSpan.FromMinutes(2);\n\n// Or retry\nint maxRetries = 3;\nfor (int i = 0; i < maxRetries; i++)\n{\n    try\n    {\n        var images = await agent.GenerateImagesAsync(prompt);\n        break;\n    }\n    catch (TimeoutException)\n    {\n        if (i == maxRetries - 1) throw;\n        await Task.Delay(1000);\n    }\n}\n```\n\n## Next Steps\n\n- Learn about [Image Input (Vision)](image-input.md)\n- Explore [Voice Input](voice-input.md)\n- Configure [Agent Settings](../essentials/configuration/agent-settings.md)\n"