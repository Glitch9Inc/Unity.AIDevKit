---
icon: brain
---

# Memory Settings

Configure how your agent manages conversation context and memory.

## Overview

Memory settings control:

- How much conversation history to retain
- When to summarize old messages
- Context window management
- Memory persistence

## AgentMemorySettings

```csharp
public class AgentMemorySettings
{
    public int MaxContextMessages { get; set; }
    public int SummarizationThreshold { get; set; }
    public bool EnableSummarization { get; set; }
    public bool EnableMemoryPersistence { get; set; }
    public MemoryCompressionStrategy CompressionStrategy { get; set; }
}
```

## Configuration

### Max Context Messages

Maximum number of messages to include in context:

```csharp
settings.Memory.MaxContextMessages = 50;
```

- **Lower** (10-20) - Faster, cheaper, less context
- **Medium** (30-50) - Balanced
- **Higher** (100+) - More context, slower, expensive

### Summarization

Automatically summarize old messages:

```csharp
settings.Memory.EnableSummarization = true;
settings.Memory.SummarizationThreshold = 100;
```

When conversation exceeds threshold, old messages are summarized to save tokens.

### Compression Strategy

```csharp
public enum MemoryCompressionStrategy
{
    None,           // No compression
    Summarize,      // Summarize old messages
    Prune,          // Remove old messages
    Hybrid          // Mix of both
}
```

Example:

```csharp
settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Hybrid;
```

## Memory Persistence

### Enable Persistence

```csharp
settings.Memory.EnableMemoryPersistence = true;
```

Persists memory across sessions (separate from conversation storage).

### Memory Store

```csharp
public interface IMemoryStore
{
    UniTask SaveAsync(string agentId, Memory memory);
    UniTask<Memory> LoadAsync(string agentId);
    UniTask ClearAsync(string agentId);
}
```

## Context Assembly

### What Goes into Context

1. **System instructions**
2. **Memory/Summary** (if enabled)
3. **Recent messages** (up to MaxContextMessages)
4. **Tool definitions**
5. **Current user message**

### Context Priority

Messages are prioritized:

1. System message (always included)
2. Most recent messages
3. Important messages (marked)
4. Tool results
5. Older messages (summarized or pruned)

## Memory Management Examples

### Minimal Memory (Fast & Cheap)

```csharp
settings.Memory.MaxContextMessages = 10;
settings.Memory.EnableSummarization = false;
settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Prune;
```

Use for:

- Simple Q&A bots
- Stateless interactions
- Cost-sensitive applications

### Moderate Memory (Balanced)

```csharp
settings.Memory.MaxContextMessages = 50;
settings.Memory.EnableSummarization = true;
settings.Memory.SummarizationThreshold = 100;
settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Hybrid;
```

Use for:

- General chat bots
- Customer support
- Most applications

### Extended Memory (Rich Context)

```csharp
settings.Memory.MaxContextMessages = 200;
settings.Memory.EnableSummarization = true;
settings.Memory.SummarizationThreshold = 300;
settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Summarize;
settings.Memory.EnableMemoryPersistence = true;
```

Use for:

- Long conversations
- Complex tasks
- Persistent assistants

## Runtime Memory Control

### Check Memory Usage

```csharp
int messageCount = agent.Messages.Count;
int contextTokens = agent.CurrentContextTokens;

Debug.Log($"Messages: {messageCount}, Tokens: {contextTokens}");
```

### Manual Summarization

```csharp
// Trigger summarization manually
await agent.SummarizeHistoryAsync();
```

### Clear Memory

```csharp
// Clear conversation history
agent.ClearConversation();

// Or keep last N messages
agent.ClearConversation(keepLastN: 10);
```

### Mark Important Messages

```csharp
// Mark message as important (won't be pruned)
agent.MarkMessageAsImportant(messageId);
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class MemoryManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void ConfigureMemory()
    {
        var settings = agent.Settings;
        
        // Configure memory
        settings.Memory.MaxContextMessages = 50;
        settings.Memory.EnableSummarization = true;
        settings.Memory.SummarizationThreshold = 100;
        settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Hybrid;
        settings.Memory.EnableMemoryPersistence = true;
    }
    
    void Start()
    {
        ConfigureMemory();
        
        // Monitor memory usage
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseCompleted(Response response)
    {
        int messageCount = agent.Messages.Count;
        
        // Auto-summarize if getting too long
        if (messageCount > 80)
        {
            SummarizeHistory();
        }
    }
    
    async void SummarizeHistory()
    {
        Debug.Log("Summarizing conversation history...");
        await agent.SummarizeHistoryAsync();
        Debug.Log($"Summarized. Messages: {agent.Messages.Count}");
    }
    
    void OnMemoryFull()
    {
        // Clear old messages but keep recent
        agent.ClearConversation(keepLastN: 20);
    }
}
```

## Token Management

### Estimate Token Usage

```csharp
// Estimate tokens in current context
int estimatedTokens = agent.EstimateContextTokens();

// Check against model limit
int modelLimit = agent.Model.GetContextWindow();
float usage = (float)estimatedTokens / modelLimit;

if (usage > 0.8f)
{
    Debug.LogWarning("Context window nearly full");
}
```

### Token Optimization

```csharp
// Reduce token usage
settings.Memory.MaxContextMessages = 30;           // Fewer messages
settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Prune;  // Aggressive pruning
settings.MaxTokens = 500;                          // Limit response length
```

## Best Practices

### 1. Match Memory to Use Case

Short-term:

```csharp
settings.Memory.MaxContextMessages = 10;
settings.Memory.EnableSummarization = false;
```

Long-term:

```csharp
settings.Memory.MaxContextMessages = 100;
settings.Memory.EnableSummarization = true;
settings.Memory.EnableMemoryPersistence = true;
```

### 2. Monitor and Adjust

```csharp
// Log memory stats periodically
void LogMemoryStats()
{
    Debug.Log($"Messages: {agent.Messages.Count}");
    Debug.Log($"Tokens: {agent.EstimateContextTokens()}");
    Debug.Log($"Summarized: {agent.HasSummarizedHistory}");
}
```

### 3. Handle Memory Pressure

```csharp
if (agent.EstimateContextTokens() > modelLimit * 0.9f)
{
    // Emergency: clear old messages
    agent.ClearConversation(keepLastN: 15);
}
```

### 4. Preserve Important Context

```csharp
// Mark system-critical messages
agent.MarkMessageAsImportant(systemMessageId);

// Or preserve specific topics
agent.PreserveMessagesWithKeyword("order #12345");
```

## Platform-Specific Considerations

### Mobile

Optimize for limited resources:

```csharp
#if UNITY_ANDROID || UNITY_IOS
    settings.Memory.MaxContextMessages = 30;
    settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Prune;
#endif
```

### Desktop

Can afford more memory:

```csharp
#if UNITY_STANDALONE
    settings.Memory.MaxContextMessages = 100;
    settings.Memory.CompressionStrategy = MemoryCompressionStrategy.Summarize;
#endif
```

## Troubleshooting

### Context too long errors

- Reduce MaxContextMessages
- Enable summarization
- Use Prune strategy

### Agent forgetting context

- Increase MaxContextMessages
- Enable memory persistence
- Mark important messages

### High API costs

- Lower MaxContextMessages
- Use aggressive pruning
- Summarize more frequently

## Next Steps

- [Context Assembly](../conversation/context-assembly.md)
- [Messages & Items](../conversation/messages-and-items.md)
- [Creating Conversations](../conversation/creating-conversations.md)
