# Messages & Items

Access and manage conversation messages and items.

## Overview

- **Messages** - Traditional chat messages (ChatCompletions, AssistantsApi)
- **Items** - Enhanced conversation items (RealtimeApi) including audio

## Messages

### Access Messages

```csharp
// Get all messages
List<Message> messages = agent.Messages;

// Get last message
Message lastMessage = agent.LastMessage;

// Get last user message
Message lastUserMessage = agent.Messages
    .LastOrDefault(m => m.Role == Role.User);

// Get last assistant message
Message lastAssistantMessage = agent.Messages
    .LastOrDefault(m => m.Role == Role.Assistant);
```

### Message Structure

```csharp
public class Message
{
    public string Id { get; set; }
    public Role Role { get; set; }  // System, User, Assistant, Tool
    public string Content { get; set; }
    public List<ContentPart> Parts { get; set; }
    public DateTime Timestamp { get; set; }
    public Dictionary<string, object> Metadata { get; set; }
}
```

### Message Roles

```csharp
public enum Role
{
    System,      // System instructions
    User,        // User messages
    Assistant,   // AI responses
    Tool         // Tool outputs
}
```

### Content Parts

```csharp
// Text message
var message = new Message
{
    Role = Role.User,
    Content = "Hello!",
    Parts = new List<ContentPart>
    {
        new ContentPart { Type = "text", Text = "Hello!" }
    }
};

// Image message
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
                Url = "data:image/png;base64,...",
                Detail = "high"
            }
        }
    }
};

// Audio message
var audioMessage = new Message
{
    Role = Role.User,
    Parts = new List<ContentPart>
    {
        new ContentPart
        {
            Type = "input_audio",
            InputAudio = new InputAudio
            {
                Data = audioBase64,
                Format = "wav"
            }
        }
    }
};
```

## Items (Realtime API)

### Access Items

```csharp
// Get all conversation items
List<ConversationItem> items = agent.Items;

// Filter by type
var messages = items.Where(i => i.Type == "message").ToList();
var functionCalls = items.Where(i => i.Type == "function_call").ToList();
var functionCallOutputs = items.Where(i => i.Type == "function_call_output").ToList();
```

### Item Structure

```csharp
public class ConversationItem
{
    public string Id { get; set; }
    public string Type { get; set; }  // message, function_call, function_call_output
    public string Status { get; set; }  // completed, in_progress, incomplete
    public Role Role { get; set; }
    public List<ContentPart> Content { get; set; }
}
```

### Item Types

```csharp
// Message item
{
    "type": "message",
    "role": "user",
    "content": [
        { "type": "text", "text": "Hello!" }
    ]
}

// Function call item
{
    "type": "function_call",
    "name": "get_weather",
    "call_id": "call_abc123",
    "arguments": "{\"location\":\"Tokyo\"}"
}

// Function call output item
{
    "type": "function_call_output",
    "call_id": "call_abc123",
    "output": "{\"temperature\":22,\"condition\":\"sunny\"}"
}
```

## Adding Messages

### Manually Add User Message

```csharp
var message = new Message
{
    Role = Role.User,
    Content = "Hello!",
    Timestamp = DateTime.UtcNow
};

agent.Conversation.Messages.Add(message);
await agent.SaveConversationAsync();
```

### Manually Add System Message

```csharp
var systemMessage = new Message
{
    Role = Role.System,
    Content = "You are a Unity development expert.",
    Timestamp = DateTime.UtcNow
};

// Insert at beginning
agent.Conversation.Messages.Insert(0, systemMessage);
```

### Add Tool Response

```csharp
var toolMessage = new Message
{
    Role = Role.Tool,
    Content = toolOutput,
    Metadata = new Dictionary<string, object>
    {
        ["tool_call_id"] = toolCallId,
        ["tool_name"] = toolName
    }
};

agent.Conversation.Messages.Add(toolMessage);
```

