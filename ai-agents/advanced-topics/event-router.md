# Event Router

Route and manage events across agent subsystems.

## Overview

Event Router provides:

- Centralized event management
- Event routing and filtering
- Pub-sub pattern implementation
- Event prioritization
- Cross-subsystem communication

## Basic Event Router

### Simple Event Bus

```csharp
public class EventBus
{
    private static EventBus instance;
    public static EventBus Instance => instance ??= new EventBus();
    
    private Dictionary<Type, List<Delegate>> subscribers = new();
    
    public void Subscribe<T>(Action<T> handler)
    {
        Type eventType = typeof(T);
        
        if (!subscribers.ContainsKey(eventType))
        {
            subscribers[eventType] = new List<Delegate>();
        }
        
        subscribers[eventType].Add(handler);
        
        Debug.Log($"Subscribed to: {eventType.Name}");
    }
    
    public void Unsubscribe<T>(Action<T> handler)
    {
        Type eventType = typeof(T);
        
        if (subscribers.TryGetValue(eventType, out var handlers))
        {
            handlers.Remove(handler);
            
            if (handlers.Count == 0)
            {
                subscribers.Remove(eventType);
            }
            
            Debug.Log($"Unsubscribed from: {eventType.Name}");
        }
    }
    
    public void Publish<T>(T eventData)
    {
        Type eventType = typeof(T);
        
        if (subscribers.TryGetValue(eventType, out var handlers))
        {
            foreach (var handler in handlers.ToList())
            {
                try
                {
                    ((Action<T>)handler).Invoke(eventData);
                }
                catch (Exception ex)
                {
                    Debug.LogError($"Event handler error: {ex.Message}");
                }
            }
        }
    }
}
```

## Event Definitions

### Agent Events

```csharp
public class AgentInitializedEvent
{
    public Agent Agent { get; set; }
    public DateTime Timestamp { get; set; }
}

public class AgentStartedEvent
{
    public Agent Agent { get; set; }
    public DateTime Timestamp { get; set; }
}

public class AgentStoppedEvent
{
    public Agent Agent { get; set; }
    public string Reason { get; set; }
    public DateTime Timestamp { get; set; }
}

public class AgentErrorEvent
{
    public Agent Agent { get; set; }
    public string Error { get; set; }
    public Exception Exception { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### Message Events

```csharp
public class MessageSentEvent
{
    public ConversationMessage Message { get; set; }
    public DateTime Timestamp { get; set; }
}

public class MessageReceivedEvent
{
    public ConversationMessage Message { get; set; }
    public AgentResponse Response { get; set; }
    public DateTime Timestamp { get; set; }
}

