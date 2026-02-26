# Conversation Stores

Learn how conversations are persisted and managed across different storage backends.

## Overview

Conversation stores determine how agent conversations are saved, loaded, and managed. AI Dev Kit supports multiple storage backends:

- **None** - No persistence (in-memory only)
- **LocalFile** - Local file system storage
- **ThreadsApi** - OpenAI Threads API
- **RealtimeApi** - OpenAI Realtime API
- **ConversationsApi** - OpenAI Conversations API
- **Custom** - Your own storage implementation

## ConversationStoreType

```csharp
public enum ConversationStoreType
{
    None,                  // Do not save
    LocalFile,             // Local file storage
    ThreadsApi,            // OpenAI Threads API
    RealtimeApi,           // OpenAI Realtime API
    ConversationsApi,      // OpenAI Conversations API
    Custom,                // Custom storage implementation
}
```

## Store Implementations

### IConversationStore Interface

All stores implement this interface:

```csharp
public interface IConversationStore
{
    ConversationStoreType StoreType { get; }
    ConversationPersistMode PersistMode { get; }

    // Conversation management (requires agentId)
    UniTask<Conversation> CreateAsync(string agentId, CancellationToken ct = default);
    UniTask<Conversation[]> ListAsync(string agentId, CancellationToken ct = default);

    // Conversation operations (convId is globally unique)
    UniTask<Conversation> LoadAsync(string agentId, string convId, CancellationToken ct = default);
    UniTask<Conversation> UpdateAsync(Conversation conversation, CancellationToken ct = default);
    UniTask<bool> DeleteAsync(string agentId, string convId, CancellationToken ct = default);

    // Item operations
    UniTask<ConversationItem[]> UpdateItemsAsync(string agentId, string convId, ConversationItem[] items, CancellationToken ct = default);
    UniTask<ConversationItem[]> LoadItemsAsync(string agentId, string convId, CancellationToken ct = default);
    UniTask<bool> DeleteItemAsync(string agentId, string convId, string itemId, CancellationToken ct = default);
    UniTask<ConversationItem> LoadItemAsync(string agentId, string convId, string itemId, CancellationToken ct = default);
}
```

### ConversationPersistMode

Controls how conversation items are persisted:

```csharp
public enum ConversationPersistMode
{
    DeltaAppend,    // Only append new items (default, most efficient)
    FullSnapshot,   // Replace all items on every update
}
```

---

## Built-in Stores

### 1. NoopConversationStore (None)

No persistence - conversations exist only in memory.

**When to use:**
- Testing and development
- Temporary conversations
- When you don't need history

**Configuration:**
```csharp
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.None
};
```

**Behavior:**
- All operations return `null` or empty results
- No data is saved to disk or API
- Conversations are lost when agent is disposed

---

### 2. LocalFileStore

Saves conversations to local JSON files.

**When to use:**
- Desktop applications
- Single-device storage
- Offline-first apps
- Development and testing

**Configuration:**
```csharp
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.LocalFile
};
```

**File Structure:**
```
Application.persistentDataPath/
  AIDevKit/
    Conversations/
      {agentId}/
        {conversationId}.json
        {conversationId}_items.json
```

**Features:**
- Fast local access
- Works offline
- No API costs
- Cross-session persistence

**Limitations:**
- Not synchronized across devices
- Manual backup required
- No multi-user support

**Example:**
```csharp
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class LocalStorageExample
{
    async void CreateLocalConversation()
    {
        var config = new AgentConfig
        {
            ConversationStore = ConversationStoreType.LocalFile
        };
        
        var agent = new Agent(config, settings);
        await agent.InitializeAsync();
        
        // Conversation is automatically saved to local file
        await agent.SendAsync("Hello!");
        
        // Load conversation in a new session
        var conversations = await agent.conversationController.ListConversationsAsync();
        if (conversations.Length > 0)
        {
            await agent.conversationController.LoadConversationAsync(conversations[0].Id);
        }
    }
}
```

---

### 3. ThreadsApiStore

Uses OpenAI's Threads API for conversation storage.

**When to use:**
- When using OpenAI Assistants API
- Multi-device synchronization needed
- Cloud-based conversation history
- Collaborative conversations

**Configuration:**
```csharp
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.ThreadsApi,
    ChatApi = ChatApiType.AssistantsApi  // Required
};
```

**Features:**
- Cloud synchronization
- Multi-device access
- OpenAI manages storage
- Built-in message management

**Limitations:**
- Requires OpenAI API key
- API costs apply
- Requires internet connection
- Only works with Assistants API

