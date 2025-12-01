# Fluent API Pattern

The Fluent API pattern in AI Dev Kit follows a consistent, beginner-friendly design that makes AI integration feel natural.

## The Pattern

All generative AI requests follow this structure:

```
[Input].GENXxx().SetYyy(...).ExecuteAsync()
```

### 1. Input (Entry Point)

The input can be various types:

```csharp
// String
"Hello, AI!".GENCompletion()

// Texture2D
texture.GENInpaint("Add clouds")

// AudioClip
audioClip.GENTranscript()

// Prompt object
prompt.GENResponse()

// Message object
message.GENCompletion()

// ConversationItem
conversationItem.GENResponse()
```

### 2. Request Factory (`.GENXxx()`)

The `.GENXxx()` methods create **strongly-typed request objects**:

```csharp
ChatCompletionRequest request = "Hello".GENCompletion();
ImageGenerationRequest request = "Cat".GENImage();
SpeechGenerationRequest request = "Hi".GENSpeech();
```

**Important:** These methods do **NOT** perform any I/O or network calls. They only create configuration objects.

### 3. Configuration (`.SetYyy()`)

Chain configuration methods to customize the request:

```csharp
var request = "Explain AI"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)           // Model selection
    .SetTemperature(0.7f)                  // Creativity
    .SetMaxTokens(500)                     // Response length
    .SetSystemMessage("You are helpful"); // Context
```

### 4. Execution (`.ExecuteAsync()`)

**This is when the actual API call happens:**

```csharp
// Network request is sent NOW
string result = await request.ExecuteAsync();
```

## Why This Pattern?

### ✅ Readable

```csharp
// Clear, self-documenting code
var response = await "Explain photosynthesis"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

### ✅ Discoverable

Type a dot after any string, texture, or audio clip and IntelliSense shows you all available `.GENXxx()` methods.

### ✅ Type-Safe

```csharp
// Compile-time type checking
Texture2D image = await "Cat".GENImage().ExecuteAsync();
AudioClip audio = await "Hello".GENSpeech().ExecuteAsync();
string text = await "Hi".GENCompletion().ExecuteAsync();
```

### ✅ Flexible

You can store and reuse request objects:

```csharp
// Create request
var request = "Explain {topic}"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o);

// Execute multiple times with different inputs
string ai = await request.SetInput("AI").ExecuteAsync();
string ml = await request.SetInput("ML").ExecuteAsync();
```

## Common Patterns

### Pattern 1: One-liner

For quick, simple requests:

```csharp
string response = await "Hello".GENCompletion().ExecuteAsync();
```

### Pattern 2: Stored Configuration

When you need to reuse settings:

```csharp
var imageRequest = "placeholder"
    .GENImage()
    .SetModel(OpenAIModel.DallE3)
    .SetSize(ImageSize._1024x1024)
    .SetQuality(ImageQuality.HD);

Texture2D logo = await imageRequest.SetPrompt("Company logo").ExecuteAsync();
Texture2D banner = await imageRequest.SetPrompt("Hero banner").ExecuteAsync();
```

### Pattern 3: Async/Await

Always use `await` or `.Forget()`:

```csharp
// ✅ Correct - await the result
string text = await "Hello".GENCompletion().ExecuteAsync();

// ✅ Correct - fire and forget
"Hello".GENCompletion().ExecuteAsync().Forget();

// ❌ Wrong - compiler warning
"Hello".GENCompletion().ExecuteAsync();
```

### Pattern 4: Error Handling

Wrap in try-catch for robust error handling:

```csharp
try
{
    var result = await "Hello".GENCompletion().ExecuteAsync();
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

## Method Naming Convention

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `GEN` | Generate/Create content | `GENCompletion`, `GENImage`, `GENSpeech` |
| `Upload` | Upload files | `UploadFile`, `UploadImage`, `UploadAudio` |
| `Download` | Download files | `DownloadFile` |
| `List` | List resources | `ListModels`, `ListVoices`, `ListFiles` |
| `Get` | Retrieve single resource | `GetModel`, `GetVoice`, `GetCredits` |
| `Delete` | Remove resource | `DeleteFile`, `DeleteModel` |
| `Count` | Count tokens | `CountTokens` |
| `Tokenize` | Get token IDs | `Tokenize` |

## Extension Method Hosts

Different types can host different methods:

```csharp
// String extensions
"text".GENCompletion()
"text".GENResponse()
"text".GENCode()
"text".GENImage()
"text".GENSpeech()
"text".GENEmbed()
"text".Tokenize()
"text".CountTokens()

// Texture2D extensions
texture.GENInpaint("instruction")
texture.ImageToImage("instruction")
texture.GENVideo()

// AudioClip extensions
audioClip.GENTranscript()
audioClip.GENTranslation()
audioClip.GENVoiceChange()
audioClip.GENAudioIsolation()

// Api enum extensions
Api.OpenAI.UploadFile(file)
Api.OpenAI.ListModels()
Api.OpenAI.GetModel("gpt-4o")
Api.OpenAI.GetCredits()

// Model extensions
model.FineTuneModel(trainingFileId)

// Message extensions
message.GENCompletion()

// Prompt extensions
prompt.GENCompletion()
prompt.GENResponse()
prompt.GENCode()
```

## Next Steps

- [Request vs Execute](request-vs-execute.md) - Understanding when API calls happen
- [Text Generation](../text/README.md) - Text generation examples
- [Image Generation](../image/README.md) - Image generation examples
