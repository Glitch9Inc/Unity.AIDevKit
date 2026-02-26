---
icon: brain
---

# Memory (RAG)

Agent's memory system provides intelligent conversation context management through a hybrid approach combining **short-term context windows** and **long-term retrieval** using Retrieval-Augmented Generation (RAG). This enables agents to maintain coherent conversations while efficiently accessing relevant historical information.

## Overview

The memory system consists of two complementary mechanisms:

1. **Short-Term Memory**: Recent messages within a sliding window (configurable via `MaxContextMessages`)
2. **Long-Term Memory (RAG)**: Vector-based retrieval of historically relevant messages using embeddings

When RAG is enabled, the agent automatically:
- Embeds all conversation messages into a vector store
- Retrieves semantically relevant past messages when responding
- Combines short-term context with retrieved long-term memories
- Re-ranks results based on similarity, recency, and thread relevance

## Memory Settings

Configure memory behavior through `AgentMemorySettings`:

```csharp
public class AgentMemorySettings
{
    // Basic Settings
    public bool autoSave = true;                    // Auto-save conversations
    public bool generateTitle = true;                // Generate conversation titles
    public int maxContextMessages = 20;              // Max messages in context (10-100)
    
    // RAG Settings
    public bool useVectorStore = false;             // Enable long-term RAG retrieval
    public string summaryModelId;                    // Model for summarization
    public string embeddingModelId;                  // Model for embeddings
    public int retrievalTopK = 8;                    // Top K results (1-32)
    public float retrievalMinSim = 0.5f;            // Min similarity threshold (0.0-1.0)
}
```

### Basic Settings

#### Auto Save
Automatically persists conversation state to the configured store:

```csharp
var settings = new AgentMemorySettings
{
    autoSave = true  // Conversations save after each message
};
```

#### Generate Title
Automatically generates descriptive titles for conversations:

```csharp
var settings = new AgentMemorySettings
{
    generateTitle = true,
    summaryModelId = DefaultModels.OpenAI_GPT5_Mini
};
```

#### Max Context Messages
Controls the size of the short-term context window:

```csharp
var settings = new AgentMemorySettings
{
    maxContextMessages = 20  // Last 20 messages sent to model
};
```

**Considerations:**
- Higher values provide more context but increase token costs
- Lower values save tokens but may lose important context
- Recommended: 10-50 depending on use case
- RAG can supplement reduced context windows

### RAG Settings

#### Use Vector Store
Enables semantic retrieval of historical messages:

```csharp
var settings = new AgentMemorySettings
{
    useVectorStore = true,
    embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Small
};
```

**When to enable:**
- Long conversations spanning multiple sessions
- Knowledge retention across conversation boundaries
- Agents requiring recall of specific past information
- Applications with large conversation histories

**When to disable:**
- Short, ephemeral conversations
- Performance-critical applications
- Limited embedding API budget

#### Embedding Model
Specifies the model for generating message embeddings:

```csharp
var settings = new AgentMemorySettings
{
    embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Small
};
```

**Available Models:**
- `OpenAI_TextEmbedding_3_Small` - Fast, cost-effective (1536 dimensions)
- `OpenAI_TextEmbedding_3_Large` - Higher accuracy (3072 dimensions)
- Custom embedding models from supported providers

#### Retrieval Top K
Number of relevant messages to retrieve from vector store:

```csharp
var settings = new AgentMemorySettings
{
    retrievalTopK = 8  // Retrieve 8 most relevant messages
};
```

**Tuning Guidelines:**
- **Lower (1-5)**: Focused retrieval, lower token cost, may miss context
- **Medium (6-12)**: Balanced approach, recommended for most cases
- **Higher (13-32)**: Comprehensive retrieval, higher cost, more context

#### Retrieval Min Similarity
Minimum cosine similarity threshold for retrieved messages:

```csharp
var settings = new AgentMemorySettings
{
    retrievalMinSim = 0.5f  // Only messages with >50% similarity
};
```

**Tuning Guidelines:**
- **High (0.7-1.0)**: Strict matching, highly relevant results only
- **Medium (0.5-0.7)**: Balanced relevance (recommended)
- **Low (0.25-0.5)**: Broader retrieval, may include less relevant messages
- **Very Low (<0.25)**: May retrieve noise

