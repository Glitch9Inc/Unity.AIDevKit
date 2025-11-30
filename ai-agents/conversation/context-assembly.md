# Context Assembly

Understand how context is assembled for API requests.

## Overview

Context assembly is the process of:

1. Gathering conversation history
2. Applying memory settings
3. Adding system instructions
4. Including tool definitions
5. Formatting for the API

## Basic Context Flow

```
System Message → Memory Window → User Message → API Request
     ↓                ↓                ↓
Instructions    Last N messages   Current input
```

## System Instructions

### From AgentSettings

```csharp
// In AgentSettings
settings.Instructions = "You are a helpful Unity development assistant.";

// Automatically added as first message in context
```

### Runtime Override

```csharp
// Update instructions at runtime
agent.Instructions = "You are now a debugging expert.";

// Next API call uses new instructions
await agent.SendAsync("Help me debug this error");
```

### Dynamic Instructions

```csharp
public string GetDynamicInstructions()
{
    var context = new StringBuilder();
    context.AppendLine("You are a Unity assistant.");
    context.AppendLine($"Current scene: {SceneManager.GetActiveScene().name}");
    context.AppendLine($"Platform: {Application.platform}");
    context.AppendLine($"Time: {DateTime.Now:HH:mm}");
    
    return context.ToString();
}

// Update before each request
agent.Instructions = GetDynamicInstructions();
await agent.SendAsync("What should I work on?");
```

## Memory Settings

### Memory Window

```csharp
// In AgentSettings.Memory
memory.MaxConversationLength = 20;  // Keep last 20 messages

// Context assembly:
// 1. System message (always included)
// 2. Last 20 messages
// 3. Current user message
```

### Summary-based Memory

```csharp
memory.UseSummary = true;
memory.SummaryMessageCount = 50;  // Summarize every 50 messages

// Context assembly:
// 1. System message
// 2. Conversation summary
// 3. Last N messages (within window)
// 4. Current user message
```

### Memory Modes

```csharp
public enum ConversationMemoryMode
{
    None,          // No memory (stateless)
    Recent,        // Keep recent messages
    Summary,       // Use summarization
    Full           // Keep all messages
}

// Configure
memory.Mode = ConversationMemoryMode.Recent;
memory.MaxConversationLength = 10;
```

## Token Management

### Estimate Context Size

```csharp
public int EstimateContextTokens()
{
    int tokens = 0;
    
    // System message
    tokens += EstimateTokens(agent.Instructions);
    
    // Messages within window
    var messages = GetMessagesInWindow();
    foreach (var message in messages)
    {
        tokens += EstimateTokens(message.Content);
    }
    
    // Tool definitions
    tokens += agent.Tools.Sum(t => EstimateTokens(t.ToString()));
    
    return tokens;
}

int EstimateTokens(string text)
{
    // Rough estimation: 1 token ≈ 4 characters
    return text.Length / 4;
}
```

### Dynamic Context Trimming

```csharp
public async UniTask SendWithContextManagement(string message)
{
    // Check context size
    int estimatedTokens = EstimateContextTokens();
    int maxContextTokens = agent.MaxTokens ?? 4096;
    
    if (estimatedTokens > maxContextTokens * 0.8f)
    {
        // Trim conversation history
        agent.ClearConversation(keepLastN: 5);
        Debug.Log("Context trimmed to fit token limit");
    }
    
    await agent.SendAsync(message);
}
```

## Tool Context

### Tool Definitions in Context

```csharp
// Tools are automatically included in context
agent.RegisterToolExecutor(new WeatherToolExecutor());

// Context now includes:
// 1. System message
// 2. Conversation history
// 3. Tool definitions:
//    {
//      "type": "function",
//      "function": {
//        "name": "get_weather",
//        "description": "Get current weather",
//        "parameters": { ... }
//      }
//    }
```

### Tool Results in Context

```csharp
// After tool execution, result is added to context
// Tool call:
{
  "role": "assistant",
  "tool_calls": [{
    "id": "call_abc",
    "function": {
      "name": "get_weather",
      "arguments": "{\"location\":\"Tokyo\"}"
    }
  }]
}

// Tool result:
{
  "role": "tool",
  "tool_call_id": "call_abc",
  "content": "{\"temperature\":22,\"condition\":\"sunny\"}"
}
```

## Image Context

### Images in Messages

```csharp
// Images are included as content parts
var imageMessage = new Message
{
    Role = Role.User,
    Parts = new List<ContentPart>
    {
        new ContentPart { Type = "text", Text = "What's in this image?" },
        new ContentPart
        {
            Type = "image_url",
            ImageUrl = new ImageUrl
            {
                Url = imageBase64,
                Detail = "high"  // or "low" for smaller context
            }
        }
    }
};

// Context includes image data (can be large!)
```

### Image Context Management

```csharp
// Use low detail to reduce context size
imageUrl.Detail = "low";  // ~65 tokens vs ~765 tokens for high

// Remove old images from context
foreach (var message in agent.Messages)
{
    if (message.Parts.Any(p => p.Type == "image_url"))
    {
        var age = DateTime.UtcNow - message.Timestamp;
        if (age.TotalMinutes > 5)
        {
            // Remove image content parts
            message.Parts.RemoveAll(p => p.Type == "image_url");
        }
    }
}
```

## Metadata in Context

### Custom Context Metadata