## Modifying Messages

### Edit Last Message

```csharp
// Get last user message
var lastUserMessage = agent.Messages
    .LastOrDefault(m => m.Role == Role.User);

if (lastUserMessage != null)
{
    // Edit content
    lastUserMessage.Content = "Updated message";
    
    // Save changes
    await agent.SaveConversationAsync();
}
```

### Delete Messages

```csharp
// Remove last message
if (agent.Messages.Count > 0)
{
    agent.Messages.RemoveAt(agent.Messages.Count - 1);
}

// Remove specific message
agent.Messages.RemoveAll(m => m.Id == messageId);

// Clear all except system message
var systemMessage = agent.Messages.FirstOrDefault(m => m.Role == Role.System);
agent.Messages.Clear();
if (systemMessage != null)
{
    agent.Messages.Add(systemMessage);
}
```

## Filtering Messages

### By Role

```csharp
// User messages only
var userMessages = agent.Messages
    .Where(m => m.Role == Role.User)
    .ToList();

// Assistant messages only
var assistantMessages = agent.Messages
    .Where(m => m.Role == Role.Assistant)
    .ToList();
```

### By Date Range

```csharp
var today = DateTime.Today;
var todayMessages = agent.Messages
    .Where(m => m.Timestamp.Date == today)
    .ToList();

var lastHour = DateTime.UtcNow.AddHours(-1);
var recentMessages = agent.Messages
    .Where(m => m.Timestamp >= lastHour)
    .ToList();
```

### By Content

```csharp
// Search for keyword
var searchResults = agent.Messages
    .Where(m => m.Content.Contains("Unity", StringComparison.OrdinalIgnoreCase))
    .ToList();
```

## Message Statistics

### Count Messages

```csharp
int totalMessages = agent.Messages.Count;
int userMessages = agent.Messages.Count(m => m.Role == Role.User);
int assistantMessages = agent.Messages.Count(m => m.Role == Role.Assistant);

Debug.Log($"Total: {totalMessages}");
Debug.Log($"User: {userMessages}");
Debug.Log($"Assistant: {assistantMessages}");
```

### Calculate Token Usage

```csharp
// Rough estimation (4 characters ≈ 1 token)
int EstimateTokens(string text)
{
    return text.Length / 4;
}

int totalTokens = agent.Messages
    .Sum(m => EstimateTokens(m.Content));

Debug.Log($"Estimated tokens: {totalTokens}");
```

### Message Rate

```csharp
if (agent.Messages.Count >= 2)
{
    var firstMessage = agent.Messages.First();
    var lastMessage = agent.Messages.Last();
    
    var duration = lastMessage.Timestamp - firstMessage.Timestamp;
    var rate = agent.Messages.Count / duration.TotalMinutes;
    
    Debug.Log($"Messages per minute: {rate:F2}");
}
```

## Display Messages in UI

### Simple Chat Display

```csharp
public class ChatDisplay : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private ChatMessageUI messagePrefab;
    [SerializeField] private Transform messagesContainer;
    
    void Start()
    {
        agent.onResponseCompleted.AddListener(OnResponseReceived);
        RefreshMessages();
    }
    
    void RefreshMessages()
    {
        // Clear existing
        foreach (Transform child in messagesContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Create UI for each message
        foreach (var message in agent.Messages)
        {
            if (message.Role == Role.System) continue;  // Skip system messages
            
            var messageUI = Instantiate(messagePrefab, messagesContainer);
            messageUI.Setup(message);
        }
    }
    
    void OnResponseReceived(Response response)
    {
        RefreshMessages();
    }
}
```

### Advanced Chat Display