## How Memory Works

### Short-Term Context (Without RAG)

When RAG is disabled, the agent uses a simple sliding window:

```
User: "Hello"
Agent: "Hi there!"
User: "What's the weather?"
Agent: "I don't have weather data..."

[Context sent to model: Last 20 messages]
```

**Process:**
1. User sends a message
2. Agent retrieves last `maxContextMessages` messages
3. Messages are sent to the LLM
4. Response is generated and added to history

### Long-Term Retrieval (With RAG)

When RAG is enabled, the system performs hybrid retrieval:

```
User: "What did we discuss about the project last week?"

[System Process]
1. Embed user query
2. Search vector store for similar messages
3. Retrieve top K relevant messages (e.g., 8)
4. Re-rank by similarity + recency + thread
5. Combine: [Summary] + [Short window] + [Retrieved]
6. Send combined context to model

Agent: "Last week we discussed the API integration timeline..."
```

#### Detailed Workflow

**1. Message Embedding**

Every message is automatically embedded and indexed:

```csharp
// Automatic process when message is sent
float[] vector = await EmbedAsync($"{role}: {content}");
await vectorStore.UpsertAsync(messageId, vector, payload);
```

**2. Query Embedding**

When user sends a new message, it's embedded for search:

```csharp
float[] queryVector = await EmbedAsync(userMessage);
```

**3. Vector Search**

Similar messages are retrieved from the vector store:

```csharp
var hits = await vectorStore.SearchAsync(
    queryVector, 
    topK: 8,           // Retrieve 8 results
    minSimilarity: 0.5f // Minimum 50% similarity
);
```

**4. Re-Ranking**

Results are re-ranked using a weighted scoring formula:

```csharp
score = 0.75 × similarity + 0.20 × recency + 0.05 × sameThread
```

**Factors:**
- **Similarity (75%)**: Semantic relevance to query
- **Recency (20%)**: Preference for recent messages
- **Same Thread (5%)**: Bonus for messages in current conversation

**Example Ranking:**

| Message | Similarity | Recency | Same Thread | Final Score |
|---------|-----------|---------|-------------|-------------|
| "API integration timeline" | 0.92 | 0.8 | Yes | 0.91 |
| "Project deadline is Friday" | 0.88 | 0.9 | Yes | 0.90 |
| "Weather is nice today" | 0.85 | 0.95 | No | 0.83 |

**5. Deduplication**

Duplicate messages are removed based on content hash:

```csharp
var seen = new HashSet<int>();
foreach (var msg in longTerm)
{
    int hash = HashCode.Combine(msg.Role, msg.Content?.Count, msg.CreatedAt);
    if (seen.Add(hash)) dedupMessages.Add(msg);
}
```

**6. Context Assembly**

Final context is assembled in order:

```
[Conversation Summary (if available)]
[Short-term context: Last 20 messages]
[Retrieved long-term messages: Top 8 relevant]
[Current user message]
```

This structure provides:
- **Summary**: High-level conversation overview
- **Short-term**: Recent conversation flow
- **Long-term**: Relevant historical information
- **Current**: Immediate query

## Configuration Examples

### Minimal Memory (Token Efficient)

For short conversations or budget constraints:

```csharp
var settings = new AgentMemorySettings
{
    autoSave = false,
    generateTitle = false,
    maxContextMessages = 10,
    useVectorStore = false
};
```

**Characteristics:**
- ✅ Minimal token usage
- ✅ Fast response times
- ❌ Limited context retention
- ❌ No long-term memory

### Balanced Memory (Recommended)

For most production applications:

```csharp
var settings = new AgentMemorySettings
{
    autoSave = true,
    generateTitle = true,
    maxContextMessages = 20,
    useVectorStore = true,
    embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Small,
    retrievalTopK = 8,
    retrievalMinSim = 0.5f
};
```

**Characteristics:**
- ✅ Good context retention
- ✅ Semantic retrieval enabled
- ✅ Reasonable token costs
- ✅ Suitable for most use cases

### Maximum Memory (Knowledge Intensive)

