# Loading & Saving Conversations

Persist and restore conversation history across sessions.

## Overview

AI Dev Kit supports:

- **Auto-save** - Automatic persistence after each message
- **Manual save** - Explicit control over when to save
- **Load by ID** - Restore specific conversations
- **List conversations** - Browse all saved conversations

## Loading Conversations

### Load by ID

```csharp
// Load specific conversation
await agent.LoadConversationAsync("conv_abc123");

// Now agent uses this conversation
await agent.SendAsync("Continue where we left off");
```

### Load with Error Handling

```csharp
try
{
    await agent.LoadConversationAsync(conversationId);
    Debug.Log($"Loaded: {agent.Conversation.Metadata.Title}");
}
catch (FileNotFoundException)
{
    Debug.LogWarning("Conversation not found, creating new one");
    await agent.CreateNewConversationAsync();
}
catch (Exception ex)
{
    Debug.LogError($"Load failed: {ex.Message}");
}
```

### Load Last Conversation

```csharp
// Save ID on app quit
void OnApplicationQuit()
{
    if (agent.Conversation != null)
    {
        PlayerPrefs.SetString("LastConversation", agent.ConversationId);
    }
}

// Restore on start
async void Start()
{
    string lastId = PlayerPrefs.GetString("LastConversation");
    if (!string.IsNullOrEmpty(lastId))
    {
        try
        {
            await agent.LoadConversationAsync(lastId);
        }
        catch
        {
            // Create new if load fails
            await agent.CreateNewConversationAsync();
        }
    }
}
```

## Saving Conversations

### Auto-Save

```csharp
// Auto-save enabled by default with persistent stores
agent.ConversationStore = ConversationStoreType.LocalFile;

// Automatically saved after each message
await agent.SendAsync("Hello!");
// Saved to disk automatically
```

### Manual Save

```csharp
// Update metadata
agent.Conversation.Metadata.Title = "Updated Title";
agent.Conversation.Metadata.Tags = new[] { "updated" };

// Save changes
await agent.SaveConversationAsync();
```

### Save with Confirmation

```csharp
public async UniTask<bool> SaveWithConfirmation()
{
    try
    {
        await agent.SaveConversationAsync();
        Debug.Log("✓ Conversation saved");
        return true;
    }
    catch (Exception ex)
    {
        Debug.LogError($"✗ Save failed: {ex.Message}");
        return false;
    }
}
```

## Listing Conversations

### Get All Conversations

```csharp
// List all saved conversations
Conversation[] conversations = await agent.ListConversationsAsync();

foreach (var conv in conversations)
{
    Debug.Log($"{conv.Metadata.Title} - {conv.Messages.Count} messages");
}
```

### Display in UI

```csharp
public class ConversationList : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private ConversationListItem itemPrefab;
    [SerializeField] private Transform listContainer;
    
    public async void RefreshList()
    {
        // Clear existing
        foreach (Transform child in listContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Load conversations
        var conversations = await agent.ListConversationsAsync();
        
        // Sort by last activity
        var sorted = conversations
            .OrderByDescending(c => c.Metadata.UpdatedAt)
            .ToArray();
        
        // Create UI items
        foreach (var conv in sorted)
        {
            var item = Instantiate(itemPrefab, listContainer);
            item.Setup(conv, OnConversationSelected);
        }
    }
    
    async void OnConversationSelected(Conversation conversation)
    {
        await agent.LoadConversationAsync(conversation.Id);
        // Navigate to chat view
    }
}
```

### Filter by Tags

```csharp
var conversations = await agent.ListConversationsAsync();

// Filter by tag
var supportChats = conversations
    .Where(c => c.Metadata.Tags?.Contains("support") == true)
    .ToArray();

// Filter by custom data
var urgentChats = conversations
    .Where(c => c.Metadata.CustomData?.ContainsKey("priority") == true &&
                c.Metadata.CustomData["priority"].ToString() == "high")
    .ToArray();
```

## Storage Locations

### Local File Storage

```csharp
agent.ConversationStore = ConversationStoreType.LocalFile;

// Saved to:
// Windows: C:\Users\{User}\AppData\LocalLow\{Company}\{Product}\Conversations\
// Mac: ~/Library/Application Support/{Company}/{Product}/Conversations/
// Android: /data/data/{package}/files/Conversations/
// iOS: /var/mobile/Containers/Data/Application/{guid}/Documents/Conversations/

string path = Path.Combine(
    Application.persistentDataPath,
    "Conversations",
    $"{conversationId}.json"
);
```

### Cloud Storage (Threads API)

```csharp
agent.ConversationStore = ConversationStoreType.ThreadsApi;

// Stored on OpenAI servers
// Accessible from any device with same API key
```

### Custom Storage

```csharp
agent.ConversationStore = ConversationStoreType.Custom;

// Implement your own storage logic
agent.onConversationSaved += async (conversation) =>
{
    // Save to your database
    await MyDatabase.SaveConversationAsync(conversation);
};

agent.onConversationLoaded += async (conversationId) =>
{
    // Load from your database
    return await MyDatabase.LoadConversationAsync(conversationId);
};
```

## Migration Between Stores

### Export to JSON

