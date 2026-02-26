---
icon: eye
---

# Image Input (Vision)

Enable your agent to see and understand images through vision capabilities.

## Overview

Vision allows your agent to analyze and understand visual content:

- **Image Description** - Describe what's in an image
- **Object Detection** - Identify objects and their locations
- **Text Extraction (OCR)** - Read text from images
- **Scene Understanding** - Understand context and relationships
- **Visual Question Answering** - Answer questions about images

## Basic Usage

### Send Image with Message

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class VisionExample : MonoBehaviour
{
    private Agent agent;
    
    async void Start()
    {
        agent = new Agent(new AgentConfiguration 
        { 
            Model = "gpt-4o",
            SupportsVision = true
        });
        
        await agent.InitializeAsync();
        
        // Analyze an image from URL
        var response = await agent.SendMessageAsync(
            "What objects do you see in this image?",
            imageUrl: "https://example.com/photo.jpg"
        );
        
        Debug.Log(response.Content);
    }
}
```

### Send Multiple Images

```csharp
// Analyze multiple images
var response = await agent.SendMessageAsync(
    "Compare these two images. What are the differences?",
    imageUrls: new[]
    {
        "https://example.com/image1.jpg",
        "https://example.com/image2.jpg"
    }
);
```

## Image Sources

### From URL

```csharp
// Public URL
await agent.SendMessageAsync(
    "Describe this scene",
    imageUrl: "https://example.com/scene.jpg"
);
```

### From Local File

```csharp
// Load from local file
string imagePath = "Assets/Images/photo.jpg";
byte[] imageBytes = File.ReadAllBytes(imagePath);
string base64Image = Convert.ToBase64String(imageBytes);

await agent.SendMessageAsync(
    "What's in this image?",
    imageBase64: base64Image
);
```

### From Texture2D

```csharp
// Convert Unity Texture2D to base64
Texture2D texture = GetComponent<SpriteRenderer>().sprite.texture;
byte[] imageBytes = texture.EncodeToPNG();
string base64Image = Convert.ToBase64String(imageBytes);

await agent.SendMessageAsync(
    "Analyze this texture",
    imageBase64: base64Image,
    imageMimeType: "image/png"
);
```

### From Screenshot

```csharp
// Capture and analyze screenshot
async void AnalyzeScreen()
{
    // Capture screenshot
    Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();
    
    // Convert to base64
    byte[] imageBytes = screenshot.EncodeToPNG();
    string base64Image = Convert.ToBase64String(imageBytes);
    
    // Analyze
    var response = await agent.SendMessageAsync(
        "What UI elements are visible in this screenshot?",
        imageBase64: base64Image
    );
    
    Debug.Log(response.Content);
    
    // Clean up
    Destroy(screenshot);
}
```

### From Webcam

```csharp
using UnityEngine;

public class WebcamVision : MonoBehaviour
{
    private WebCamTexture webcam;
    private Agent agent;
    
    async void Start()
    {
        // Initialize agent
        agent = new Agent(new AgentConfiguration { Model = "gpt-4o" });
        await agent.InitializeAsync();
        
        // Start webcam
        webcam = new WebCamTexture();
        webcam.Play();
    }
    
    async void CaptureAndAnalyze()
    {
        // Create texture from webcam frame
        Texture2D snapshot = new Texture2D(webcam.width, webcam.height);
        snapshot.SetPixels(webcam.GetPixels());
        snapshot.Apply();
        
        // Convert to base64
        byte[] imageBytes = snapshot.EncodeToJPG();
        string base64Image = Convert.ToBase64String(imageBytes);
        
        // Analyze
        var response = await agent.SendMessageAsync(
            "What do you see?",
            imageBase64: base64Image
        );
        
        Debug.Log(response.Content);
        
        // Clean up
        Destroy(snapshot);
    }
}
```

## Vision Detail Levels

Control the level of detail in image analysis:

```csharp
// Low detail - faster, cheaper
var response = await agent.SendMessageAsync(
    "Quick description",
    imageUrl: photoUrl,
    visionDetail: VisionDetail.Low
);

// Auto - AI decides optimal detail level
var response = await agent.SendMessageAsync(
    "Analyze this",
    imageUrl: photoUrl,
    visionDetail: VisionDetail.Auto
);

// High detail - slower, more expensive, more accurate
var response = await agent.SendMessageAsync(
    "Detailed analysis with all text",
    imageUrl: documentUrl,
    visionDetail: VisionDetail.High
);
```

## Use Cases

### Image Description

```csharp
var response = await agent.SendMessageAsync(
    "Describe this image in detail",
    imageUrl: imageUrl
);
```

### Object Detection

```csharp
var response = await agent.SendMessageAsync(
    "List all objects visible in this image with their approximate locations",
    imageUrl: imageUrl
);
```

### OCR (Text Extraction)

```csharp
var response = await agent.SendMessageAsync(
    "Extract all text from this image",
    imageUrl: documentUrl,
    visionDetail: VisionDetail.High
);
```

### Scene Understanding

```csharp
var response = await agent.SendMessageAsync(
    "Describe the setting, atmosphere, and what's happening in this scene",
    imageUrl: sceneUrl
);
```

### Visual Q&A

```csharp
// Ask specific questions
var response = await agent.SendMessageAsync(
    "How many people are in this photo? What are they doing?",
    imageUrl: photoUrl
);
```

### Code Reading

```csharp
var response = await agent.SendMessageAsync(
    "What does this code do? Explain line by line.",
    imageBase64: codeScreenshot
);
```

### Chart Analysis

```csharp
var response = await agent.SendMessageAsync(
    "Analyze this chart. What are the key trends?",
    imageUrl: chartUrl
);
```

## Image Optimization

### Resize Before Sending

```csharp
public static string OptimizeImageForVision(Texture2D original)
{
    // Resize to optimal dimensions
    int maxSize = 2048;
    Texture2D resized = original;
    
    if (original.width > maxSize || original.height > maxSize)
    {
        float scale = Mathf.Min(
            maxSize / (float)original.width,
            maxSize / (float)original.height
        );
        
        int newWidth = Mathf.RoundToInt(original.width * scale);
        int newHeight = Mathf.RoundToInt(original.height * scale);
        
        resized = ResizeTexture(original, newWidth, newHeight);
    }
    
    // Convert to JPEG for smaller size
    byte[] imageBytes = resized.EncodeToJPG(85);
    return Convert.ToBase64String(imageBytes);
}
```

### Compress Images

```csharp
// Use JPEG with quality setting
byte[] imageBytes = texture.EncodeToJPG(quality: 80);