For applications requiring extensive context:

```csharp
var settings = new AgentMemorySettings
{
    autoSave = true,
    generateTitle = true,
    maxContextMessages = 50,
    useVectorStore = true,
    embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Large,
    retrievalTopK = 16,
    retrievalMinSim = 0.25f
};
```

**Characteristics:**
- ✅ Maximum context retention
- ✅ Comprehensive retrieval
- ✅ High-quality embeddings
- ❌ Higher token costs
- ❌ Slower response times

### Custom Configuration

Adapt settings to specific needs:

```csharp
// For technical support agent
var supportSettings = new AgentMemorySettings
{
    maxContextMessages = 30,      // More context for troubleshooting
    useVectorStore = true,
    retrievalTopK = 12,           // Retrieve more historical issues
    retrievalMinSim = 0.6f        // Higher relevance threshold
};

// For creative writing assistant
var creativeSettings = new AgentMemorySettings
{
    maxContextMessages = 15,      // Focus on recent creative flow
    useVectorStore = true,
    retrievalTopK = 6,            // Moderate retrieval
    retrievalMinSim = 0.4f        // Broader context for inspiration
};
```

## Conversation Stores

Memory persistence is handled through `ConversationStoreType`:

### Local File Store

Save conversations to local device storage:

```csharp
var agent = new Agent
{
    conversationStoreType = ConversationStoreType.LocalFile
};
```

**Characteristics:**
- ✅ No external API required
- ✅ Fast local access
- ✅ Full data control
- ❌ Not synchronized across devices
- ❌ Limited to device storage

**Use Cases:**
- Single-player games
- Offline applications
- Development/testing

### Threads API Store (OpenAI)

Use OpenAI's Threads API for conversation persistence:

```csharp
var agent = new Agent
{
    conversationStoreType = ConversationStoreType.ThreadsApi
};
```

**Characteristics:**
- ✅ Cloud synchronized
- ✅ Built-in OpenAI integration
- ✅ Scalable storage
- ❌ Requires OpenAI API
- ❌ Limited to OpenAI ecosystem

**Use Cases:**
- Multi-device applications
- OpenAI-based agents
- Cloud-backed services

### Conversations API Store (OpenAI)

Use OpenAI's newer Conversations API:

```csharp
var agent = new Agent
{
    conversationStoreType = ConversationStoreType.ConversationsApi
};
```

Similar to Threads API with enhanced features.

### Realtime API Store (OpenAI)

For real-time voice/streaming applications:

```csharp
var agent = new Agent
{
    conversationStoreType = ConversationStoreType.RealtimeApi
};
```

**Note:** Specialized for real-time streaming scenarios.

### Custom Store

Implement your own storage backend:

```csharp
public class CustomCloudStore : IConversationStore
{
    public ConversationStoreType StoreType => ConversationStoreType.Custom;
    
    public async UniTask<Conversation> CreateAsync(string agentId, CancellationToken ct)
    {
        // Your custom cloud storage logic
        return await cloudService.CreateConversationAsync(agentId);
    }
    
    // Implement other IConversationStore methods...
}

// Use custom store
var agent = new Agent
{
    conversationStoreType = ConversationStoreType.Custom,
    customConversationStore = new CustomCloudStore()
};
```

**Use Cases:**
- Custom cloud backends (Firebase, AWS, Azure)
- Specialized persistence requirements
- Integration with existing systems

## Memory Management APIs

### Creating Conversations

```csharp
// Create a new conversation
Conversation newConv = await agent.CreateConversationAsync();
Debug.Log($"Created conversation: {newConv.Id}");
```

### Loading Conversations

```csharp
// Load existing conversation
Conversation conv = await agent.LoadConversationAsync("conv_xyz123");

// Load with fallback creation
Conversation conv = await agent.LoadConversationAsync("conv_xyz123", createIfNotFound: true);
```

### Listing Conversations

```csharp
// Get all conversations for this agent
Conversation[] conversations = await agent.ListConversationsAsync();

foreach (var conv in conversations)
{
    Debug.Log($"{conv.Name}: {conv.LastUserMessage}");
}
```

### Saving Conversations

