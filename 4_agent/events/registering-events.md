---
icon: pen-to-square
---

# Registering & Unregistering Events

Learn how to subscribe to agent events and manage event subscriptions properly.

## Basic Registration

### Subscribe to an Event

Use `RegisterEvent<T>()` to subscribe to a specific event type:

```csharp
using Glitch9.AIDevKit.Agents;

Agent agent = new Agent(config, settings);

// Register an event handler
var subscription = agent.RegisterEvent<AgentStatusChanged>(evt => 
{
    Debug.Log($"Agent status changed to: {evt.NewStatus}");
});
```

### Unregister an Event

#### Method 1: Dispose the Subscription (Recommended)

```csharp
var subscription = agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);

// Later, when done
subscription.Dispose();
```

#### Method 2: Explicit Unregistration

```csharp
void OnStatusChanged(AgentStatusChanged evt)
{
    Debug.Log($"Status: {evt.NewStatus}");
}

// Register
agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);

// Unregister
agent.UnregisterEvent<AgentStatusChanged>(OnStatusChanged);
```

## Common Patterns

### Single Event Handler

```csharp
void Start()
{
    agent.RegisterEvent<ConversationCreated>(evt => 
    {
        Debug.Log($"Conversation created: {evt.Conversation.Id}");
    });
}
```

### Multiple Event Handlers

```csharp
private List<IDisposable> subscriptions = new();

void Start()
{
    // Subscribe to multiple events
    subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
    subscriptions.Add(agent.RegisterEvent<ConversationCreated>(OnConversationCreated));
    subscriptions.Add(agent.RegisterEvent<ToolOutputEvent>(OnToolOutput));
    subscriptions.Add(agent.RegisterEvent<Delta<TextData>>(OnTextDelta));
}

void OnDestroy()
{
    // Clean up all subscriptions
    foreach (var subscription in subscriptions)
    {
        subscription.Dispose();
    }
    subscriptions.Clear();
}
```

### Method Reference vs Lambda

Both approaches work:

```csharp
// Method reference
agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);

void OnStatusChanged(AgentStatusChanged evt)
{
    Debug.Log($"Status: {evt.NewStatus}");
}
```

```csharp
// Lambda expression
agent.RegisterEvent<AgentStatusChanged>(evt => 
{
    Debug.Log($"Status: {evt.NewStatus}");
});
```

**Method reference** is better when you need to unregister later:
```csharp
agent.UnregisterEvent<AgentStatusChanged>(OnStatusChanged);  // ✅ Works
```

**Lambda** cannot be unregistered unless you keep a reference:
```csharp
Action<AgentStatusChanged> handler = evt => Debug.Log(evt.NewStatus);
agent.RegisterEvent(handler);
agent.UnregisterEvent(handler);  // ✅ Works
```

## Async Event Handlers

Event handlers can be async:

```csharp
agent.RegisterEvent<ConversationCreated>(async evt => 
{
    Debug.Log("Saving conversation...");
    await SaveConversationToCloudAsync(evt.Conversation);
    Debug.Log("Conversation saved!");
});
```

⚠️ **Important:** The event dispatcher does not wait for async handlers. If you need sequential processing, manage it within your handler.

## Event Handler Lifecycle

### Component-based Lifecycle (Unity)

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using System.Collections.Generic;

public class AgentEventHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    
    private Agent agent;
    private List<IDisposable> subscriptions = new();
    
    void Start()
    {
        agent = agentBehaviour.Agent;
        RegisterEvents();
    }
    
    void RegisterEvents()
    {
        subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
        subscriptions.Add(agent.RegisterEvent<Delta<TextData>>(OnTextDelta));
        subscriptions.Add(agent.RegisterEvent<Usage>(OnUsage));
    }
    
    void OnDestroy()
    {
        UnregisterAllEvents();
    }
    
    void UnregisterAllEvents()
    {
        foreach (var subscription in subscriptions)
        {
            subscription?.Dispose();
        }
        subscriptions.Clear();
    }
    
    void OnStatusChanged(AgentStatusChanged evt)
    {
        Debug.Log($"Status: {evt.NewStatus}");
    }
    
    void OnTextDelta(Delta<TextData> delta)
    {
        if (delta.Value?.Text != null)
            Debug.Log($"Text: {delta.Value.Text}");
    }
    
    void OnUsage(Usage usage)
    {
        Debug.Log($"Tokens used: {usage.TotalTokens}");
    }
}
```

### Class-based Lifecycle

```csharp
using Glitch9.AIDevKit.Agents;
using System;
using System.Collections.Generic;

