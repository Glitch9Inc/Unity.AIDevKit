---
icon: plus-circle
---

# Creating Conversations

Learn how to create new conversations for your agents.

## Overview

Conversations store the message history between users and agents. Creating a new conversation gives you a fresh context for interactions.

## Basic Creation

### Using Agent

```csharp
// Create with default metadata
Conversation conversation = await agent.CreateNewConversationAsync();

Debug.Log($"Created conversation: {conversation.Id}");
```

### Using AgentBehaviour

```csharp
public class ChatManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void StartNewChat()
    {
        Conversation conv = await agent.CreateNewConversationAsync();
        Debug.Log($"New chat started: {conv.Id}");
    }
}
```

## With Metadata

### ConversationMetadata

```csharp
var metadata = new ConversationMetadata
{
    Title = "Customer Support Chat",
    Description = "Help with product setup",
    Tags = new[] { "support", "setup", "urgent" },
    CustomData = new Dictionary<string, object>
    {
        ["customerId"] = "12345",
        ["productId"] = "ABC-001",
        ["priority"] = "high"
    }
};

Conversation conversation = await agent.CreateNewConversationAsync(metadata);
```

### Update Metadata Later

```csharp
// Update title
conversation.Metadata.Title = "Resolved: Product Setup";

// Add tags
conversation.Metadata.Tags = conversation.Metadata.Tags
    .Append("resolved")
    .ToArray();

// Save changes
await agent.SaveConversationAsync();
```

## Auto-Save on Creation

```csharp
// Conversation is automatically saved based on ConversationStore setting
agent.ConversationStore = ConversationStoreType.LocalFile;

// Creates and saves immediately
var conversation = await agent.CreateNewConversationAsync();

// File saved to: Application.persistentDataPath/Conversations/{id}.json
```

## Store-Specific Creation

### Local File Storage

```csharp
agent.ConversationStore = ConversationStoreType.LocalFile;
var conv = await agent.CreateNewConversationAsync();

// Saved to: persistentDataPath/Conversations/{conversationId}.json
```

### Threads API (OpenAI Assistants)

```csharp
agent.ChatServiceApi = ChatService.AssistantsApi;
agent.ConversationStore = ConversationStoreType.ThreadsApi;

var conv = await agent.CreateNewConversationAsync();

// Creates OpenAI Thread and stores thread_id
Debug.Log($"Thread ID: {conv.Id}");
```

### Conversations API (OpenAI)

```csharp
agent.ChatServiceApi = ChatService.ChatCompletions;
agent.ConversationStore = ConversationStoreType.ConversationsApi;

var conv = await agent.CreateNewConversationAsync();

// Uses OpenAI Conversations API (if available)
```

### Realtime API

```csharp
agent.ChatServiceApi = ChatService.RealtimeApi;
agent.ConversationStore = ConversationStoreType.RealtimeApi;

// Session-based, created on connect
await agent.InitializeAsync();
```

### No Storage

```csharp
agent.ConversationStore = ConversationStoreType.None;

var conv = await agent.CreateNewConversationAsync();

// Conversation exists in memory only
// Lost on app restart
```

## Initialize with System Message

```csharp
var conversation = await agent.CreateNewConversationAsync();

// Agent automatically uses Instructions as system message
// Instructions come from AgentSettings

// To customize per-conversation:
conversation.Messages.Insert(0, new Message
{
    Role = Role.System,
    Content = "You are a helpful assistant specialized in Unity development."
});
```

## Multiple Conversations

```csharp
public class MultiChatManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, Conversation> activeChats = new();
    
    public async UniTask<string> CreateChatRoom(string roomName)
    {
        var metadata = new ConversationMetadata
        {
            Title = roomName,
            Tags = new[] { "room" }
        };
        
        var conversation = await agent.CreateNewConversationAsync(metadata);
        activeChats[conversation.Id] = conversation;
        
        return conversation.Id;
    }
    
    public async UniTask SwitchToRoom(string conversationId)
    {
        await agent.LoadConversationAsync(conversationId);
    }
}
```

## Error Handling

```csharp
try
{
    var conversation = await agent.CreateNewConversationAsync();
}
catch (ApiException ex) when (ex.StatusCode == 401)
{
    Debug.LogError("API key invalid");
}
catch (ApiException ex) when (ex.StatusCode == 429)
{
    Debug.LogError("Rate limit exceeded");
}
catch (Exception ex)
{
    Debug.LogError($"Failed to create conversation: {ex.Message}");
}
```

## Best Practices

### 1. Always Handle Errors

```csharp
public async UniTask<Conversation> SafeCreateConversation()
{
    try
    {
        return await agent.CreateNewConversationAsync();
    }
    catch (Exception ex)
    {
        Debug.LogError($"Conversation creation failed: {ex}");
        
        // Fallback: use in-memory only
        agent.ConversationStore = ConversationStoreType.None;
        return await agent.CreateNewConversationAsync();
    }
}
```

### 2. Set Meaningful Metadata

```csharp
var metadata = new ConversationMetadata
{
    Title = $"Chat - {DateTime.Now:yyyy-MM-dd HH:mm}",
    Description = "Support conversation",
    Tags = new[] { "support", userId },
    CustomData = new Dictionary<string, object>
    {
        ["userId"] = userId,
        ["startTime"] = DateTime.UtcNow,
        ["platform"] = Application.platform.ToString()
    }
};
```

### 3. Track Conversation IDs

```csharp
// Save current conversation ID
PlayerPrefs.SetString("LastConversationId", conversation.Id);

// Resume on restart
string lastId = PlayerPrefs.GetString("LastConversationId");
if (!string.IsNullOrEmpty(lastId))
{
    await agent.LoadConversationAsync(lastId);
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using System;

public class ConversationCreator : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void Start()
    {
        // Configure storage
        agent.ConversationStore = ConversationStoreType.LocalFile;
        
        // Create conversation with metadata
        var metadata = new ConversationMetadata
        {
            Title = "Unity Support Chat",
            Description = "Getting help with AI agents",
            Tags = new[] { "unity", "support", "agents" },
            CustomData = new Dictionary<string, object>
            {
                ["userId"] = GetUserId(),
                ["sessionStart"] = DateTime.UtcNow,
                ["appVersion"] = Application.version
            }
        };
        
        try
        {
            Conversation conversation = await agent.CreateNewConversationAsync(metadata);
            
            Debug.Log($"✓ Conversation created");
            Debug.Log($"  ID: {conversation.Id}");
            Debug.Log($"  Title: {conversation.Metadata.Title}");
            Debug.Log($"  Stored: {agent.ConversationStore}");
            
            // Save conversation ID for later
            PlayerPrefs.SetString("CurrentConversation", conversation.Id);
            
            // Start chatting
            await agent.SendAsync("Hello! I need help with agents.");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Failed to create conversation: {ex.Message}");
        }
    }
    
    string GetUserId()
    {
        // Get or create user ID
        string userId = PlayerPrefs.GetString("UserId");
        if (string.IsNullOrEmpty(userId))
        {
            userId = Guid.NewGuid().ToString();
            PlayerPrefs.SetString("UserId", userId);
        }
        return userId;
    }
}
```

## Next Steps

- [Loading & Saving](loading-and-saving.md)
- [Messages & Items](messages-and-items.md)
- [Conversation Store](../configuration/conversation-store.md)
