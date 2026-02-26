---
icon: hard-drive
---

# Conversation Store

Configure how conversations are persisted and loaded across sessions.

## Store Types

### None (Do Not Save)

Conversations exist only in memory and are lost when agent is disposed.

**When to use:**

- Temporary conversations
- Privacy-sensitive applications
- Testing and development

```csharp
behaviour.ConversationStoreType = ConversationStoreType.None;
```

### Local File

Save conversations to local JSON files in persistent data path.

**When to use:**

- Single-player applications
- Offline support needed
- Full control over data

```csharp
behaviour.ConversationStoreType = ConversationStoreType.LocalFile;

// Files saved to:
// Application.persistentDataPath/AIDevKit/Conversations/{agentId}/{conversationId}.json
```

### Threads API (OpenAI)

Use OpenAI's Threads API for server-side persistence.

**When to use:**

- Using Assistants API
- Want OpenAI to manage state
- Need cross-device sync

```csharp
settings.ChatServiceApi = ChatService.AssistantsApi;
behaviour.ConversationStoreType = ConversationStoreType.ThreadsApi;
```

### Realtime API (OpenAI)

Use OpenAI's Realtime API session management.

**When to use:**

- Using Realtime API for voice
- Need ultra-low latency
- Server manages audio state

```csharp
settings.ChatServiceApi = ChatService.RealtimeApi;
behaviour.ConversationStoreType = ConversationStoreType.RealtimeApi;
```

### Custom Store

Implement your own storage backend (cloud, database, etc.).

**When to use:**

- Need custom storage solution
- Cloud database integration
- Multi-user sync requirements

```csharp
behaviour.ConversationStoreType = ConversationStoreType.Custom;

// Provide custom store implementation
var customStore = new MyCloudConversationStore();
// Configure agent with custom store
```

## Comparison

| Store Type | Persistence | Multi-Device | Offline | Server-Side |
|-----------|-------------|--------------|---------|-------------|
| **None** | ❌ | ❌ | ✅ | ❌ |
| **Local File** | ✅ | ❌ | ✅ | ❌ |
| **Threads API** | ✅ | ✅ | ❌ | ✅ |
| **Realtime API** | ✅ | ✅ | ❌ | ✅ |
| **Custom** | ✅ | ✅ | Depends | Depends |

## Loading Existing Conversations

```csharp
// Set conversation ID to load
behaviour.InitialConversationId = "conv_abc123";

// Agent will load this conversation on initialization
await agent.InitializeAsync();
```

## Creating New Conversations

```csharp
// Leave InitialConversationId empty for new conversation
behaviour.InitialConversationId = null;

// Or create explicitly
await agent.CreateNewConversationAsync();
```

## Listing Conversations

```csharp
// Get all conversations for this agent
var conversations = await agent.ListConversationsAsync();

foreach (var conv in conversations)
{
    Debug.Log($"Conversation: {conv.Id} - {conv.Metadata.Title}");
}
```

## Deleting Conversations

```csharp
// Delete current conversation
await agent.DeleteConversationAsync();

// Delete specific conversation
await agent.DeleteConversationAsync("conv_abc123");
```

## Best Practices

### Auto-Save Frequency

```csharp
// Configure auto-save interval
agent.ConversationAutoSaveInterval = TimeSpan.FromMinutes(5);
```

### Conversation Metadata

```csharp
// Add metadata to conversations
await agent.UpdateConversationMetadataAsync(new ConversationMetadata
{
    Title = "Project Discussion",
    Tags = new[] { "work", "important" },
    CustomData = new Dictionary<string, object>
    {
        ["projectId"] = "proj_123"
    }
});
```

### Cleanup Old Conversations

```csharp
// Delete conversations older than 30 days
var oldConversations = await agent.ListConversationsAsync();
var cutoffDate = DateTime.UtcNow.AddDays(-30);

foreach (var conv in oldConversations.Where(c => c.CreatedAt < cutoffDate))
{
    await agent.DeleteConversationAsync(conv.Id);
}
```

## Next Steps

- [Creating Conversations](../conversation/creating-conversations.md)
- [Loading & Saving](../conversation/loading-and-saving.md)
- [Messages & Items](../conversation/messages-and-items.md)
