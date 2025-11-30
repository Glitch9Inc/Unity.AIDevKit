# File Attachments

Send files and images along with your messages.

## Overview

AI Dev Kit supports:

- **Images** (PNG, JPG, WebP, GIF)
- **Documents** (for File Search tool)
- **Code files** (for Code Interpreter)
- **Audio files** (for transcription)

## Image Attachments

### Send Image with Message

```csharp
// Load image
Texture2D image = LoadImage();

// Send with text
Response response = await agent.SendAsync(
    message: "What's in this image?",
    image: image
);

Debug.Log(response.Text);
```

### Image Only

```csharp
// Send image without text (uses default prompt)
Response response = await agent.SendAsync(
    message: "",
    image: image
);
```

### Multiple Images (Not Directly Supported)

```csharp
// Send multiple images by adding to message manually
var message = new Message
{
    Role = Role.User,
    Parts = new List<ContentPart>
    {
        new ContentPart { Type = "text", Text = "Compare these images:" },
        new ContentPart
        {
            Type = "image_url",
            ImageUrl = new ImageUrl
            {
                Url = ConvertToBase64(image1),
                Detail = "high"
            }
        },
        new ContentPart
        {
            Type = "image_url",
            ImageUrl = new ImageUrl
            {
                Url = ConvertToBase64(image2),
                Detail = "high"
            }
        }
    }
};

agent.Conversation.Messages.Add(message);
await agent.SendAsync("");
```

## Image Formats

### Convert Texture2D to Base64

```csharp
public string ConvertToBase64(Texture2D texture)
{
    byte[] imageBytes = texture.EncodeToPNG();
    string base64 = Convert.ToBase64String(imageBytes);
    return $"data:image/png;base64,{base64}";
}
```

### Load from File

```csharp
public Texture2D LoadImageFromFile(string path)
{
    byte[] bytes = File.ReadAllBytes(path);
    Texture2D texture = new Texture2D(2, 2);
    texture.LoadImage(bytes);
    return texture;
}

// Usage
Texture2D image = LoadImageFromFile("screenshot.png");
await agent.SendAsync("Analyze this", image);
```

### From Screenshot

```csharp
public async UniTask SendScreenshot(string question)
{
    // Capture screenshot
    Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();
    
    // Send to agent
    Response response = await agent.SendAsync(question, screenshot);
    
    Debug.Log(response.Text);
    
    // Cleanup
    Destroy(screenshot);
}
```

### From Camera

```csharp
public async UniTask AnalyzeCameraView(Camera camera)
{
    // Render camera to texture
    RenderTexture rt = new RenderTexture(512, 512, 24);
    camera.targetTexture = rt;
    camera.Render();
    
    // Convert to Texture2D
    RenderTexture.active = rt;
    Texture2D screenshot = new Texture2D(512, 512, TextureFormat.RGB24, false);
    screenshot.ReadPixels(new Rect(0, 0, 512, 512), 0, 0);
    screenshot.Apply();
    
    // Cleanup
    camera.targetTexture = null;
    RenderTexture.active = null;
    Destroy(rt);
    
    // Send to agent
    await agent.SendAsync("What do you see?", screenshot);
    
    Destroy(screenshot);
}
```

## Image Detail Levels

### High vs Low Detail

```csharp
// High detail (more tokens, better accuracy)
var highDetailPart = new ContentPart
{
    Type = "image_url",
    ImageUrl = new ImageUrl
    {
        Url = imageBase64,
        Detail = "high"  // ~765 tokens per image
    }
};

// Low detail (fewer tokens, faster)
var lowDetailPart = new ContentPart
{
    Type = "image_url",
    ImageUrl = new ImageUrl
    {
        Url = imageBase64,
        Detail = "low"  // ~85 tokens per image
    }
};
```

## Document Attachments

### Upload for File Search

```csharp
// Upload document to OpenAI
public async UniTask<string> UploadDocument(string filePath)
{
    byte[] fileBytes = File.ReadAllBytes(filePath);
    string fileName = Path.GetFileName(filePath);
    
    // Upload via OpenAI API
    var fileInfo = await openAIClient.Files.UploadFileAsync(
        file: fileBytes,
        fileName: fileName,
        purpose: "assistants"
    );
    
    return fileInfo.Id;
}

// Add to agent tools
agent.Settings.ToolDefinitions.Add(new FileSearchToolDefinition
{
    FileIds = new[] { fileId }
});
```

### Use with File Search Tool

```csharp
// Enable File Search tool
agent.Settings.EnableFileSearch = true;
agent.Settings.FileSearchFileIds = new[] { fileId };

// Ask questions about the document
await agent.SendAsync("What does the document say about pricing?");
```

## Code File Attachments

### Send Code for Analysis

```csharp
public async UniTask AnalyzeCodeFile(string filePath)
{
    string code = File.ReadAllText(filePath);
    string fileName = Path.GetFileName(filePath);
    
    string message = $@"
Analyze this {Path.GetExtension(filePath)} file:
File: {fileName}

```{Path.GetExtension(filePath).TrimStart('.')}
{code}
```

What improvements would you suggest?
";

    Response response = await agent.SendAsync(message);
    Debug.Log(response.Text);
}

```

### Use Code Interpreter

```csharp
// Enable Code Interpreter tool
agent.Settings.EnableCodeInterpreter = true;

// Upload code file
string fileId = await UploadDocument("script.py");

// Ask agent to execute it
await agent.SendAsync("Run this Python script and show me the output");
```

## Audio File Attachments

### Send Audio for Transcription

