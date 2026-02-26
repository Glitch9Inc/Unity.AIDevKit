---
icon: arrows-left-right-to-line
---

# Modalities (IO)

Enable multimodal interactions with your agent through image and voice capabilities.

## Overview

AI Dev Kit agents support multiple input and output modalities, allowing for rich, natural interactions:

- **Image Input (Vision)** - Analyze and understand images
- **Image Output (Generation)** - Generate images from text descriptions
- **Voice Input (Transcription)** - Convert speech to text
- **Voice Output (Speech)** - Generate natural speech from text

## Architecture

Each modality is managed by a dedicated controller:

```
Agent
 ├─ ImageController
 │   ├─ Vision (Input)
 │   └─ Generation (Output)
 └─ AudioController
     ├─ InputTranscriptionController (Voice Input)
     └─ OutputSpeechController (Voice Output)
```

## Supported Modalities

### Image Modalities

#### Vision (Input)

Send images to your agent for analysis:

```csharp
// Analyze an image
var response = await agent.SendMessageAsync(
    "What's in this image?",
    imageUrl: "https://example.com/image.jpg"
);
```

**Use Cases:**
- Image description
- Object detection
- OCR (text extraction)
- Visual understanding
- Scene analysis

#### Generation (Output)

Generate images from text descriptions:

```csharp
// Generate an image
var images = await agent.GenerateImagesAsync(
    "A serene mountain landscape at sunset"
);
```

**Use Cases:**
- Concept art
- Asset creation
- Visual prototyping
- Illustration

### Voice Modalities

#### Transcription (Input)

Convert speech to text:

```csharp
// Transcribe audio input
await agent.StartListeningAsync();
// Agent automatically transcribes and processes speech
```

**Use Cases:**
- Voice commands
- Dictation
- Speech-to-text
- Voice interfaces

#### Speech (Output)

Generate natural speech from text:

```csharp
// Speak text
await agent.SpeakAsync("Hello! How can I help you?");
```

**Use Cases:**
- Voice responses
- Narration
- Accessibility
- Audio feedback

## Multimodal Workflow

Combine multiple modalities for rich interactions:

```csharp
// 1. User speaks a question
await agent.StartListeningAsync();
// "What's in this image?" (Voice Input)

// 2. User provides an image
var response = await agent.SendMessageAsync(
    imageUrl: capturedImage
);
// Agent analyzes image (Image Input)

// 3. Agent responds with speech
// "I see a beautiful sunset over mountains." (Voice Output)

// 4. Agent generates a similar image
var newImage = await agent.GenerateImagesAsync(
    "Create a similar sunset scene"
);
// (Image Output)
```

## Configuration

Enable and configure modalities in AgentSettings:

```csharp
var config = new AgentConfiguration
{
    // Image capabilities
    SupportsVision = true,
    SupportsImageGeneration = true,
    ImageGenerationProvider = AiProvider.OpenAI,
    
    // Voice capabilities
    EnableVoiceInput = true,
    EnableVoiceOutput = true,
    VoiceId = "alloy",
    
    // Audio settings
    InputAudioFormat = AudioFormat.Wav,
    OutputAudioFormat = AudioFormat.Mp3,
};

Agent agent = new Agent(config);
```

## Provider Support

Different providers support different modalities:

| Provider | Vision | Image Gen | Transcription | Speech |
|----------|--------|-----------|---------------|--------|
| OpenAI | ✅ GPT-4V | ✅ DALL-E | ✅ Whisper | ✅ TTS |
| Google | ✅ Gemini | ❌ | ❌ | ❌ |
| Anthropic | ✅ Claude | ❌ | ❌ | ❌ |
| Stability | ❌ | ✅ SDXL | ❌ | ❌ |
| ElevenLabs | ❌ | ❌ | ❌ | ✅ TTS |

## Events

Subscribe to modality-specific events:

```csharp
// Image events
agent.OnImageGenerated += (images) => {
    Debug.Log($"Generated {images.Count} images");
};

agent.OnVisionAnalyzed += (result) => {
    Debug.Log($"Vision: {result.Description}");
};

// Voice events
agent.OnTranscriptionCompleted += (text) => {
    Debug.Log($"Heard: {text}");
};

agent.OnSpeechStarted += () => {
    Debug.Log("Agent speaking...");
};

agent.OnSpeechCompleted += () => {
    Debug.Log("Speech finished");
};
```

## Performance Considerations

### Image Processing

```csharp
// Optimize image size before sending
var optimizedImage = ImageUtility.ResizeForVision(
    originalImage,
    maxWidth: 2048
);

// Use appropriate detail level
var response = await agent.SendMessageAsync(
    "Analyze this",
    imageUrl: optimizedImage,
    visionDetail: VisionDetail.Auto // or Low, High
);
```

### Audio Streaming

```csharp
// Enable streaming for real-time transcription
agent.Configuration.EnableAudioStreaming = true;

// Stream audio chunks
agent.OnAudioChunkReceived += (chunk) => {
    // Process audio in real-time
};
```

## Best Practices

### 1. Image Input

```csharp
// ✓ Provide context with images
await agent.SendMessageAsync(
    "What type of architecture is shown in this building?",
    imageUrl: buildingPhoto
);

// ✗ Vague prompts
await agent.SendMessageAsync(
    "What's this?",
    imageUrl: buildingPhoto
);
```

### 2. Image Generation

```csharp
// ✓ Detailed, specific prompts
await agent.GenerateImagesAsync(
    "A photorealistic portrait of a warrior, " +
    "medieval armor, sunset lighting, cinematic"
);

// ✗ Generic prompts
await agent.GenerateImagesAsync("a person");
```

### 3. Voice Input

```csharp
// ✓ Handle transcription errors
agent.OnTranscriptionFailed += (error) => {
    ShowMessage("Could not understand. Please try again.");
};
```

### 4. Voice Output

```csharp
// ✓ Provide audio feedback
agent.OnSpeechStarted += () => {
    ShowSpeakingIndicator();
};

agent.OnSpeechCompleted += () => {
    HideSpeakingIndicator();
};
```

## Troubleshooting

### Vision Not Working

```csharp
// Check if model supports vision
if (!agent.CurrentModel.SupportsVision)
{
    Debug.LogError("Current model does not support vision");
    // Switch to vision-capable model
    agent.SetModel("gpt-4o");
}
```

### Audio Issues

```csharp
// Check microphone permissions
if (!Microphone.devices.Any())
{
    Debug.LogError("No microphone detected");
}

// Verify audio configuration
Debug.Log($"Audio format: {agent.Configuration.InputAudioFormat}");
Debug.Log($"Sample rate: {agent.Configuration.AudioSampleRate}");
```

## Learn More

Explore each modality in detail:

- **[Image Input (Vision)](image-input.md)** - Analyze images with AI
- **[Image Output (Generation)](image-output.md)** - Generate images from text
- **[Voice Input (Transcription)](voice-input.md)** - Convert speech to text
- **[Voice Output (Speech)](voice-output.md)** - Generate natural speech

## Related Topics

- [Audio Configuration](../essentials/configuration/audio-setup.md)
- [Response Generation](../essentials/responses/README.md)
- [Agent Events](../events/README.md)