public class AgentEventManager : IDisposable
{
    private Agent agent;
    private List<IDisposable> subscriptions = new();
    
    public AgentEventManager(Agent agent)
    {
        this.agent = agent;
        RegisterEvents();
    }
    
    void RegisterEvents()
    {
        subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
        subscriptions.Add(agent.RegisterEvent<ToolOutputEvent>(OnToolOutput));
        subscriptions.Add(agent.RegisterEvent<ConversationEvent>(OnConversationEvent));
    }
    
    void OnStatusChanged(AgentStatusChanged evt) { }
    void OnToolOutput(ToolOutputEvent evt) { }
    void OnConversationEvent(ConversationEvent evt) { }
    
    public void Dispose()
    {
        foreach (var subscription in subscriptions)
        {
            subscription?.Dispose();
        }
        subscriptions.Clear();
    }
}

// Usage
using (var manager = new AgentEventManager(agent))
{
    // Use agent with event monitoring
}
// Automatically disposes on exit
```

## Conditional Registration

Register events based on configuration:

```csharp
void RegisterEvents(bool enableLogging, bool trackUsage, bool monitorTools)
{
    if (enableLogging)
    {
        subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
        subscriptions.Add(agent.RegisterEvent<Delta<TextData>>(OnTextDelta));
    }
    
    if (trackUsage)
    {
        subscriptions.Add(agent.RegisterEvent<Usage>(OnUsage));
    }
    
    if (monitorTools)
    {
        subscriptions.Add(agent.RegisterEvent<ToolStatusEvent>(OnToolStatus));
        subscriptions.Add(agent.RegisterEvent<ToolOutputEvent>(OnToolOutput));
    }
}
```

## Error Handling in Event Handlers

Always handle potential errors in your event handlers:

```csharp
agent.RegisterEvent<ToolOutputEvent>(evt => 
{
    try
    {
        ProcessToolOutput(evt);
    }
    catch (Exception ex)
    {
        Debug.LogError($"Error processing tool output: {ex.Message}");
    }
});
```

## Event Types as Event Handlers

You can handle all events of a base type:

```csharp
// ConversationEvent is a base class for multiple event types
agent.RegisterEvent<ConversationEvent>(evt => 
{
    switch (evt)
    {
        case ConversationCreated created:
            Debug.Log($"Created: {created.Conversation.Id}");
            break;
            
        case ConversationLoaded loaded:
            Debug.Log($"Loaded: {loaded.Conversation.Id}");
            break;
            
        case ConversationDeleted deleted:
            Debug.Log($"Deleted: {deleted.ConversationId}");
            break;
    }
});
```

## Best Practices

### ✅ Do

```csharp
// Store subscriptions for cleanup
private List<IDisposable> subscriptions = new();

void Start()
{
    subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
}

void OnDestroy()
{
    foreach (var sub in subscriptions) sub?.Dispose();
    subscriptions.Clear();
}
```

```csharp
// Handle errors within event handlers
agent.RegisterEvent<ToolOutputEvent>(evt => 
{
    try
    {
        ProcessToolOutput(evt);
    }
    catch (Exception ex)
    {
        Debug.LogError($"Error: {ex.Message}");
    }
});
```

```csharp
// Use method references when you need to unregister
agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);
// Later:
agent.UnregisterEvent<AgentStatusChanged>(OnStatusChanged);
```

### ❌ Don't

```csharp
// Don't forget to dispose subscriptions
agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);
// ❌ Memory leak if never disposed
```

```csharp
// Don't block the event thread
agent.RegisterEvent<Delta<TextData>>(evt => 
{
    Thread.Sleep(5000);  // ❌ Blocks all event processing
});
```

```csharp
// Don't unregister lambda without reference
agent.RegisterEvent<AgentStatusChanged>(evt => Debug.Log(evt.NewStatus));
agent.UnregisterEvent<AgentStatusChanged>(/* ❌ Can't reference the lambda */);
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Glitch9.AIDevKit.Conversations;
using Glitch9.IO;
using System;
using System.Collections.Generic;