```csharp
// Save conversation metadata
await agent.SaveConversationAsync();

// Save conversation items (messages)
await agent.SaveConversationItemsAsync();
```

**Note:** With `autoSave = true`, these are called automatically.

### Deleting Conversations

```csharp
// Delete a specific conversation
bool success = await agent.DeleteConversationAsync("conv_xyz123");

if (success)
    Debug.Log("Conversation deleted");
```

### Accessing Conversation Data

```csharp
// Current conversation
Conversation current = agent.Conversation;
Debug.Log($"Conversation ID: {current.Id}");
Debug.Log($"Created: {current.CreatedAt}");
Debug.Log($"Updated: {current.UpdatedAt}");

// Conversation items
List<ConversationItem> items = agent.Items;
Debug.Log($"Total items: {items.Count}");

// Messages only
List<Message> messages = agent.Messages;
foreach (var msg in messages)
{
    Debug.Log($"{msg.Role}: {msg.Content}");
}

// Last message
Message last = agent.LastMessage;
Debug.Log($"Last: {last.Content}");
```

## RAG Performance Tuning

### Optimizing Retrieval Quality

**Problem: Irrelevant results**

```csharp
// Solution: Increase similarity threshold
settings.retrievalMinSim = 0.7f;  // Stricter matching
settings.retrievalTopK = 5;       // Fewer, more relevant results
```

**Problem: Missing relevant context**

```csharp
// Solution: Broaden retrieval
settings.retrievalMinSim = 0.4f;  // Lower threshold
settings.retrievalTopK = 12;      // More results
```

**Problem: Too much noise in results**

```csharp
// Solution: Balance quality and quantity
settings.retrievalTopK = 8;
settings.retrievalMinSim = 0.6f;
settings.embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Large;  // Better embeddings
```

### Optimizing Token Usage

**Reduce context window:**

```csharp
settings.maxContextMessages = 10;  // Fewer short-term messages
settings.retrievalTopK = 5;        // Fewer retrieved messages
```

**Smart retrieval:**

```csharp
// Only enable RAG when beneficial
settings.useVectorStore = conversationLength > 50;
```

**Selective embedding:**

```csharp
// Implement custom logic to skip embedding trivial messages
// (e.g., "ok", "thanks", etc.)
```

### Optimizing Response Speed

**Use faster embedding model:**

```csharp
settings.embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Small;  // Faster
```

**Reduce retrieval count:**

```csharp
settings.retrievalTopK = 5;  // Fewer vector searches
```

**Disable RAG for simple queries:**

```csharp
// Custom logic to detect if RAG is needed
if (IsSimpleQuery(userMessage))
{
    // Skip RAG for this message
    settings.useVectorStore = false;
}
```

## Best Practices

### 1. Choose Appropriate Store Types

```csharp
// Single-player game
storeType = ConversationStoreType.LocalFile;

// Multi-platform app with OpenAI
storeType = ConversationStoreType.ThreadsApi;

// Custom backend
storeType = ConversationStoreType.Custom;
```

### 2. Enable RAG for Long Conversations

```csharp
// Enable RAG after conversation grows
if (agent.Messages.Count > 50)
{
    settings.useVectorStore = true;
}
```

### 3. Balance Context Window and RAG

```csharp
// If using RAG, reduce short-term window
settings.useVectorStore = true;
settings.maxContextMessages = 15;  // Rely more on retrieval
settings.retrievalTopK = 10;       // Compensate with retrieval
```

### 4. Monitor Token Usage

```csharp
// Track token costs
agent.OnResponseCompleted += (response) =>
{
    Debug.Log($"Tokens used: {response.Usage.TotalTokens}");
    Debug.Log($"Context size: {agent.Messages.Count}");
};
```

### 5. Implement Smart Caching

```csharp
// Cache embeddings for frequently repeated messages
Dictionary<string, float[]> embeddingCache = new();

float[] GetOrCreateEmbedding(string text)
{
    if (embeddingCache.TryGetValue(text, out var cached))
        return cached;
    
    var embedding = await EmbedAsync(text);
    embeddingCache[text] = embedding;
    return embedding;
}
```

### 6. Handle Vector Store Initialization

