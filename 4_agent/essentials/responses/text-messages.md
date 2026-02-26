---
icon: message
---

# Text Messages

Send text messages and receive text responses from your agent.

## Overview

Text messaging is the most common way to interact with agents:

- Simple string messages
- Multi-line messages
- Context-aware responses
- Async/await pattern

## Basic Text Messages

### Send and Await

```csharp
// Send message and wait for response
Response response = await agent.SendAsync("What is Unity?");

// Access response text
Debug.Log(response.Text);
```

### Fire and Forget

```csharp
// Send without waiting
_ = agent.SendAsync("Hello!");

// Use event to get response
agent.onResponseCompleted.AddListener(OnResponseReceived);

void OnResponseReceived(Response response)
{
    Debug.Log(response.Text);
}
```

## Multi-line Messages

### Using String Interpolation

```csharp
string code = @"
public class Player : MonoBehaviour
{
    void Start() {
        // Code here
    }
}
";

await agent.SendAsync($@"
Review this code:
{code}

What improvements would you suggest?
");
```

### Using StringBuilder

```csharp
var message = new StringBuilder();
message.AppendLine("I need help with:");
message.AppendLine("1. Player movement");
message.AppendLine("2. Camera follow");
message.AppendLine("3. Input handling");

await agent.SendAsync(message.ToString());
```

## Response Object

### Response Structure

```csharp
public class Response
{
    public string Id { get; }
    public string Text { get; }
    public Role Role { get; }
    public List<ToolCall> ToolCalls { get; }
    public Usage Usage { get; }
    public FinishReason FinishReason { get; }
    public DateTime Timestamp { get; }
}
```

### Access Response Data

```csharp
Response response = await agent.SendAsync("Hello!");

// Text content
string text = response.Text;

// Token usage
int promptTokens = response.Usage.PromptTokens;
int completionTokens = response.Usage.CompletionTokens;
int totalTokens = response.Usage.TotalTokens;

// Finish reason
FinishReason reason = response.FinishReason;
// Possible values: Stop, Length, ToolCalls, ContentFilter

Debug.Log($"Response: {text}");
Debug.Log($"Tokens: {totalTokens}");
Debug.Log($"Reason: {reason}");
```

## Error Handling

### Try-Catch Pattern

```csharp
try
{
    Response response = await agent.SendAsync("Hello!");
    Debug.Log(response.Text);
}
catch (ApiException ex) when (ex.StatusCode == 401)
{
    Debug.LogError("Invalid API key");
}
catch (ApiException ex) when (ex.StatusCode == 429)
{
    Debug.LogError("Rate limit exceeded");
}
catch (ApiException ex) when (ex.StatusCode == 500)
{
    Debug.LogError("API server error");
}
catch (OperationCanceledException)
{
    Debug.Log("Request was cancelled");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

### With Retry Logic

```csharp
public async UniTask<Response> SendWithRetry(
    string message,
    int maxRetries = 3)
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            return await agent.SendAsync(message);
        }
        catch (ApiException ex) when (ex.StatusCode == 429)
        {
            // Rate limited, wait and retry
            int delay = (int)Math.Pow(2, i) * 1000; // Exponential backoff
            Debug.Log($"Rate limited, retrying in {delay}ms...");
            await UniTask.Delay(delay);
        }
        catch (Exception ex) when (i < maxRetries - 1)
        {
            Debug.LogWarning($"Attempt {i + 1} failed: {ex.Message}");
            await UniTask.Delay(1000);
        }
    }
    
    throw new Exception("Max retries exceeded");
}
```

## Message History

### Access Previous Messages

```csharp
// Get all messages
List<Message> messages = agent.Messages;

// Get last user message
Message lastUser = agent.Messages
    .LastOrDefault(m => m.Role == Role.User);

// Get last assistant message
Message lastAssistant = agent.LastMessage;
```

### Reference Previous Context

```csharp
// Agent automatically maintains context
await agent.SendAsync("What is Unity?");
// Response: "Unity is a game engine..."

await agent.SendAsync("What are its main features?");
// Response uses context from previous message
// "Unity's main features include..."
```

## Formatting Responses

### Display in UI

```csharp
public class ChatUI : MonoBehaviour
{
    [SerializeField] private TMP_Text chatText;
    [SerializeField] private AgentBehaviour agent;
    
    public async void SendMessage(string message)
    {
        // Show user message
        AppendMessage("You", message);
        
        // Get response
        Response response = await agent.SendAsync(message);
        
        // Show assistant message
        AppendMessage("Assistant", response.Text);
    }
    
    void AppendMessage(string sender, string message)
    {
        chatText.text += $"\n<b>{sender}:</b> {message}\n";
    }
}
```

### Markdown to Rich Text

```csharp
public string ConvertMarkdownToRichText(string markdown)
{
    // Convert **bold**
    markdown = Regex.Replace(markdown, @"\*\*(.*?)\*\*", "<b>$1</b>");
    
    // Convert *italic*
    markdown = Regex.Replace(markdown, @"\*(.*?)\*", "<i>$1</i>");
    
    // Convert `code`
    markdown = Regex.Replace(markdown, @"`(.*?)`", "<color=#00ff00>$1</color>");
    
    return markdown;
}
```

## Streaming Responses

### Enable Streaming

```csharp
// Enable in AgentBehaviour
agent.Stream = true;

