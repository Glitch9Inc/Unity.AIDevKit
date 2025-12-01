# Generating Responses

Send messages to your agent and handle the responses.

## Basic Response Generation

### Simple Request

```csharp
// Send text message
Response response = await agent.SendAsync("Hello! How are you?");
Debug.Log(response.Text);
```

### With Streaming

```csharp
agent.Stream = true;
agent.onTextDelta.AddListener(delta => Debug.Log(delta));

await agent.SendAsync("Tell me a story");
```

## Response Types

### Text Messages

```csharp
// Simple text
await agent.SendAsync("What is AI?");

// With role
await agent.SendAsync("Explain quantum computing", Role.User);

// System message
await agent.SendAsync("You are an expert physicist", Role.System);
```

### Audio Input

```csharp
// Send audio clip
AudioClip recording = await audioRecorder.RecordAsync();
await agent.SendAsync(recording);
```

### File Attachments

```csharp
// Send image
Texture2D image = LoadImage();
await agent.SendAsync("Describe this image", image);

// Send document
string filePath = "document.pdf";
await agent.SendAsync("Summarize this document", filePath);
```

### Multi-Modal Input

```csharp
// Text + Image
await agent.SendAsync(
    text: "What's in this image?",
    attachments: new[] { imageTexture }
);

// Text + Audio
await agent.SendAsync(
    text: "Context: Customer support call",
    audio: audioClip
);
```

## Response Handling

### Non-Streaming

```csharp
Response response = await agent.SendAsync("Hello");

// Access response data
Debug.Log($"Text: {response.Text}");
Debug.Log($"Role: {response.Role}");
Debug.Log($"Tool Calls: {response.ToolCalls?.Count}");
Debug.Log($"Tokens Used: {response.Usage.TotalTokens}");
```

### Streaming

```csharp
agent.Stream = true;

// Subscribe to events
agent.onResponseStarted.AddListener(r => Debug.Log("Started"));
agent.onTextDelta.AddListener(delta => AppendText(delta));
agent.onResponseCompleted.AddListener(r => Debug.Log("Completed"));

// Send message
await agent.SendAsync("Write a poem");
```

### Error Handling

```csharp
try
{
    await agent.SendAsync("Hello");
}
catch (RateLimitException ex)
{
    Debug.LogWarning("Rate limit exceeded");
}
catch (AuthenticationException ex)
{
    Debug.LogError("Invalid API key");
}
catch (Exception ex)
{
    Debug.LogError($"Error: {ex.Message}");
}
```

## Advanced Features

### Cancellation

```csharp
// Start request
var cts = new CancellationTokenSource();
var task = agent.SendAsync("Long response...", cts.Token);

// Cancel after delay
await UniTask.Delay(TimeSpan.FromSeconds(5));
cts.Cancel();
```

### Retry Logic

```csharp
int maxRetries = 3;
for (int i = 0; i < maxRetries; i++)
{
    try
    {
        await agent.SendAsync("Hello");
        break;
    }
    catch (Exception ex)
    {
        if (i == maxRetries - 1) throw;
        await UniTask.Delay(1000 * (i + 1));
    }
}
```

### Response Caching

```csharp
Dictionary<string, Response> cache = new();

async Task<Response> GetCachedResponse(string message)
{
    if (cache.TryGetValue(message, out var cached))
        return cached;
    
    var response = await agent.SendAsync(message);
    cache[message] = response;
    return response;
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ResponseHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.Stream = true;
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        agent.onError.AddListener(OnError);
    }
    
    public async void SendMessage(string text)
    {
        try
        {
            await agent.SendAsync(text);
        }
        catch (Exception ex)
        {
            Debug.LogError($"Send failed: {ex.Message}");
        }
    }
    
    void OnResponseStarted(Response response)
    {
        Debug.Log("Response started");
    }
    
    void OnTextDelta(string delta)
    {
        Debug.Log($"Delta: {delta}");
    }
    
    void OnResponseCompleted(Response response)
    {
        Debug.Log($"Completed. Tokens: {response.Usage.TotalTokens}");
    }
    
    void OnError(string error)
    {
        Debug.LogError($"Error: {error}");
    }
}
```

## Next Steps

- [Text Messages](text-messages.md) - Text message details
- [Streaming vs Non-Streaming](streaming.md) - Choose response mode
- [File Attachments](file-attachments.md) - Work with files
- [Audio Input](audio-input.md) - Voice input guide
- [Canceling Requests](canceling.md) - Cancel ongoing requests
