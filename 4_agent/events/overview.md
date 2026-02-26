---
icon: eye
---

# Agent Events Overview

Understanding the agent event system architecture and how events flow through your application.

## Architecture

The event system is built on a centralized `EventBus` managed by the `AgentControlHub`. Events flow through a structured pipeline:

```
Agent Action
    ↓
AgentControlHub
    ↓
EventBus.Publish()
    ↓
Registered Handlers
```

### Key Components

#### EventBus
The `EventBus` is a generic publish-subscribe system that:
- Manages typed event subscriptions
- Dispatches events to registered handlers
- Provides automatic type checking
- Returns `IDisposable` subscriptions for cleanup

#### AgentControlHub
The control hub serves as the central dispatcher:
- Receives events from various agent subsystems
- Publishes events to the EventBus
- Also notifies the `IAgentEventListener` interface (if provided)
- Manages agent status transitions

#### IEvent Interface
All events implement the `IEvent` marker interface:
```csharp
public interface IEvent { }
```

This allows the EventBus to handle any event type generically while maintaining type safety.

## Event Flow

### 1. Agent Actions Generate Events

When an agent performs an action, it generates corresponding events:

```csharp
// Example: Agent status changes
public void SetStatus(AgentStatus status, string args = null)
{
    m_StatusHandler.SetStatus(status, args);
    var evt = m_StatusHandler.GetEvent();
    DispatchEvent(evt);  // Publish to EventBus
    m_Listener?.OnAgentStatusChanged(evt);  // Also notify listener
}
```

### 2. Events Are Dispatched

The control hub publishes events to the EventBus:

```csharp
public void DispatchEvent<T>(T e) where T : IEvent
    => m_Events.Publish(e);
```

### 3. Handlers Are Invoked

All registered handlers for that event type are called:

```csharp
agent.RegisterEvent<AgentStatusChanged>(evt => 
{
    // Your handler code
    Debug.Log($"Status: {evt.NewStatus}");
});
```

## Event Listeners vs Event Registration

The agent supports two event notification patterns:

### 1. Event Registration (Recommended)
Generic, type-safe event subscription:

```csharp
// Subscribe to specific events
var sub1 = agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);
var sub2 = agent.RegisterEvent<ToolOutputEvent>(OnToolOutput);

// Dispose when done
sub1.Dispose();
sub2.Dispose();
```

**Advantages:**
- Subscribe to specific events only
- Type-safe event handling
- Easy to unsubscribe
- Supports multiple handlers per event
- No interface implementation required

### 2. IAgentEventListener (Interface-based)
Implement the `IAgentEventListener` interface to receive all agent events:

```csharp
public class MyAgentListener : IAgentEventListener
{
    public void OnAgentStatusChanged(AgentStatusChanged evt) { }
    public void OnConversationEvent(ConversationEvent evt) { }
    public void OnTextDelta(Delta<TextData> e) { }
    public void OnImageDelta(Delta<IImagePayload> e) { }
    public void OnToolOutput(ToolOutputEvent e) { }
    public void OnToolStatus(ToolStatusEvent e) { }
    public void OnUsage(Usage usage) { }
    public void OnError(Exception exception) { }
    // ... implement all interface methods
}

// Pass to agent during construction
var service = new AgentService { EventListener = new MyAgentListener() };
var agent = new Agent(config, settings, service);
```

**Advantages:**
- Receive all events in one place
- Structured interface
- Good for comprehensive monitoring

**Disadvantages:**
- Must implement all interface methods
- Less flexible than selective subscription

## Event Timing

Events are dispatched synchronously on the calling thread. For async operations:

```csharp
agent.RegisterEvent<ConversationCreated>(async evt => 
{
    // Perform async work
    await SaveConversationAsync(evt.Conversation);
});
```

## Event Categories

### Lifecycle Events
Events that track the agent's operational state:
- `AgentStatusChanged` - Status transitions

### Conversation Events
Events related to conversation management:
- Creation, loading, deletion
- Title and summary updates
- Item loading

### Tool Events
Events during tool execution:
- `ToolStatusEvent` - Execution status updates
- `ToolOutputEvent` - Tool results