public class ComprehensiveEventHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    
    private Agent agent;
    private List<IDisposable> subscriptions = new();
    
    void Start()
    {
        agent = agentBehaviour.Agent;
        RegisterAllEvents();
    }
    
    void RegisterAllEvents()
    {
        // Agent lifecycle
        subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
        
        // Conversations
        subscriptions.Add(agent.RegisterEvent<ConversationCreated>(OnConversationCreated));
        subscriptions.Add(agent.RegisterEvent<ConversationLoaded>(OnConversationLoaded));
        subscriptions.Add(agent.RegisterEvent<ConversationDeleted>(OnConversationDeleted));
        
        // Streaming
        subscriptions.Add(agent.RegisterEvent<Delta<TextData>>(OnTextDelta));
        subscriptions.Add(agent.RegisterEvent<Delta<IImagePayload>>(OnImageDelta));
        subscriptions.Add(agent.RegisterEvent<Delta<IAudioDelta>>(OnAudioDelta));
        
        // Tools
        subscriptions.Add(agent.RegisterEvent<ToolStatusEvent>(OnToolStatus));
        subscriptions.Add(agent.RegisterEvent<ToolOutputEvent>(OnToolOutput));
        
        // Metadata
        subscriptions.Add(agent.RegisterEvent<Usage>(OnUsage));
        subscriptions.Add(agent.RegisterEvent<Exception>(OnError));
    }
    
    void OnDestroy()
    {
        foreach (var subscription in subscriptions)
        {
            subscription?.Dispose();
        }
        subscriptions.Clear();
    }
    
    // Event handlers
    void OnStatusChanged(AgentStatusChanged evt)
    {
        Debug.Log($"[STATUS] {evt.NewStatus}" + 
                  (string.IsNullOrEmpty(evt.Args) ? "" : $" ({evt.Args})"));
    }
    
    void OnConversationCreated(ConversationCreated evt)
    {
        Debug.Log($"[CONVERSATION] Created: {evt.Conversation.Id}");
    }
    
    void OnConversationLoaded(ConversationLoaded evt)
    {
        Debug.Log($"[CONVERSATION] Loaded: {evt.Conversation.Id}");
    }
    
    void OnConversationDeleted(ConversationDeleted evt)
    {
        Debug.Log($"[CONVERSATION] Deleted: {evt.ConversationId} " +
                  $"(Success: {evt.Success})");
    }
    
    void OnTextDelta(Delta<TextData> delta)
    {
        if (delta.Value?.Text != null)
        {
            Debug.Log($"[TEXT] {delta.Value.Text}" + 
                      (delta.Done ? " [COMPLETE]" : ""));
        }
    }
    
    void OnImageDelta(Delta<IImagePayload> delta)
    {
        Debug.Log($"[IMAGE] Received image data");
    }
    
    void OnAudioDelta(Delta<IAudioDelta> delta)
    {
        Debug.Log($"[AUDIO] Received audio data");
    }
    
    void OnToolStatus(ToolStatusEvent evt)
    {
        Debug.Log($"[TOOL STATUS] {evt.Type}: {evt.ToolStatus}");
    }
    
    void OnToolOutput(ToolOutputEvent evt)
    {
        Debug.Log($"[TOOL OUTPUT] {evt.Type}: {evt.ToolName}");
    }
    
    void OnUsage(Usage usage)
    {
        Debug.Log($"[USAGE] Prompt: {usage.PromptTokens}, " +
                  $"Completion: {usage.CompletionTokens}, " +
                  $"Total: {usage.TotalTokens}");
    }
    
    void OnError(Exception ex)
    {
        Debug.LogError($"[ERROR] {ex.Message}");
    }
}
```

## Next Steps

- [Available Events](available-events.md) - Complete event reference
- [Overview](overview.md) - Event system architecture