```csharp
// Add context metadata
agent.Conversation.Metadata.CustomData = new Dictionary<string, object>
{
    ["user_id"] = "12345",
    ["session_id"] = "session_abc",
    ["platform"] = Application.platform.ToString()
};

// Use in system instructions
agent.Instructions = $@"
You are a Unity assistant.
User ID: {agent.Conversation.Metadata.CustomData["user_id"]}
Platform: {agent.Conversation.Metadata.CustomData["platform"]}
";
```

## Context Assembly Example

```csharp
// Simplified version of internal context assembly
public class ContextAssembler
{
    public List<Message> AssembleContext(
        Agent agent,
        string userMessage)
    {
        var context = new List<Message>();
        
        // 1. System message
        context.Add(new Message
        {
            Role = Role.System,
            Content = agent.Instructions
        });
        
        // 2. Conversation history (with memory window)
        var historyMessages = GetHistoryMessages(
            agent.Messages,
            agent.Settings.Memory
        );
        context.AddRange(historyMessages);
        
        // 3. Current user message
        context.Add(new Message
        {
            Role = Role.User,
            Content = userMessage
        });
        
        return context;
    }
    
    List<Message> GetHistoryMessages(
        List<Message> allMessages,
        AgentMemorySettings memory)
    {
        if (memory.Mode == ConversationMemoryMode.None)
        {
            return new List<Message>();
        }
        
        if (memory.Mode == ConversationMemoryMode.Full)
        {
            return allMessages;
        }
        
        // Recent mode: take last N messages
        int maxLength = memory.MaxConversationLength ?? 10;
        return allMessages
            .Skip(Math.Max(0, allMessages.Count - maxLength))
            .ToList();
    }
}
```

## Debugging Context

### Log Context Before Sending

```csharp
public async UniTask SendWithContextLog(string message)
{
    Debug.Log("=== Context Assembly ===");
    
    // System message
    Debug.Log($"System: {agent.Instructions}");
    
    // History
    Debug.Log($"History: {agent.Messages.Count} messages");
    foreach (var msg in agent.Messages.TakeLast(5))
    {
        Debug.Log($"  {msg.Role}: {msg.Content.Substring(0, Math.Min(50, msg.Content.Length))}...");
    }
    
    // Tools
    Debug.Log($"Tools: {agent.Tools.Count}");
    
    // Current message
    Debug.Log($"User: {message}");
    
    // Send
    await agent.SendAsync(message);
}
```

### Inspect API Request

```csharp
// Enable verbose logging (if supported)
agent.Settings.EnableVerboseLogging = true;

// Hook into request event
agent.onRequestSent += (request) =>
{
    Debug.Log($"API Request:");
    Debug.Log($"  Model: {request.Model}");
    Debug.Log($"  Messages: {request.Messages.Count}");
    Debug.Log($"  Tokens (estimated): {EstimateTokens(request)}");
};
```

## Best Practices

### 1. Optimize Context Size

```csharp
// Keep context within 50-80% of max tokens
int maxContextTokens = (int)((agent.MaxTokens ?? 4096) * 0.8f);

if (EstimateContextTokens() > maxContextTokens)
{
    // Trim conversation
    agent.ClearConversation(keepLastN: 5);
}
```

### 2. Clear Old Context

```csharp
// Periodically clear old messages
public void OnNewTopic()
{
    // Keep only system message and last exchange
    agent.ClearConversation(keepLastN: 2);
}
```

### 3. Use Summaries for Long Conversations

```csharp
memory.UseSummary = true;
memory.SummaryMessageCount = 50;

// Every 50 messages, create summary:
// "Previous conversation covered: X, Y, Z..."
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using System.Linq;

public class ContextManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Context Settings")]
    [SerializeField] private int maxContextMessages = 10;
    [SerializeField] private int maxContextTokens = 3000;
    
    void Start()
    {
        // Configure memory
        agent.Settings.Memory.Mode = ConversationMemoryMode.Recent;
        agent.Settings.Memory.MaxConversationLength = maxContextMessages;
    }
    
    public async void SendMessage(string message)
    {
        // Check context size
        if (!CheckContextSize())
        {
            TrimContext();
        }
        
        // Log context
        LogContext(message);
        
        // Send
        await agent.SendAsync(message);
    }
    
    bool CheckContextSize()
    {
        int estimatedTokens = EstimateContextTokens();
        return estimatedTokens <= maxContextTokens;
    }
    
    void TrimContext()
    {
        int keepCount = maxContextMessages / 2;
        agent.ClearConversation(keepLastN: keepCount);
        
        Debug.Log($"Context trimmed, keeping last {keepCount} messages");
    }
    
    int EstimateContextTokens()
    {
        int tokens = 0;
        
        // System message
        tokens += EstimateTokens(agent.Instructions);
        
        // History
        foreach (var message in agent.Messages)
        {
            tokens += EstimateTokens(message.Content);
        }
        
        // Tools
        tokens += agent.Tools.Count * 100;  // Rough estimate
        
        return tokens;
    }
    
    int EstimateTokens(string text)
    {
        return text.Length / 4;
    }
    
    void LogContext(string currentMessage)
    {
        Debug.Log($"=== Context ({EstimateContextTokens()} tokens) ===");
        Debug.Log($"System: {agent.Instructions.Substring(0, 50)}...");
        Debug.Log($"History: {agent.Messages.Count} messages");
        Debug.Log($"Current: {currentMessage}");
    }
}
```

## Next Steps

- [Messages & Items](messages-and-items.md)
- [Memory Settings](../configuration/memory-settings.md)
- [Temperature & Max Tokens](../parameters/temperature-tokens.md)