### Streaming Events (Deltas)
Real-time content updates:
- `Delta<TextData>` - Text streaming
- `Delta<IImagePayload>` - Image streaming
- `Delta<IAudioDelta>` - Audio streaming
- `Delta<Annotation>` - Annotation streaming

### Audio Events
Audio-specific events:
- `AudioBufferStateChanged` - Buffer state transitions
- `AudioRateLimitsUpdated` - Rate limit changes

### Metadata Events
General agent metadata:
- `Usage` - Token usage
- `Exception` - Errors

## Best Practices

### ✅ Do

- Use `RegisterEvent<T>()` for selective event handling
- Dispose subscriptions when no longer needed
- Handle exceptions within event handlers
- Use async handlers when needed
- Check event properties before accessing

### ❌ Don't

- Subscribe to events you don't need
- Forget to dispose event subscriptions
- Throw unhandled exceptions in handlers
- Block the event thread with long operations
- Assume event delivery order

## Example: Event Monitoring System

```csharp
using Glitch9.AIDevKit.Agents;
using Glitch9.AIDevKit.Conversations;
using Glitch9.IO;
using System;
using System.Collections.Generic;

public class AgentEventMonitor : IDisposable
{
    private Agent agent;
    private List<IDisposable> subscriptions = new();
    
    public AgentEventMonitor(Agent agent)
    {
        this.agent = agent;
        RegisterAllEvents();
    }
    
    void RegisterAllEvents()
    {
        // Status monitoring
        subscriptions.Add(agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged));
        
        // Conversation tracking
        subscriptions.Add(agent.RegisterEvent<ConversationCreated>(OnConversationCreated));
        subscriptions.Add(agent.RegisterEvent<ConversationLoaded>(OnConversationLoaded));
        
        // Tool monitoring
        subscriptions.Add(agent.RegisterEvent<ToolOutputEvent>(OnToolOutput));
        subscriptions.Add(agent.RegisterEvent<ToolStatusEvent>(OnToolStatus));
        
        // Streaming content
        subscriptions.Add(agent.RegisterEvent<Delta<TextData>>(OnTextDelta));
        
        // Usage tracking
        subscriptions.Add(agent.RegisterEvent<Usage>(OnUsage));
        
        // Error handling
        subscriptions.Add(agent.RegisterEvent<Exception>(OnError));
    }
    
    void OnStatusChanged(AgentStatusChanged evt)
    {
        Console.WriteLine($"[STATUS] {evt.NewStatus}");
        if (!string.IsNullOrEmpty(evt.Args))
            Console.WriteLine($"  Args: {evt.Args}");
    }
    
    void OnConversationCreated(ConversationCreated evt)
    {
        Console.WriteLine($"[CONVERSATION] Created: {evt.Conversation.Id}");
    }
    
    void OnConversationLoaded(ConversationLoaded evt)
    {
        Console.WriteLine($"[CONVERSATION] Loaded: {evt.Conversation.Id}");
    }
    
    void OnToolOutput(ToolOutputEvent evt)
    {
        Console.WriteLine($"[TOOL] {evt.Type}: {evt.ToolName}");
    }
    
    void OnToolStatus(ToolStatusEvent evt)
    {
        Console.WriteLine($"[TOOL STATUS] {evt.Type}: {evt.ToolStatus}");
    }
    
    void OnTextDelta(Delta<TextData> delta)
    {
        if (delta.Value?.Text != null)
        {
            Console.Write(delta.Value.Text);
            if (delta.Done) Console.WriteLine();
        }
    }
    
    void OnUsage(Usage usage)
    {
        Console.WriteLine($"[USAGE] Prompt: {usage.PromptTokens}, Completion: {usage.CompletionTokens}");
    }
    
    void OnError(Exception ex)
    {
        Console.WriteLine($"[ERROR] {ex.Message}");
    }
    
    public void Dispose()
    {
        foreach (var subscription in subscriptions)
        {
            subscription.Dispose();
        }
        subscriptions.Clear();
    }
}

// Usage
var monitor = new AgentEventMonitor(agent);
// ... use agent ...
monitor.Dispose();
```

## Next Steps

- [Registering Events](registering-events.md) - Event subscription patterns
- [Available Events](available-events.md) - Complete event reference