**Example:**
```csharp
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ThreadsApiExample
{
    async void CreateThreadConversation()
    {
        var config = new AgentConfig
        {
            ConversationStore = ConversationStoreType.ThreadsApi,
            ChatApi = ChatApiType.AssistantsApi
        };
        
        var agent = new Agent(config, settings);
        await agent.InitializeAsync();
        
        // Creates a new thread on OpenAI
        await agent.SendAsync("Hello!");
        
        // Thread is accessible from any device
        string threadId = agent.Conversation.Id;
        
        // Later, load the same thread
        await agent.conversationController.LoadConversationAsync(threadId);
    }
}
```

---

### 4. ConversationsApiStore

Uses OpenAI's Conversations API (Responses API).

**When to use:**
- When using OpenAI Responses API
- Modern conversation management
- Delta-based updates
- Efficient storage

**Configuration:**
```csharp
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.ConversationsApi,
    ChatApi = ChatApiType.ResponsesApi  // Required
};
```

**Features:**
- Efficient delta updates
- Cloud synchronization
- Metadata support
- Item-level operations

**Implementation:**
```csharp
sealed class ConversationsApiStore : IConversationStore
{
    public ConversationStoreType StoreType => ConversationStoreType.ConversationsApi;
    public ConversationPersistMode PersistMode { get; set; }
    
    public async UniTask<Conversation> CreateAsync(string agentId, CancellationToken ct = default)
    {
        return await OpenAIClient.DefaultInstance.Conversations.CreateAsync(agentId, ct.ToRequestOptions());
    }
    
    public async UniTask<ConversationItem[]> UpdateItemsAsync(string agentId, string convId, ConversationItem[] items, CancellationToken ct = default)
    {
        if (items.IsNullOrEmpty()) return items;
        var response = await OpenAIClient.DefaultInstance.Conversations.CreateItemsAsync(convId, items, ct.ToRequestOptions());
        return response?.Data;
    }
    
    // ... other methods
}
```

**Example:**
```csharp
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ConversationsApiExample
{
    async void CreateConversation()
    {
        var config = new AgentConfig
        {
            ConversationStore = ConversationStoreType.ConversationsApi,
            ChatApi = ChatApiType.ResponsesApi
        };
        
        var agent = new Agent(config, settings);
        await agent.InitializeAsync();
        
        // Conversation created via Conversations API
        await agent.SendAsync("Hello!");
        
        // Items are persisted incrementally
        await agent.SendAsync("How are you?");
        
        // Load items
        var items = await agent.conversationController.LoadConversationItemsAsync();
        foreach (var item in items)
        {
            Debug.Log($"{item.Role}: {item.GetText()}");
        }
    }
}
```

---

### 5. RealtimeApiStore

Storage for OpenAI Realtime API conversations.

**When to use:**
- Voice-based conversations
- Real-time audio interactions
- Low-latency applications

**Configuration:**
```csharp
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.RealtimeApi,
    ChatApi = ChatApiType.RealtimeApi  // Required
};
```

---

## Custom Store Implementation

Create your own storage backend by implementing `IConversationStore`.

### Example: Cloud Storage

```csharp
using Glitch9.AIDevKit.Conversations;
using Cysharp.Threading.Tasks;
using System.Threading;

public class CloudConversationStore : IConversationStore
{
    public ConversationStoreType StoreType => ConversationStoreType.Custom;
    public ConversationPersistMode PersistMode { get; set; } = ConversationPersistMode.DeltaAppend;
    
    private readonly ICloudStorageService cloudService;
    
    public CloudConversationStore(ICloudStorageService cloudService)
    {
        this.cloudService = cloudService;
    }
    
    public async UniTask<Conversation> CreateAsync(string agentId, CancellationToken ct = default)
    {
        var conversation = new Conversation
        {
            Id = System.Guid.NewGuid().ToString(),
            AgentId = agentId,
            CreatedAt = System.DateTime.UtcNow
        };
        
        await cloudService.SaveAsync($"conversations/{agentId}/{conversation.Id}.json", conversation);
        return conversation;
    }
    
    public async UniTask<Conversation> LoadAsync(string agentId, string convId, CancellationToken ct = default)
    {
        return await cloudService.LoadAsync<Conversation>($"conversations/{agentId}/{convId}.json");
    }
    
    public async UniTask<Conversation> UpdateAsync(Conversation conversation, CancellationToken ct = default)
    {
        await cloudService.SaveAsync($"conversations/{conversation.AgentId}/{conversation.Id}.json", conversation);
        return conversation;
    }
    
    public async UniTask<bool> DeleteAsync(string agentId, string convId, CancellationToken ct = default)
    {
        return await cloudService.DeleteAsync($"conversations/{agentId}/{convId}.json");
    }
    
    public async UniTask<Conversation[]> ListAsync(string agentId, CancellationToken ct = default)
    {
        return await cloudService.ListAsync<Conversation>($"conversations/{agentId}/");
    }
    
    public async UniTask<ConversationItem[]> UpdateItemsAsync(string agentId, string convId, ConversationItem[] items, CancellationToken ct = default)
    {
        await cloudService.SaveAsync($"conversations/{agentId}/{convId}_items.json", items);
        return items;
    }
    
    public async UniTask<ConversationItem[]> LoadItemsAsync(string agentId, string convId, CancellationToken ct = default)
    {
        return await cloudService.LoadAsync<ConversationItem[]>($"conversations/{agentId}/{convId}_items.json");
    }
    
    public async UniTask<ConversationItem> LoadItemAsync(string agentId, string convId, string itemId, CancellationToken ct = default)
    {
        var items = await LoadItemsAsync(agentId, convId, ct);
        return items?.FirstOrDefault(i => i.Id == itemId);
    }
    
    public async UniTask<bool> DeleteItemAsync(string agentId, string convId, string itemId, CancellationToken ct = default)
    {
        var items = await LoadItemsAsync(agentId, convId, ct);
        if (items == null) return false;
        
        var newItems = items.Where(i => i.Id != itemId).ToArray();
        await UpdateItemsAsync(agentId, convId, newItems, ct);
        return true;
    }
}
```