// Listen for text deltas
agent.onTextDelta.AddListener(OnTextDelta);

void OnTextDelta(string delta)
{
    // Append delta to UI
    chatText.text += delta;
}
```

See [Streaming vs Non-Streaming](streaming.md) for detailed streaming documentation.

## Batch Messages

### Send Multiple Messages

```csharp
public async UniTask SendBatch(string[] messages)
{
    foreach (string message in messages)
    {
        Response response = await agent.SendAsync(message);
        Debug.Log($"Q: {message}");
        Debug.Log($"A: {response.Text}\n");
        
        // Small delay between messages
        await UniTask.Delay(100);
    }
}

// Usage
await SendBatch(new[]
{
    "What is Unity?",
    "What is C#?",
    "How do I make a game?"
});
```

### Parallel Processing (Not Recommended)

```csharp
// ⚠️ Not recommended: Agent maintains conversation state
// Parallel requests will interfere with each other

// Instead, use separate agent instances
public async UniTask SendParallel(string[] messages)
{
    var tasks = messages.Select(async message =>
    {
        // Create new agent for each message
        var tempAgent = new Agent(settings, behaviour);
        await tempAgent.InitializeAsync();
        return await tempAgent.SendAsync(message);
    });
    
    Response[] responses = await UniTask.WhenAll(tasks);
}
```

## Special Characters

### Handling Quotes

```csharp
// Use verbatim strings
string message = @"The code says ""Hello World""";

// Or escape quotes
string message2 = "The code says \"Hello World\"";

await agent.SendAsync(message);
```

### Handling Line Breaks

```csharp
// Use Environment.NewLine
string message = $"Line 1{Environment.NewLine}Line 2";

// Or verbatim string
string message2 = @"
Line 1
Line 2
Line 3
";

await agent.SendAsync(message);
```

## Message Validation

### Check Message Length

```csharp
public async UniTask<Response> SendValidated(string message)
{
    if (string.IsNullOrWhiteSpace(message))
    {
        throw new ArgumentException("Message cannot be empty");
    }
    
    if (message.Length > 10000)
    {
        throw new ArgumentException("Message too long (max 10000 chars)");
    }
    
    return await agent.SendAsync(message);
}
```

### Content Filtering

```csharp
public bool IsMessageSafe(string message)
{
    // Check for blocked words
    string[] blockedWords = { "spam", "inappropriate" };
    
    foreach (string word in blockedWords)
    {
        if (message.Contains(word, StringComparison.OrdinalIgnoreCase))
        {
            return false;
        }
    }
    
    return true;
}

public async void SendSafe(string message)
{
    if (!IsMessageSafe(message))
    {
        Debug.LogWarning("Message contains inappropriate content");
        return;
    }
    
    await agent.SendAsync(message);
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using TMPro;
using System;

public class TextMessaging : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField inputField;
    [SerializeField] private TMP_Text chatText;
    [SerializeField] private GameObject sendButton;
    
    void Start()
    {
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        agent.onError.AddListener(OnError);
    }
    
    public async void SendMessage()
    {
        string message = inputField.text.Trim();
        
        // Validate
        if (string.IsNullOrWhiteSpace(message))
        {
            return;
        }
        
        // Clear input
        inputField.text = "";
        inputField.interactable = false;
        sendButton.SetActive(false);
        
        // Show user message
        AppendMessage("You", message);
        
        try
        {
            // Send and get response
            Response response = await agent.SendAsync(message);
            
            // Show assistant message
            AppendMessage("Assistant", response.Text);
            
            // Log usage
            Debug.Log($"Tokens used: {response.Usage.TotalTokens}");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Error: {ex.Message}");
            AppendMessage("System", $"Error: {ex.Message}");
        }
        finally
        {
            inputField.interactable = true;
            sendButton.SetActive(true);
        }
    }
    
    void AppendMessage(string sender, string message)
    {
        string color = sender switch
        {
            "You" => "#4A90E2",
            "Assistant" => "#50C878",
            _ => "#999999"
        };
        
        chatText.text += $"\n<color={color}><b>{sender}:</b></color> {message}\n";
    }
    
    void OnResponseStarted(Response response)
    {
        Debug.Log("Response started...");
    }
    
    void OnResponseCompleted(Response response)
    {
        Debug.Log($"Response completed: {response.FinishReason}");
    }
    
    void OnError(string error)
    {
        AppendMessage("Error", error);
        inputField.interactable = true;
        sendButton.SetActive(true);
    }
}
```

## Next Steps

- [Streaming vs Non-Streaming](streaming.md)
- [File Attachments](file-attachments.md)
- [Audio Input](audio-input.md)
- [Canceling Requests](canceling.md)