public class TextDeltaEvent
{
    public string Delta { get; set; }
    public string FullText { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### Tool Events

```csharp
public class ToolCallRequestedEvent
{
    public ToolCall ToolCall { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ToolCallCompletedEvent
{
    public ToolCall ToolCall { get; set; }
    public string Result { get; set; }
    public TimeSpan Duration { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ToolCallErrorEvent
{
    public ToolCall ToolCall { get; set; }
    public string Error { get; set; }
    public DateTime Timestamp { get; set; }
}
```

## Event Router Implementation

### Advanced Event Router

```csharp
public class AgentEventRouter
{
    private static AgentEventRouter instance;
    public static AgentEventRouter Instance => instance ??= new AgentEventRouter();
    
    private EventBus eventBus = EventBus.Instance;
    private Agent agent;
    
    public void Initialize(Agent agent)
    {
        this.agent = agent;
        RegisterAgentEvents();
        
        Debug.Log("✓ Event router initialized");
    }
    
    void RegisterAgentEvents()
    {
        // Agent lifecycle
        agent.onAgentInitialized.AddListener(OnAgentInitialized);
        agent.onAgentStarted.AddListener(OnAgentStarted);
        agent.onAgentStopped.AddListener(OnAgentStopped);
        agent.onAgentError.AddListener(OnAgentError);
        
        // Messages
        agent.onMessageAdded.AddListener(OnMessageAdded);
        agent.onResponseReceived.AddListener(OnResponseReceived);
        agent.onTextDelta.AddListener(OnTextDelta);
        
        // Tools
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnAgentInitialized()
    {
        eventBus.Publish(new AgentInitializedEvent
        {
            Agent = agent,
            Timestamp = DateTime.Now
        });
    }
    
    void OnAgentStarted()
    {
        eventBus.Publish(new AgentStartedEvent
        {
            Agent = agent,
            Timestamp = DateTime.Now
        });
    }
    
    void OnAgentStopped(string reason)
    {
        eventBus.Publish(new AgentStoppedEvent
        {
            Agent = agent,
            Reason = reason,
            Timestamp = DateTime.Now
        });
    }
    
    void OnAgentError(string error)
    {
        eventBus.Publish(new AgentErrorEvent
        {
            Agent = agent,
            Error = error,
            Timestamp = DateTime.Now
        });
    }
    
    void OnMessageAdded(ConversationMessage message)
    {
        eventBus.Publish(new MessageSentEvent
        {
            Message = message,
            Timestamp = DateTime.Now
        });
    }
    
    void OnResponseReceived(AgentResponse response)
    {
        eventBus.Publish(new MessageReceivedEvent
        {
            Message = response.Message,
            Response = response,
            Timestamp = DateTime.Now
        });
    }
    
    void OnTextDelta(string delta)
    {
        eventBus.Publish(new TextDeltaEvent
        {
            Delta = delta,
            FullText = agent.LastResponse?.TextContent ?? "",
            Timestamp = DateTime.Now
        });
    }
    
    void OnToolCallRequested(ToolCall toolCall)
    {
        eventBus.Publish(new ToolCallRequestedEvent
        {
            ToolCall = toolCall,
            Timestamp = DateTime.Now
        });
    }
    
    void OnToolCallCompleted(ToolCall toolCall, string result)
    {
        eventBus.Publish(new ToolCallCompletedEvent
        {
            ToolCall = toolCall,
            Result = result,
            Duration = DateTime.Now - toolCall.RequestedAt,
            Timestamp = DateTime.Now
        });
    }
    
    void OnToolCallError(ToolCall toolCall, string error)
    {
        eventBus.Publish(new ToolCallErrorEvent
        {
            ToolCall = toolCall,
            Error = error,
            Timestamp = DateTime.Now
        });
    }
}
```

## Event Filtering

### Filter Events by Type

```csharp
public class EventFilter
{
    private HashSet<Type> allowedTypes = new();
    private HashSet<Type> blockedTypes = new();
    
    public void AllowType<T>()
    {
        allowedTypes.Add(typeof(T));
    }
    
    public void BlockType<T>()
    {
        blockedTypes.Add(typeof(T));
    }
    
    public bool ShouldProcess<T>()
    {
        Type eventType = typeof(T);
        
        // If blocked, don't process
        if (blockedTypes.Contains(eventType))
            return false;
        
        // If allow list is empty, allow all (except blocked)
        if (allowedTypes.Count == 0)
            return true;
        
        // Otherwise, check allow list
        return allowedTypes.Contains(eventType);
    }
}
```

### Filter Events by Criteria

```csharp
public class EventCriteriaFilter
{
    public bool FilterToolEvents(ToolCallCompletedEvent evt)
    {
        // Only process successful tool calls
        return evt.Result != null;
    }
    
    public bool FilterMessageEvents(MessageReceivedEvent evt)
    {
        // Only process long messages
        return evt.Message.Content.Length > 100;
    }
    
    public bool FilterErrorEvents(AgentErrorEvent evt)
    {
        // Only process critical errors
        return evt.Error.Contains("critical") || evt.Error.Contains("fatal");
    }
}
```

## Event Prioritization

### Priority Queue

```csharp
public enum EventPriority
{
    Low = 0,
    Normal = 1,
    High = 2,
    Critical = 3
}

public class PrioritizedEvent
{
    public object EventData { get; set; }
    public EventPriority Priority { get; set; }
    public DateTime Timestamp { get; set; }
}

public class PriorityEventQueue
{
    private List<PrioritizedEvent> queue = new();
    
    public void Enqueue(object eventData, EventPriority priority)
    {
        queue.Add(new PrioritizedEvent
        {
            EventData = eventData,
            Priority = priority,
            Timestamp = DateTime.Now
        });
        
        // Sort by priority (descending), then timestamp (ascending)
        queue = queue
            .OrderByDescending(e => e.Priority)
            .ThenBy(e => e.Timestamp)
            .ToList();
    }
    
    public PrioritizedEvent Dequeue()
    {
        if (queue.Count == 0)
            return null;
        
        var evt = queue[0];
        queue.RemoveAt(0);
        return evt;
    }
    
    public int Count => queue.Count;
}
```

## Event Aggregation

### Aggregate Multiple Events

```csharp
public class EventAggregator
{
    private Dictionary<Type, List<object>> eventBuffer = new();
    private float flushInterval = 1f;
    private float lastFlushTime;
    
    public void BufferEvent<T>(T eventData)
    {
        Type eventType = typeof(T);
        
        if (!eventBuffer.ContainsKey(eventType))
        {
            eventBuffer[eventType] = new List<object>();
        }
        
        eventBuffer[eventType].Add(eventData);
    }
    
    public void Update()
    {
        if (Time.time - lastFlushTime >= flushInterval)
        {
            FlushEvents();
            lastFlushTime = Time.time;
        }
    }
    
    void FlushEvents()
    {
        foreach (var kvp in eventBuffer)
        {
            if (kvp.Value.Count > 0)
            {
                Debug.Log($"Flushing {kvp.Value.Count} events of type {kvp.Key.Name}");
                
                // Process aggregated events
                ProcessAggregatedEvents(kvp.Key, kvp.Value);
                
                kvp.Value.Clear();
            }
        }
    }
    
    void ProcessAggregatedEvents(Type eventType, List<object> events)
    {
        // Handle aggregated events
        if (eventType == typeof(TextDeltaEvent))
        {
            ProcessTextDeltas(events.Cast<TextDeltaEvent>().ToList());
        }
        else if (eventType == typeof(ToolCallCompletedEvent))
        {
            ProcessToolCalls(events.Cast<ToolCallCompletedEvent>().ToList());
        }
    }
    
    void ProcessTextDeltas(List<TextDeltaEvent> events)
    {
        string combined = string.Join("", events.Select(e => e.Delta));
        Debug.Log($"Combined text: {combined}");
    }
    
    void ProcessToolCalls(List<ToolCallCompletedEvent> events)
    {
        float avgDuration = events.Average(e => e.Duration.TotalMilliseconds);
        Debug.Log($"Average tool call duration: {avgDuration:F2}ms");
    }
}
```

## Event Listeners

### Subscribe to Events

```csharp
public class EventListenerExample : MonoBehaviour
{
    void Start()
    {
        var eventBus = EventBus.Instance;
        
        // Subscribe to events
        eventBus.Subscribe<AgentInitializedEvent>(OnAgentInitialized);
        eventBus.Subscribe<MessageReceivedEvent>(OnMessageReceived);
        eventBus.Subscribe<ToolCallCompletedEvent>(OnToolCallCompleted);
        eventBus.Subscribe<AgentErrorEvent>(OnAgentError);
    }
    
    void OnAgentInitialized(AgentInitializedEvent evt)
    {
        Debug.Log($"✓ Agent initialized at {evt.Timestamp}");
    }
    
    void OnMessageReceived(MessageReceivedEvent evt)
    {
        Debug.Log($"📨 Message received: {evt.Message.Content}");
    }
    
    void OnToolCallCompleted(ToolCallCompletedEvent evt)
    {
        Debug.Log($"🔧 Tool completed: {evt.ToolCall.FunctionName} ({evt.Duration.TotalMilliseconds}ms)");
    }
    
    void OnAgentError(AgentErrorEvent evt)
    {
        Debug.LogError($"❌ Agent error: {evt.Error}");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentEventSystem : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private EventBus eventBus;
    private AgentEventRouter eventRouter;
    private EventAggregator eventAggregator;
    
    void Start()
    {
        InitializeEventSystem();
        SubscribeToEvents();
    }
    
    void InitializeEventSystem()
    {
        eventBus = EventBus.Instance;
        eventRouter = AgentEventRouter.Instance;
        eventAggregator = new EventAggregator();
        
        eventRouter.Initialize(agent.Agent);
        
        Debug.Log("✓ Event system initialized");
    }
    
    void SubscribeToEvents()
    {
        eventBus.Subscribe<AgentInitializedEvent>(OnAgentInitialized);
        eventBus.Subscribe<AgentStartedEvent>(OnAgentStarted);
        eventBus.Subscribe<AgentStoppedEvent>(OnAgentStopped);
        eventBus.Subscribe<MessageSentEvent>(OnMessageSent);
        eventBus.Subscribe<MessageReceivedEvent>(OnMessageReceived);
        eventBus.Subscribe<TextDeltaEvent>(OnTextDelta);
        eventBus.Subscribe<ToolCallRequestedEvent>(OnToolCallRequested);
        eventBus.Subscribe<ToolCallCompletedEvent>(OnToolCallCompleted);
        eventBus.Subscribe<AgentErrorEvent>(OnAgentError);
    }
    
    void OnAgentInitialized(AgentInitializedEvent evt)
    {
        Debug.Log("✓ Agent initialized");
    }
    
    void OnAgentStarted(AgentStartedEvent evt)
    {
        Debug.Log("▶️ Agent started");
    }
    
    void OnAgentStopped(AgentStoppedEvent evt)
    {
        Debug.Log($"⏹️ Agent stopped: {evt.Reason}");
    }
    
    void OnMessageSent(MessageSentEvent evt)
    {
        Debug.Log($"📤 Message sent: {evt.Message.Content}");
    }
    
    void OnMessageReceived(MessageReceivedEvent evt)
    {
        Debug.Log($"📥 Message received: {evt.Message.Content}");
    }
    
    void OnTextDelta(TextDeltaEvent evt)
    {
        // Buffer for aggregation
        eventAggregator.BufferEvent(evt);
    }
    
    void OnToolCallRequested(ToolCallRequestedEvent evt)
    {
        Debug.Log($"🔧 Tool requested: {evt.ToolCall.FunctionName}");
    }
    
    void OnToolCallCompleted(ToolCallCompletedEvent evt)
    {
        Debug.Log($"✓ Tool completed: {evt.ToolCall.FunctionName} ({evt.Duration.TotalMilliseconds}ms)");
        
        // Buffer for aggregation
        eventAggregator.BufferEvent(evt);
    }
    
    void OnAgentError(AgentErrorEvent evt)
    {
        Debug.LogError($"❌ Error: {evt.Error}");
    }
    
    void Update()
    {
        eventAggregator.Update();
    }
}
```

## Next Steps

- [Agent Services](agent-services.md)
- [Controllers](controllers.md)
- [Custom Chat Services](custom-chat-services.md)
