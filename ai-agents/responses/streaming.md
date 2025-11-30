# Streaming vs Non-Streaming

Choose between streaming and non-streaming response modes.

## Overview

**Streaming Mode:**

- Responses arrive token-by-token in real-time
- Better user experience (immediate feedback)
- More complex to handle

**Non-Streaming Mode:**

- Full response arrives at once
- Simpler to implement
- User waits for complete response

## Enabling Streaming

### AgentBehaviour

```csharp
// In Inspector or code
agent.Stream = true;

// Subscribe to text deltas
agent.onTextDelta.AddListener(OnTextDelta);

void OnTextDelta(string delta)
{
    // Append delta to UI
    chatText.text += delta;
}
```

### Agent (Direct)

```csharp
// Create agent with streaming enabled
var hooks = new AgentHooks
{
    TextDelta = (delta) => Debug.Log(delta)
};

var agent = new Agent(settings, behaviour, hooks: hooks);
```

## Non-Streaming Mode

### Basic Usage

```csharp
agent.Stream = false;

// Wait for complete response
Response response = await agent.SendAsync("Hello!");

// Display full response
chatText.text = response.Text;
```

### With Loading Indicator

```csharp
public async void SendNonStreaming(string message)
{
    loadingIndicator.SetActive(true);
    
    try
    {
        Response response = await agent.SendAsync(message);
        chatText.text = response.Text;
    }
    finally
    {
        loadingIndicator.SetActive(false);
    }
}
```

## Streaming Mode

### Real-time Display

```csharp
agent.Stream = true;

public class StreamingChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text chatText;
    
    private string currentResponse = "";
    
    void Start()
    {
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseStarted(Response response)
    {
        currentResponse = "";
        chatText.text = "Assistant: ";
    }
    
    void OnTextDelta(string delta)
    {
        currentResponse += delta;
        chatText.text = $"Assistant: {currentResponse}";
    }
    
    void OnResponseCompleted(Response response)
    {
        Debug.Log("Streaming complete");
    }
}
```

### Typewriter Effect

```csharp
public class TypewriterChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text chatText;
    
    private Queue<string> deltaQueue = new();
    private bool isTyping = false;
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
    }
    
    void OnTextDelta(string delta)
    {
        deltaQueue.Enqueue(delta);
        
        if (!isTyping)
        {
            StartCoroutine(TypewriterEffect());
        }
    }
    
    IEnumerator TypewriterEffect()
    {
        isTyping = true;
        
        while (deltaQueue.Count > 0)
        {
            string delta = deltaQueue.Dequeue();
            chatText.text += delta;
            
            // Delay between characters
            yield return new WaitForSeconds(0.03f);
        }
        
        isTyping = false;
    }
}
```

## Performance Comparison

### Token Arrival Time

```csharp
// Non-streaming: Wait 5-10 seconds for full response
agent.Stream = false;
var startTime = DateTime.Now;
Response response = await agent.SendAsync("Write a story");
var duration = DateTime.Now - startTime;
Debug.Log($"Received after {duration.TotalSeconds}s");

// Streaming: First token in ~500ms
agent.Stream = true;
agent.onTextDelta += (delta) =>
{
    var duration = DateTime.Now - startTime;
    Debug.Log($"First token after {duration.TotalMilliseconds}ms");
};
```

### User Perception

```csharp
// Non-streaming
// User waits: ⏳⏳⏳⏳⏳ → Full response appears
// Perceived as slow

// Streaming
// User sees: T → Th → The → The q → The qui → ...
// Perceived as fast
```

## Handling Both Modes

### Mode Toggle

```csharp
public class FlexibleChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text chatText;
    [SerializeField] private Toggle streamingToggle;
    
    private string currentResponse = "";
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        
        streamingToggle.onValueChanged.AddListener(OnStreamingToggled);
    }
    
    void OnStreamingToggled(bool streaming)
    {
        agent.Stream = streaming;
        Debug.Log($"Streaming: {streaming}");
    }
    
    public async void SendMessage(string message)
    {
        currentResponse = "";
        
        if (agent.Stream)
        {
            // Streaming: text appears via onTextDelta
            await agent.SendAsync(message);
        }
        else
        {
            // Non-streaming: wait for full response
            Response response = await agent.SendAsync(message);
            chatText.text = response.Text;
        }
    }
    
    void OnTextDelta(string delta)
    {
        if (agent.Stream)
        {
            currentResponse += delta;
            chatText.text = currentResponse;
        }
    }
    
    void OnResponseCompleted(Response response)
    {
        if (agent.Stream)
        {
            Debug.Log("Streaming completed");
        }
    }
}
```

## Streaming with Tool Calls

### Handle Interruptions

```csharp
public class StreamingWithTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text chatText;
    
    private string currentResponse = "";
    private bool isToolCall = false;
    
    void Start()
    {
        agent.Stream = true;
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onToolCallStarted.AddListener(OnToolCallStarted);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseStarted(Response response)
    {
        currentResponse = "";
        isToolCall = false;
    }
    
    void OnTextDelta(string delta)
    {
        if (!isToolCall)
        {
            currentResponse += delta;
            chatText.text = currentResponse;
        }
    }
    
    void OnToolCallStarted(ToolCall toolCall)
    {
        isToolCall = true;
        chatText.text += $"\n<i>[Using tool: {toolCall.Function.Name}]</i>\n";
    }
    
    void OnToolCallCompleted(ToolCall toolCall, string output)
    {
        isToolCall = false;
        chatText.text += $"<i>[Tool completed]</i>\n";
    }
    
    void OnResponseCompleted(Response response)
    {
        Debug.Log("Full response completed");
    }
}
```

## Error Handling

### Streaming Errors