### Usage:

```csharp
var customStore = new CloudConversationStore(myCloudService);

// Option 1: Pass during config creation
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.Custom,
    CustomConversationStore = customStore
};

// Option 2: Set on conversation controller
agent.conversationController.SetCustomStore(customStore);
```

---

## Store Selection Guide

| Store Type | Best For | Pros | Cons |
|------------|----------|------|------|
| **None** | Testing, temporary chats | Fast, no setup | No persistence |
| **LocalFile** | Desktop apps, offline | Fast, offline, free | No sync, single device |
| **ThreadsApi** | Assistants API, cloud sync | Multi-device, managed | Requires OpenAI, costs |
| **ConversationsApi** | Responses API, modern apps | Efficient, managed | Requires OpenAI, costs |
| **RealtimeApi** | Voice conversations | Real-time, low latency | Requires OpenAI, costs |
| **Custom** | Specific requirements | Full control | Implementation effort |

---

## Persist Modes

### DeltaAppend (Recommended)

Only new items are saved to storage:

```csharp
var store = new ConversationsApiStore(ConversationPersistMode.DeltaAppend);
```

**Advantages:**
- Most efficient
- Minimal API calls
- Faster saves
- Reduced bandwidth

**Works for:**
- Append-only conversations
- Most use cases

### FullSnapshot

Entire conversation is saved on every update:

```csharp
var store = new ConversationsApiStore(ConversationPersistMode.FullSnapshot);
```

**Advantages:**
- Complete state preservation
- Handles edits/deletions
- Simpler consistency model

**Disadvantages:**
- More API calls
- Higher costs
- Slower for large conversations

**When to use:**
- Conversations with edits
- Item deletions
- Complex state management

---

## Best Practices

### ✅ Do

```csharp
// Choose store based on requirements
var config = new AgentConfig
{
    ConversationStore = IsOnline() ? ConversationStoreType.ConversationsApi : ConversationStoreType.LocalFile
};
```

```csharp
// Handle store errors gracefully
try
{
    await agent.conversationController.LoadConversationAsync(convId);
}
catch (Exception ex)
{
    Debug.LogError($"Failed to load conversation: {ex.Message}");
    // Fall back to local storage or create new conversation
}
```

```csharp
// Dispose agents properly
using (var agent = new Agent(config, settings))
{
    // Use agent
}
// Store cleanup happens automatically
```

### ❌ Don't

```csharp
// Don't use API stores without internet
var config = new AgentConfig
{
    ConversationStore = ConversationStoreType.ConversationsApi  // ❌ Requires internet
};
```

```csharp
// Don't forget migration when switching stores
// Conversations won't automatically transfer between stores
```

---

## Migration Between Stores

To migrate conversations between stores:

```csharp
public async UniTask MigrateConversations(IConversationStore fromStore, IConversationStore toStore, string agentId)
{
    // List all conversations
    var conversations = await fromStore.ListAsync(agentId);
    
    foreach (var conversation in conversations)
    {
        // Load items from old store
        var items = await fromStore.LoadItemsAsync(agentId, conversation.Id);
        
        // Create in new store
        var newConv = await toStore.CreateAsync(agentId);
        
        // Copy metadata
        newConv.Title = conversation.Title;
        newConv.Summary = conversation.Summary;
        newConv.Metadata = conversation.Metadata;
        await toStore.UpdateAsync(newConv);
        
        // Copy items
        await toStore.UpdateItemsAsync(agentId, newConv.Id, items);
        
        Debug.Log($"Migrated: {conversation.Id} -> {newConv.Id}");
    }
}
```

---

## Next Steps

- [How Conversations Work](how-conversations-work.md) - Conversation architecture
- [Saving & Loading](saving-and-loading.md) - Conversation persistence
- [Multiple Conversations](multiple-conversations.md) - Managing multiple conversations