```csharp
// Ensure vector store is ready before enabling
try
{
    settings.useVectorStore = true;
    await agent.InitializeAsync();
}
catch (Exception e)
{
    Debug.LogWarning($"Vector store failed: {e.Message}");
    settings.useVectorStore = false;  // Fallback to short-term only
}
```

### 7. Clean Up Old Conversations

```csharp
// Periodically clean up old conversations
async Task CleanupOldConversations()
{
    var conversations = await agent.ListConversationsAsync();
    var cutoffDate = DateTime.UtcNow.AddDays(-30);
    
    foreach (var conv in conversations)
    {
        if (conv.UpdatedAt < cutoffDate)
        {
            await agent.DeleteConversationAsync(conv.Id);
        }
    }
}
```

## Troubleshooting

### RAG Not Working

**Symptom:** Retrieved messages don't seem relevant

**Solutions:**
1. Check embedding model is configured:
   ```csharp
   Debug.Log($"Embedding model: {settings.embeddingModelId}");
   ```

2. Verify vector store is initialized:
   ```csharp
   Debug.Log($"Use vector store: {settings.useVectorStore}");
   ```

3. Adjust similarity threshold:
   ```csharp
   settings.retrievalMinSim = 0.4f;  // Lower threshold
   ```

4. Increase retrieval count:
   ```csharp
   settings.retrievalTopK = 12;  // More results
   ```

### High Token Costs

**Symptom:** Token usage is higher than expected

**Solutions:**
1. Reduce context window:
   ```csharp
   settings.maxContextMessages = 10;
   ```

2. Reduce retrieved messages:
   ```csharp
   settings.retrievalTopK = 5;
   ```

3. Increase similarity threshold:
   ```csharp
   settings.retrievalMinSim = 0.7f;  // Only highly relevant
   ```

4. Disable RAG for short conversations:
   ```csharp
   settings.useVectorStore = agent.Messages.Count > 50;
   ```

### Slow Response Times

**Symptom:** Responses take too long

**Solutions:**
1. Use faster embedding model:
   ```csharp
   settings.embeddingModelId = DefaultModels.OpenAI_TextEmbedding_3_Small;
   ```

2. Reduce retrieval operations:
   ```csharp
   settings.retrievalTopK = 5;
   settings.maxContextMessages = 15;
   ```

3. Disable RAG for simple queries:
   ```csharp
   if (userMessage.Length < 20)
       settings.useVectorStore = false;
   ```

### Conversations Not Persisting

**Symptom:** Conversations don't save between sessions

**Solutions:**
1. Enable auto-save:
   ```csharp
   settings.autoSave = true;
   ```

2. Verify store type is configured:
   ```csharp
   Debug.Log($"Store type: {agent.Conversation.StoreType}");
   ```

3. Manually save conversations:
   ```csharp
   await agent.SaveConversationAsync();
   await agent.SaveConversationItemsAsync();
   ```

4. Check for errors during save:
   ```csharp
   try
   {
       await agent.SaveConversationAsync();
   }
   catch (Exception e)
   {
       Debug.LogError($"Save failed: {e.Message}");
   }
   ```

### Memory Leaks with Large Conversations

**Symptom:** Memory usage grows over time

**Solutions:**
1. Limit conversation history:
   ```csharp
   // Periodically trim old messages
   if (agent.Messages.Count > 1000)
   {
       // Archive and create new conversation
       await agent.CreateConversationAsync();
   }
   ```

2. Implement conversation rotation:
   ```csharp
   // Archive conversation after certain duration
   if (DateTime.UtcNow - agent.Conversation.CreatedAt > TimeSpan.FromHours(24))
   {
       await agent.SaveConversationAsync();
       await agent.CreateConversationAsync();
   }
   ```

3. Clear vector store periodically:
   ```csharp
   // Reset vector store for new conversation
   await vectorStore.ClearAsync();
   ```

## Related Documentation

- [How Agent Works](how-agent-works.md) - Agent architecture and components
- [Conversations](../conversations/README.md) - Managing conversations
- [Agent Events](../events/README.md) - Memory-related events
- [Conversation Stores](../conversations/conversation-stores.md) - Storage backends
