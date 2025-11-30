# Conversation Management

Manage conversations, messages, and conversation history.

## Overview

Conversations in AI Dev Kit consist of:

- **Conversation** - Container for all interaction history
- **Messages** - User and assistant text messages
- **Items** - Low-level events (messages, tool calls, etc.)
- **Metadata** - Title, tags, timestamps

## Creating Conversations

### New Conversation (Auto)

```csharp
// Agent creates new conversation automatically
await agent.InitializeAsync();

Debug.Log($"Conversation ID: {agent.ConversationId}");
```

### New Conversation (Manual)

```csharp
await agent.CreateNewConversationAsync();
Debug.Log($"Created: {agent.ConversationId}");
```

### With Metadata

```csharp
var metadata = new ConversationMetadata
{
    Title = "Customer Support",
    Tags = new[] { "support", "billing" },
    CustomData = new Dictionary<string, object>
    {
        ["customerId"] = "cust_123",
        ["priority"] = "high"
    }
};

await agent.CreateNewConversationAsync(metadata);
```

## Loading Conversations

### Load by ID

```csharp
// Set before initialization
agent.ConversationId = "conv_abc123";
await agent.InitializeAsync();
```

### Load at Runtime

```csharp
await agent.LoadConversationAsync("conv_abc123");
Debug.Log($"Loaded {agent.Messages.Count} messages");
```

### List All Conversations

```csharp
var conversations = await agent.ListConversationsAsync();

foreach (var conv in conversations)
{
    Debug.Log($"{conv.Id}: {conv.Metadata.Title}");
}
```

## Saving Conversations

### Auto-Save

```csharp
// Configure auto-save
agent.AutoSaveOnDispose = true;
agent.AutoSaveInterval = TimeSpan.FromMinutes(5);
```

### Manual Save

```csharp
await agent.SaveConversationAsync();
```

### Save with Metadata Update

```csharp
agent.Conversation.Metadata.Title = "Updated Title";
await agent.SaveConversationAsync();
```

## Messages & Items

### Access Messages

```csharp
// All messages (user + assistant)
List<Message> messages = agent.Messages;

// Last message
Message lastMessage = agent.LastMessage;

// Filter by role
var userMessages = messages.Where(m => m.Role == Role.User);
var assistantMessages = messages.Where(m => m.Role == Role.Assistant);
```

### Access Items

```csharp
// All items (messages, tool calls, etc.)
List<ConversationItem> items = agent.Items;

// Filter by type
var messageItems = items.OfType<MessageItem>();
var toolCallItems = items.OfType<ToolCallItem>();
```

### Add Message

```csharp
// System message
agent.AddMessage("You are a helpful assistant", Role.System);

// User message
agent.AddMessage("Hello!", Role.User);
```

### Remove Message

```csharp
agent.RemoveMessage(messageId);
```

## Context Assembly

### What's Included in Context

1. System instructions
2. Memory/summary (if enabled)
3. Recent messages (up to MaxContextMessages)
4. Tool definitions
5. Current user message

### Custom Context

```csharp
// Add custom context
agent.AddContextItem(new ContextItem
{
    Type = "custom_data",
    Content = JsonConvert.SerializeObject(myData)
});
```

### Context Window Management

```csharp
// Estimate tokens
int tokens = agent.EstimateContextTokens();
int limit = agent.Model.GetContextWindow();

Debug.Log($"Using {tokens}/{limit} tokens ({tokens * 100 / limit}%)");
```

## Conversation Metadata

### Update Metadata

```csharp
agent.Conversation.Metadata.Title = "New Title";
agent.Conversation.Metadata.Tags = new[] { "important" };
agent.Conversation.Metadata.CustomData["status"] = "resolved";

await agent.SaveConversationAsync();
```

### Search by Metadata

```csharp
var conversations = await agent.ListConversationsAsync();
var tagged = conversations.Where(c => 
    c.Metadata.Tags.Contains("important")
);
```

## Deleting Conversations

### Delete Current

```csharp
await agent.DeleteConversationAsync();
```

### Delete by ID

```csharp
await agent.DeleteConversationAsync("conv_abc123");
```

### Delete Multiple

```csharp
var conversations = await agent.ListConversationsAsync();
var old = conversations.Where(c => c.CreatedAt < DateTime.UtcNow.AddDays(-30));

foreach (var conv in old)
{
    await agent.DeleteConversationAsync(conv.Id);
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ConversationManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    // Create new conversation
    public async void CreateNew()
    {
        await agent.CreateNewConversationAsync(new ConversationMetadata
        {
            Title = "New Chat",
            Tags = new[] { "chat" }
        });
        
        Debug.Log($"Created: {agent.ConversationId}");
    }
    
    // Load conversation
    public async void Load(string conversationId)
    {
        await agent.LoadConversationAsync(conversationId);
        Debug.Log($"Loaded {agent.Messages.Count} messages");
    }
    
    // List conversations
    public async void ListAll()
    {
        var conversations = await agent.ListConversationsAsync();
        
        foreach (var conv in conversations)
        {
            Debug.Log($"[{conv.Id}] {conv.Metadata.Title}");
            Debug.Log($"  Messages: {conv.MessageCount}");
            Debug.Log($"  Created: {conv.CreatedAt}");
        }
    }
    
    // Save conversation
    public async void Save()
    {
        await agent.SaveConversationAsync();
        Debug.Log("Saved");
    }
    
    // Delete conversation
    public async void Delete()
    {
        string id = agent.ConversationId;
        await agent.DeleteConversationAsync();
        Debug.Log($"Deleted: {id}");
    }
    
    // Clear current conversation
    public void Clear()
    {
        agent.ClearConversation();
        Debug.Log("Conversation cleared");
    }
}
```

## Next Steps

- [Creating Conversations](creating-conversations.md)
- [Loading & Saving](loading-and-saving.md)
- [Messages & Items](messages-and-items.md)
- [Context Assembly](context-assembly.md)