```csharp
void Start()
{
    agent.onTextDelta.AddListener(OnTextDelta);
    agent.onError.AddListener(OnError);
    agent.onResponseCompleted.AddListener(OnResponseCompleted);
}

void OnTextDelta(string delta)
{
    chatText.text += delta;
}

void OnError(string error)
{
    // Streaming was interrupted
    chatText.text += $"\n<color=red>[Error: {error}]</color>";
    Debug.LogError($"Streaming error: {error}");
}

void OnResponseCompleted(Response response)
{
    // Check if completed successfully
    if (response.FinishReason == FinishReason.Stop)
    {
        Debug.Log("✓ Streaming completed normally");
    }
    else
    {
        Debug.LogWarning($"Streaming ended: {response.FinishReason}");
    }
}
```

## Buffer Management

### Accumulate Deltas

```csharp
public class StreamingBuffer : MonoBehaviour
{
    private StringBuilder responseBuffer = new();
    
    void Start()
    {
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseStarted(Response response)
    {
        responseBuffer.Clear();
    }
    
    void OnTextDelta(string delta)
    {
        responseBuffer.Append(delta);
        UpdateUI();
    }
    
    void OnResponseCompleted(Response response)
    {
        // Final text
        string finalText = responseBuffer.ToString();
        
        // Verify matches response
        if (finalText != response.Text)
        {
            Debug.LogWarning("Buffer mismatch with response!");
        }
    }
    
    void UpdateUI()
    {
        chatText.text = responseBuffer.ToString();
    }
}
```

## Performance Tips

### 1. Throttle UI Updates

```csharp
private float lastUpdateTime = 0f;
private const float updateInterval = 0.1f; // Update every 100ms

void OnTextDelta(string delta)
{
    currentResponse += delta;
    
    // Throttle UI updates
    if (Time.time - lastUpdateTime >= updateInterval)
    {
        chatText.text = currentResponse;
        lastUpdateTime = Time.time;
    }
}

void OnResponseCompleted(Response response)
{
    // Final update
    chatText.text = response.Text;
}
```

### 2. Use StringBuilder

```csharp
private StringBuilder responseBuilder = new StringBuilder(1000);

void OnTextDelta(string delta)
{
    // StringBuilder is more efficient than string concatenation
    responseBuilder.Append(delta);
    chatText.text = responseBuilder.ToString();
}
```

### 3. Batch Small Deltas

```csharp
private List<string> deltaBatch = new();
private const int batchSize = 5;

void OnTextDelta(string delta)
{
    deltaBatch.Add(delta);
    
    if (deltaBatch.Count >= batchSize)
    {
        FlushBatch();
    }
}

void FlushBatch()
{
    string combined = string.Join("", deltaBatch);
    chatText.text += combined;
    deltaBatch.Clear();
}

void OnResponseCompleted(Response response)
{
    // Flush remaining
    if (deltaBatch.Count > 0)
    {
        FlushBatch();
    }
}
```

## Best Practices

### When to Use Streaming

✅ **Use streaming when:**

- User experience is priority
- Responses are long (> 100 tokens)
- Real-time feedback is important
- Building chat interfaces

❌ **Avoid streaming when:**

- Processing response programmatically
- Need complete response for parsing
- Simple short responses
- Background processing

### Example Decision

```csharp
public async UniTask<Response> SendSmart(string message)
{
    // Estimate response length
    bool isLongResponse = message.Contains("write") || 
                         message.Contains("explain") ||
                         message.Contains("describe");
    
    // Enable streaming for long responses
    agent.Stream = isLongResponse;
    
    return await agent.SendAsync(message);
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using TMPro;
using System.Text;

public class StreamingExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text chatText;
    [SerializeField] private Toggle streamingToggle;
    [SerializeField] private GameObject loadingIndicator;
    
    private StringBuilder currentResponse = new();
    
    void Start()
    {
        // Setup events
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        agent.onError.AddListener(OnError);
        
        // Toggle streaming
        streamingToggle.onValueChanged.AddListener(enabled =>
        {
            agent.Stream = enabled;
            Debug.Log($"Streaming: {enabled}");
        });
    }
    
    public async void SendMessage(string message)
    {
        currentResponse.Clear();
        
        if (agent.Stream)
        {
            // Streaming mode: text appears via events
            chatText.text = "Assistant: ";
            await agent.SendAsync(message);
        }
        else
        {
            // Non-streaming: show loading
            loadingIndicator.SetActive(true);
            
            try
            {
                Response response = await agent.SendAsync(message);
                chatText.text = $"Assistant: {response.Text}";
            }
            finally
            {
                loadingIndicator.SetActive(false);
            }
        }
    }
    
    void OnResponseStarted(Response response)
    {
        if (agent.Stream)
        {
            currentResponse.Clear();
            Debug.Log("Streaming started...");
        }
    }
    
    void OnTextDelta(string delta)
    {
        if (agent.Stream)
        {
            currentResponse.Append(delta);
            chatText.text = $"Assistant: {currentResponse}";
        }
    }
    
    void OnResponseCompleted(Response response)
    {
        Debug.Log($"✓ Response completed ({response.Usage.TotalTokens} tokens)");
        
        if (agent.Stream)
        {
            // Verify buffer
            if (currentResponse.ToString() != response.Text)
            {
                Debug.LogWarning("Stream buffer mismatch!");
                chatText.text = $"Assistant: {response.Text}";
            }
        }
    }
    
    void OnError(string error)
    {
        Debug.LogError($"Error: {error}");
        chatText.text += $"\n<color=red>Error: {error}</color>";
    }
}
```

## Next Steps

- [Text Messages](text-messages.md)
- [File Attachments](file-attachments.md)
- [Canceling Requests](canceling.md)