```csharp
public async UniTask<string> TranscribeAudioFile(string audioPath)
{
    // Load audio clip
    AudioClip clip = LoadAudioClip(audioPath);
    
    // Transcribe
    string transcription = await agent.TranscribeAsync(clip);
    
    Debug.Log($"Transcription: {transcription}");
    
    return transcription;
}
```

### Send Audio as Message

```csharp
AudioClip clip = LoadAudioClip("recording.wav");

// Send audio (will be transcribed automatically if enabled)
Response response = await agent.SendAsync(clip);

Debug.Log(response.Text);
```

## File Size Limits

### Check File Size

```csharp
public bool IsFileSizeValid(string filePath, long maxSizeBytes = 20_000_000)
{
    FileInfo fileInfo = new FileInfo(filePath);
    
    if (fileInfo.Length > maxSizeBytes)
    {
        Debug.LogError($"File too large: {fileInfo.Length} bytes (max: {maxSizeBytes})");
        return false;
    }
    
    return true;
}
```

### Compress Images

```csharp
public Texture2D CompressImage(Texture2D original, int maxDimension = 1024)
{
    if (original.width <= maxDimension && original.height <= maxDimension)
    {
        return original;
    }
    
    // Calculate new size maintaining aspect ratio
    float scale = Mathf.Min(
        (float)maxDimension / original.width,
        (float)maxDimension / original.height
    );
    
    int newWidth = Mathf.RoundToInt(original.width * scale);
    int newHeight = Mathf.RoundToInt(original.height * scale);
    
    // Resize
    RenderTexture rt = RenderTexture.GetTemporary(newWidth, newHeight);
    Graphics.Blit(original, rt);
    
    RenderTexture.active = rt;
    Texture2D compressed = new Texture2D(newWidth, newHeight);
    compressed.ReadPixels(new Rect(0, 0, newWidth, newHeight), 0, 0);
    compressed.Apply();
    
    RenderTexture.active = null;
    RenderTexture.ReleaseTemporary(rt);
    
    return compressed;
}
```

## Error Handling

### Invalid File Types

```csharp
public async UniTask<Response> SendImageSafe(string imagePath)
{
    string extension = Path.GetExtension(imagePath).ToLower();
    string[] validExtensions = { ".png", ".jpg", ".jpeg", ".webp", ".gif" };
    
    if (!validExtensions.Contains(extension))
    {
        throw new ArgumentException($"Invalid image format: {extension}");
    }
    
    Texture2D image = LoadImageFromFile(imagePath);
    return await agent.SendAsync("Analyze this image", image);
}
```

### File Not Found

```csharp
try
{
    Texture2D image = LoadImageFromFile(path);
    await agent.SendAsync("What's this?", image);
}
catch (FileNotFoundException)
{
    Debug.LogError($"Image not found: {path}");
}
catch (Exception ex)
{
    Debug.LogError($"Failed to load image: {ex.Message}");
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using System;
using System.IO;

public class FileAttachmentManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void AnalyzeScreenshot()
    {
        // Capture
        Texture2D screenshot = ScreenCapture.CaptureScreenshotAsTexture();
        
        // Compress if needed
        Texture2D compressed = CompressImage(screenshot, 1024);
        
        try
        {
            // Send to agent
            Response response = await agent.SendAsync(
                "Describe what you see in this screenshot. What is the user likely doing?",
                compressed
            );
            
            Debug.Log($"Analysis: {response.Text}");
        }
        finally
        {
            // Cleanup
            Destroy(screenshot);
            if (compressed != screenshot)
            {
                Destroy(compressed);
            }
        }
    }
    
    public async void AnalyzeImage(string imagePath)
    {
        if (!File.Exists(imagePath))
        {
            Debug.LogError($"File not found: {imagePath}");
            return;
        }
        
        // Validate
        if (!IsImageFile(imagePath))
        {
            Debug.LogError("Not a valid image file");
            return;
        }
        
        // Load
        Texture2D image = LoadImageFromFile(imagePath);
        
        try
        {
            // Send
            Response response = await agent.SendAsync(
                "What's in this image?",
                image
            );
            
            Debug.Log(response.Text);
        }
        finally
        {
            Destroy(image);
        }
    }
    
    bool IsImageFile(string path)
    {
        string ext = Path.GetExtension(path).ToLower();
        return ext == ".png" || ext == ".jpg" || ext == ".jpeg" || 
               ext == ".webp" || ext == ".gif";
    }
    
    Texture2D LoadImageFromFile(string path)
    {
        byte[] bytes = File.ReadAllBytes(path);
        Texture2D texture = new Texture2D(2, 2);
        texture.LoadImage(bytes);
        return texture;
    }
    
    Texture2D CompressImage(Texture2D original, int maxDimension)
    {
        if (original.width <= maxDimension && original.height <= maxDimension)
        {
            return original;
        }
        
        float scale = Mathf.Min(
            (float)maxDimension / original.width,
            (float)maxDimension / original.height
        );
        
        int newWidth = Mathf.RoundToInt(original.width * scale);
        int newHeight = Mathf.RoundToInt(original.height * scale);
        
        RenderTexture rt = RenderTexture.GetTemporary(newWidth, newHeight);
        Graphics.Blit(original, rt);
        
        RenderTexture.active = rt;
        Texture2D compressed = new Texture2D(newWidth, newHeight);
        compressed.ReadPixels(new Rect(0, 0, newWidth, newHeight), 0, 0);
        compressed.Apply();
        
        RenderTexture.active = null;
        RenderTexture.ReleaseTemporary(rt);
        
        return compressed;
    }
}
```

## Next Steps

- [Text Messages](text-messages.md)
- [Audio Input](audio-input.md)
- [File Search Tool](../tools/hosted-tools/file-search.md)
- [Code Interpreter](../tools/hosted-tools/code-interpreter.md)