// Or PNG for lossless (larger file)
byte[] imageBytes = texture.EncodeToPNG();
```

## Events

```csharp
// Vision analysis started
agent.OnVisionStarted += (imageUrl) => {
    Debug.Log($"Analyzing image: {imageUrl}");
    ShowLoadingIndicator();
};

// Vision analysis completed
agent.OnVisionCompleted += (result) => {
    Debug.Log($"Analysis complete: {result.Description}");
    HideLoadingIndicator();
};

// Vision analysis failed
agent.OnVisionFailed += (error) => {
    Debug.LogError($"Vision failed: {error.Message}");
    ShowErrorMessage(error.Message);
};
```

## Supported Models

Not all models support vision. Check compatibility:

```csharp
// Vision-capable models
var visionModels = new[]
{
    "gpt-4o",
    "gpt-4o-mini",
    "gpt-4-turbo",
    "claude-3-opus",
    "claude-3-sonnet",
    "claude-3-haiku",
    "gemini-1.5-pro",
    "gemini-1.5-flash"
};

// Check if current model supports vision
if (!agent.CurrentModel.SupportsVision)
{
    Debug.LogWarning("Current model doesn't support vision");
    agent.SetModel("gpt-4o");
}
```

## Cost Considerations

Vision requests consume more tokens:

```csharp
// Monitor token usage
agent.OnResponseCompleted += (response) => {
    Debug.Log($"Input tokens: {response.Usage.InputTokens}");
    Debug.Log($"Output tokens: {response.Usage.OutputTokens}");
    Debug.Log($"Total cost: ${response.Usage.TotalCost}");
};
```

**Tips to reduce costs:**
- Use lower resolution images when possible
- Use `VisionDetail.Low` for simple tasks
- Batch multiple questions about the same image
- Cache analysis results

## Best Practices

### 1. Provide Context

```csharp
// ✓ Good - specific question
await agent.SendMessageAsync(
    "Is this medical scan showing any abnormalities?",
    imageUrl: scanUrl
);

// ✗ Poor - vague question
await agent.SendMessageAsync(
    "What is this?",
    imageUrl: scanUrl
);
```

### 2. Be Specific

```csharp
// ✓ Good - clear requirements
await agent.SendMessageAsync(
    "Count the number of red cars in this parking lot",
    imageUrl: parkingLotUrl
);

// ✗ Poor - ambiguous
await agent.SendMessageAsync(
    "Tell me about the cars",
    imageUrl: parkingLotUrl
);
```

### 3. Use Appropriate Detail Level

```csharp
// For text extraction - use high detail
await agent.SendMessageAsync(
    "Extract text",
    imageUrl: documentUrl,
    visionDetail: VisionDetail.High
);

// For general description - use auto or low
await agent.SendMessageAsync(
    "What's the overall mood?",
    imageUrl: photoUrl,
    visionDetail: VisionDetail.Auto
);
```

### 4. Handle Errors

```csharp
try
{
    var response = await agent.SendMessageAsync(
        "Analyze this",
        imageUrl: imageUrl
    );
}
catch (VisionNotSupportedException)
{
    Debug.LogError("Current model doesn't support vision");
    // Switch to vision-capable model
}
catch (ImageTooLargeException ex)
{
    Debug.LogError($"Image too large: {ex.Message}");
    // Resize image
}
```

## Limitations

- **File Size**: Maximum 20MB per image
- **Image Formats**: JPEG, PNG, GIF, WebP
- **Resolution**: Optimal is 2048x2048 or smaller
- **Multiple Images**: Some models limit the number of images per request
- **Content Restrictions**: No inappropriate content

## Troubleshooting

### Image Not Loading

```csharp
// Verify image URL is accessible
if (!await IsImageAccessible(imageUrl))
{
    Debug.LogError("Image URL is not accessible");
}

// Check image format
if (!IsValidImageFormat(imageBytes))
{
    Debug.LogError("Invalid image format");
}
```

### Low Quality Results

```csharp
// Use higher detail level
visionDetail: VisionDetail.High

// Provide more context
await agent.SendMessageAsync(
    "This is a medical X-ray. Please analyze it for any abnormalities.",
    imageUrl: xrayUrl
);

// Use better quality images
var optimizedImage = UpscaleImage(originalImage);
```

## Next Steps

- Learn about [Image Output (Generation)](image-output.md)
- Configure [Audio Setup](../essentials/configuration/audio-setup.md)
- Explore [Agent Events](../events/README.md)