```csharp
public class AdvancedChatDisplay : MonoBehaviour
{
    private Dictionary<string, ChatMessageUI> messageUIs = new();
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnTextDelta(string delta)
    {
        // Update last message in real-time
        var lastMessage = agent.LastMessage;
        
        if (!messageUIs.TryGetValue(lastMessage.Id, out var messageUI))
        {
            messageUI = CreateMessageUI(lastMessage);
            messageUIs[lastMessage.Id] = messageUI;
        }
        
        messageUI.UpdateText(lastMessage.Content);
    }
    
    void OnResponseCompleted(Response response)
    {
        // Finalize message display
        if (messageUIs.TryGetValue(response.Id, out var messageUI))
        {
            messageUI.SetCompleted();
        }
    }
}
```

## Export Messages

### Export to Text

```csharp
public string ExportToText()
{
    var sb = new StringBuilder();
    
    foreach (var message in agent.Messages)
    {
        if (message.Role == Role.System) continue;
        
        sb.AppendLine($"[{message.Timestamp:yyyy-MM-dd HH:mm:ss}] {message.Role}:");
        sb.AppendLine(message.Content);
        sb.AppendLine();
    }
    
    return sb.ToString();
}
```

### Export to Markdown

```csharp
public string ExportToMarkdown()
{
    var sb = new StringBuilder();
    sb.AppendLine($"# Conversation: {agent.Conversation.Metadata.Title}");
    sb.AppendLine();
    
    foreach (var message in agent.Messages)
    {
        if (message.Role == Role.System) continue;
        
        string role = message.Role == Role.User ? "👤 User" : "🤖 Assistant";
        sb.AppendLine($"## {role}");
        sb.AppendLine($"*{message.Timestamp:yyyy-MM-dd HH:mm:ss}*");
        sb.AppendLine();
        sb.AppendLine(message.Content);
        sb.AppendLine();
    }
    
    return sb.ToString();
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using System.Linq;
using System.Text;

public class MessageManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onResponseCompleted.AddListener(OnResponseReceived);
    }
    
    void OnResponseReceived(Response response)
    {
        DisplayMessageStats();
    }
    
    void DisplayMessageStats()
    {
        int total = agent.Messages.Count;
        int user = agent.Messages.Count(m => m.Role == Role.User);
        int assistant = agent.Messages.Count(m => m.Role == Role.Assistant);
        
        Debug.Log($"Messages: {total} (User: {user}, Assistant: {assistant})");
        
        if (agent.LastMessage != null)
        {
            Debug.Log($"Last: {agent.LastMessage.Content.Substring(0, Math.Min(50, agent.LastMessage.Content.Length))}...");
        }
    }
    
    public void ClearOldMessages()
    {
        var cutoff = DateTime.UtcNow.AddDays(-7);
        
        int removed = agent.Messages.RemoveAll(m =>
            m.Role != Role.System &&
            m.Timestamp < cutoff
        );
        
        Debug.Log($"Removed {removed} old messages");
    }
    
    public void ExportConversation()
    {
        string markdown = ExportToMarkdown();
        
        string path = Path.Combine(
            Application.persistentDataPath,
            $"conversation_{DateTime.Now:yyyyMMdd_HHmmss}.md"
        );
        
        File.WriteAllText(path, markdown);
        Debug.Log($"Exported to: {path}");
    }
    
    string ExportToMarkdown()
    {
        var sb = new StringBuilder();
        sb.AppendLine($"# {agent.Conversation.Metadata.Title}");
        sb.AppendLine();
        
        foreach (var message in agent.Messages)
        {
            if (message.Role == Role.System) continue;
            
            string icon = message.Role == Role.User ? "👤" : "🤖";
            sb.AppendLine($"## {icon} {message.Role}");
            sb.AppendLine($"*{message.Timestamp:yyyy-MM-dd HH:mm:ss}*");
            sb.AppendLine();
            sb.AppendLine(message.Content);
            sb.AppendLine();
        }
        
        return sb.ToString();
    }
}
```

## Next Steps

- [Creating Conversations](creating-conversations.md)
- [Loading & Saving](loading-and-saving.md)
- [Context Assembly](context-assembly.md)
