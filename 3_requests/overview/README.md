---
icon: store
---

# Requests Overview

AI Dev Kit provides a **Fluent API** that makes it incredibly easy to generate AI content with just a few lines of code.

## What is the Fluent API?

The Fluent API is a set of extension methods that let you chain configuration calls in a natural, readable way:

```csharp
string response = await "Explain quantum computing"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

## Key Concepts

### 1. Request Objects

All `.GENXxx()` methods create **request objects** that configure your AI generation task:

```csharp
// This creates a ChatCompletionRequest, but doesn't send anything yet
ChatCompletionRequest request = "Hello, AI!".GENCompletion();
```

### 2. Method Chaining

You can chain multiple configuration methods before execution:

```csharp
var request = "Generate a logo"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024)
    .SetQuality(ImageQuality.HD);
```

### 3. Execution

Nothing happens until you call `.ExecuteAsync()`:

```csharp
// NOW the request is sent to the AI provider
Texture2D image = await request.ExecuteAsync();
```

## Available Request Types

### Text Generation

```csharp
// Chat Completions API
"Explain AI".GENCompletion()

// Responses API (most capable)
"Write a story".GENResponse()

// Code Generation
"Create a C# quicksort".GENCode()

// Structured Output (JSON)
"User profile for John".GENStruct<UserProfile>()
```

### Image Generation

```csharp
// Text to Image
"Cyberpunk city".GENImage()

// Image Inpainting/Editing
texture.GENInpaint("Add a red car")

// Image to Image
texture.ImageToImage("Make it watercolor")
```

### Audio Generation

```csharp
// Text to Speech
"Welcome back!".GENSpeech()

// Speech to Text
audioClip.GENTranscript()

// Speech Translation (to English)
audioClip.GENTranslation()

// Sound Effects
"Dog barking".GENSoundEffect()

// Voice Change
audioClip.GENVoiceChange()

// Audio Isolation
audioClip.GENAudioIsolation()
```

### Video Generation

```csharp
// Text to Video
"Ocean waves".GENVideo()

// Image to Video
texture.GENVideo()
```

### Embeddings & Moderation

```csharp
// Text Embedding (single)
"Hello world".GENEmbed()

// Batch Embedding
new[] { "Text 1", "Text 2" }.GENEmbed()

// Content Moderation
"Check this text".GENModeration(safetySettings)
```

### File Operations

```csharp
// Upload File
Api.OpenAI.UploadFile(file)

// Upload Image
Api.OpenAI.UploadImage(texture)

// Upload Audio
Api.OpenAI.UploadAudio(audioClip)

// Upload Screenshot
Api.OpenAI.UploadScreenshot()

// Download File
Api.OpenAI.DownloadFile(fileId)

// Delete File
Api.OpenAI.DeleteFile(fileId)

// List Files
Api.OpenAI.ListFiles()
```

### Model Operations

```csharp
// Get Model
Api.OpenAI.GetModel("gpt-4o")

// List Models
Api.OpenAI.ListModels()

// Get Custom Model
Api.OpenAI.GetCustomModel("ft:gpt-4o:...")

// List Custom Models
Api.OpenAI.ListCustomModels()

// Delete Model
Api.OpenAI.DeleteModel("ft:gpt-4o:...")

// Fine-tune Model
model.FineTuneModel(trainingFileId)
```

### Voice Operations

```csharp
// Get Voice
Api.ElevenLabs.GetVoice("rachel")

// List Voices
Api.ElevenLabs.ListVoices()

// List Custom Voices
Api.ElevenLabs.ListCustomVoices()
```

### Utilities

```csharp
// Tokenize
"Hello world".Tokenize()

// Count Tokens
"Long text...".CountTokens()

// Get Credits (billing info)
Api.OpenAI.GetCredits()
```

## Next Steps

- [Fluent API Pattern](fluent-api-pattern.md) - Deep dive into the pattern
- [Request vs Execute](request-vs-execute.md) - Understanding the execution model
- [Provider Query Types](provider-query-types.md) - Advanced query options