```csharp
public async UniTask ExportConversation(string conversationId)
{
    await agent.LoadConversationAsync(conversationId);
    
    string json = JsonConvert.SerializeObject(
        agent.Conversation,
        Formatting.Indented
    );
    
    string path = Path.Combine(
        Application.persistentDataPath,
        $"export_{conversationId}.json"
    );
    
    File.WriteAllText(path, json);
    Debug.Log($"Exported to: {path}");
}
```

### Import from JSON

```csharp
public async UniTask ImportConversation(string jsonPath)
{
    string json = File.ReadAllText(jsonPath);
    
    Conversation conversation = JsonConvert.DeserializeObject<Conversation>(json);
    
    // Create new conversation with imported data
    var newConv = await agent.CreateNewConversationAsync(conversation.Metadata);
    
    // Copy messages
    foreach (var message in conversation.Messages)
    {
        newConv.Messages.Add(message);
    }
    
    // Save
    await agent.SaveConversationAsync();
}
```

## Backup & Restore

### Create Backup

```csharp
public async UniTask BackupAllConversations()
{
    var conversations = await agent.ListConversationsAsync();
    
    var backup = new ConversationBackup
    {
        Timestamp = DateTime.UtcNow,
        Conversations = conversations
    };
    
    string json = JsonConvert.SerializeObject(backup, Formatting.Indented);
    
    string path = Path.Combine(
        Application.persistentDataPath,
        $"backup_{DateTime.Now:yyyyMMdd_HHmmss}.json"
    );
    
    File.WriteAllText(path, json);
    Debug.Log($"Backup created: {path}");
}

[Serializable]
public class ConversationBackup
{
    public DateTime Timestamp;
    public Conversation[] Conversations;
}
```

### Restore from Backup

```csharp
public async UniTask RestoreFromBackup(string backupPath)
{
    string json = File.ReadAllText(backupPath);
    var backup = JsonConvert.DeserializeObject<ConversationBackup>(json);
    
    foreach (var conversation in backup.Conversations)
    {
        // Check if already exists
        try
        {
            await agent.LoadConversationAsync(conversation.Id);
            Debug.Log($"Skipping existing: {conversation.Id}");
            continue;
        }
        catch
        {
            // Doesn't exist, restore it
        }
        
        // Create and save
        var newConv = await agent.CreateNewConversationAsync(conversation.Metadata);
        newConv.Messages.AddRange(conversation.Messages);
        await agent.SaveConversationAsync();
        
        Debug.Log($"Restored: {conversation.Metadata.Title}");
    }
}
```

## Performance Tips

### 1. Lazy Loading

```csharp
// Only load conversation metadata, not full messages
public class ConversationMetadataOnly
{
    public string Id;
    public ConversationMetadata Metadata;
    // Don't load Messages until needed
}
```

### 2. Pagination

```csharp
public async UniTask<Conversation[]> ListConversationsPagedAsync(
    int page,
    int pageSize)
{
    var all = await agent.ListConversationsAsync();
    
    return all
        .OrderByDescending(c => c.Metadata.UpdatedAt)
        .Skip(page * pageSize)
        .Take(pageSize)
        .ToArray();
}
```

### 3. Cache Recently Used

```csharp
private Dictionary<string, Conversation> conversationCache = new();

public async UniTask<Conversation> GetConversationCached(string id)
{
    if (conversationCache.TryGetValue(id, out var cached))
    {
        return cached;
    }
    
    await agent.LoadConversationAsync(id);
    conversationCache[id] = agent.Conversation;
    
    return agent.Conversation;
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using System;
using System.Linq;

public class ConversationPersistence : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void Start()
    {
        // Setup storage
        agent.ConversationStore = ConversationStoreType.LocalFile;
        
        // Try to load last conversation
        await LoadLastConversation();
    }
    
    async UniTask LoadLastConversation()
    {
        string lastId = PlayerPrefs.GetString("LastConversation");
        
        if (string.IsNullOrEmpty(lastId))
        {
            Debug.Log("No previous conversation, creating new one");
            await agent.CreateNewConversationAsync();
            return;
        }
        
        try
        {
            await agent.LoadConversationAsync(lastId);
            Debug.Log($"✓ Loaded: {agent.Conversation.Metadata.Title}");
            Debug.Log($"  Messages: {agent.Messages.Count}");
        }
        catch (Exception ex)
        {
            Debug.LogWarning($"Failed to load last conversation: {ex.Message}");
            await agent.CreateNewConversationAsync();
        }
    }
    
    void OnApplicationQuit()
    {
        // Save current conversation ID
        if (agent.Conversation != null)
        {
            PlayerPrefs.SetString("LastConversation", agent.ConversationId);
            PlayerPrefs.Save();
        }
    }
    
    public async void SaveConversation()
    {
        try
        {
            await agent.SaveConversationAsync();
            Debug.Log("✓ Conversation saved");
        }
        catch (Exception ex)
        {
            Debug.LogError($"✗ Save failed: {ex.Message}");
        }
    }
    
    public async void ShowConversationList()
    {
        var conversations = await agent.ListConversationsAsync();
        
        Debug.Log($"Found {conversations.Length} conversations:");
        foreach (var conv in conversations.OrderByDescending(c => c.Metadata.UpdatedAt))
        {
            Debug.Log($"  • {conv.Metadata.Title} ({conv.Messages.Count} messages)");
        }
    }
}
```

## Next Steps

- [Creating Conversations](creating-conversations.md)
- [Messages & Items](messages-and-items.md)
- [Conversation Store](../configuration/conversation-store.md)
