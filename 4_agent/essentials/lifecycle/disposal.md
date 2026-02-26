---
icon: trash-can
---

# Disposal

Properly clean up agent resources when done.

## Why Dispose?

Proper disposal:

- Saves conversation state
- Cancels pending requests
- Releases network connections
- Frees memory and resources
- Prevents memory leaks

## Automatic Disposal (AgentBehaviour)

AgentBehaviour handles disposal automatically:

```csharp
public class MyScript : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    // Agent disposes automatically in OnDestroy
    void OnDestroy()
    {
        // No manual cleanup needed
    }
}
```

## Manual Disposal (Agent Class)

### Basic Disposal

```csharp
public class AgentManager
{
    private Agent agent;
    
    public void Shutdown()
    {
        agent?.Dispose();
        agent = null;
    }
}
```

### Proper Disposal Pattern

```csharp
public class AgentManager : IDisposable
{
    private Agent agent;
    private bool disposed = false;
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (disposed) return;
        
        if (disposing)
        {
            // Dispose managed resources
            agent?.Dispose();
            agent = null;
        }
        
        disposed = true;
    }
    
    ~AgentManager()
    {
        Dispose(false);
    }
}
```

## Disposal Process

When you call `Dispose()`, the agent:

1. **Transitions to Terminating** status
2. **Cancels active requests**
3. **Saves conversation** (if configured)
4. **Disposes controllers**
   - ConversationController
   - AudioController
   - ToolCallController
   - McpController
5. **Releases services**
   - Chat API service
   - Audio services
   - Tool services
6. **Clears event subscriptions**
7. **Marks as disposed**

## Safe Disposal

### Check Before Dispose

```csharp
if (agent != null && !agent.IsDisposed)
{
    agent.Dispose();
}
```

### Cancel Before Dispose

```csharp
async UniTask SafeDisposeAsync()
{
    if (agent == null) return;
    
    // Cancel any active request
    if (agent.Status == AgentStatus.Processing)
    {
        await agent.CancelAsync();
    }
    
    // Wait for cancellation to complete
    await UniTask.WaitUntil(() => 
        agent.Status == AgentStatus.Ready || 
        agent.Status == AgentStatus.Cancelling
    );
    
    // Now safe to dispose
    agent.Dispose();
}
```

## Unity Lifecycle Integration

### OnDestroy

```csharp
public class ChatManager : MonoBehaviour
{
    private Agent agent;
    
    void OnDestroy()
    {
        agent?.Dispose();
    }
}
```

### OnApplicationQuit

```csharp
public class AgentController : MonoBehaviour
{
    private Agent agent;
    
    void OnApplicationQuit()
    {
        // Ensure cleanup before app closes
        agent?.Dispose();
    }
}
```

### OnDisable

```csharp
public class AgentPanel : MonoBehaviour
{
    private Agent agent;
    
    void OnDisable()
    {
        // Dispose when panel is hidden
        if (agent != null)
        {
            agent.Dispose();
            agent = null;
        }
    }
}
```

## Graceful Shutdown

### With Progress

```csharp
async UniTask GracefulShutdownAsync()
{
    if (agent == null) return;
    
    try
    {
        // Cancel active work
        if (agent.Status == AgentStatus.Processing)
        {
            ShowMessage("Stopping active request...");
            await agent.CancelAsync();
        }
        
        // Save conversation
        if (agent.Conversation != null)
        {
            ShowMessage("Saving conversation...");
            await agent.SaveConversationAsync();
        }
        
        // Dispose
        ShowMessage("Cleaning up...");
        agent.Dispose();
        
        ShowMessage("Shutdown complete");
    }
    catch (Exception ex)
    {
        Debug.LogError($"Shutdown error: {ex.Message}");
    }
}
```

### With Timeout

```csharp
async UniTask DisposeWithTimeoutAsync(float timeoutSeconds = 5f)
{
    var cts = new CancellationTokenSource(TimeSpan.FromSeconds(timeoutSeconds));
    
    try
    {
        // Try graceful shutdown
        await SafeDisposeAsync(cts.Token);
    }
    catch (OperationCanceledException)
    {
        // Force dispose on timeout
        Debug.LogWarning("Disposal timeout, forcing cleanup");
        agent?.Dispose();
    }
}
```

## Multiple Agents

### Dispose All

```csharp
public class MultiAgentManager : MonoBehaviour
{
    private List<Agent> agents = new();
    
    public void Shutdown()
    {
        foreach (var agent in agents)
        {
            agent?.Dispose();
        }
        
        agents.Clear();
    }
    
    void OnDestroy()
    {
        Shutdown();
    }
}
```

### Dispose Selectively

```csharp
public void DisposeAgent(string agentId)
{
    var agent = agents.Find(a => a.Id == agentId);
    
    if (agent != null)
    {
        agent.Dispose();
        agents.Remove(agent);
    }
}
```

## Conversation Persistence

### Save Before Dispose

```csharp
async UniTask DisposeWithSaveAsync()
{
    if (agent.Conversation != null && agent.Conversation.IsDirty)
    {
        Debug.Log("Saving conversation before disposal...");
        await agent.SaveConversationAsync();
    }
    
    agent.Dispose();
}
```

### Auto-Save Configuration

```csharp
// Configure auto-save
agent.AutoSaveOnDispose = true;

// Dispose will automatically save
agent.Dispose();
```

## Error Handling

### Safe Disposal

```csharp
void SafeDispose()
{
    try
    {
        agent?.Dispose();
    }
    catch (Exception ex)
    {
        Debug.LogError($"Disposal error: {ex.Message}");
    }
    finally
    {
        agent = null;
    }
}
```

### Disposal Exceptions

```csharp
try
{
    agent.Dispose();
}
catch (ObjectDisposedException)
{
    // Already disposed
    Debug.LogWarning("Agent already disposed");
}
catch (InvalidOperationException ex)
{
    // Invalid state for disposal
    Debug.LogError($"Cannot dispose: {ex.Message}");
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentLifecycleManager : MonoBehaviour
{
    [SerializeField] private AgentSettings settings;
    private Agent agent;
    private bool isShuttingDown = false;
    
    async void Start()
    {
        agent = new Agent(settings, CreateBehaviour(), CreateHooks());
        await agent.InitializeAsync();
    }
    
    void OnApplicationQuit()
    {
        // Unity calls this on quit
        GracefulShutdown().Forget();
    }
    
    void OnDestroy()
    {
        // Ensure cleanup
        if (!isShuttingDown)
        {
            agent?.Dispose();
        }
    }
    
    async UniTaskVoid GracefulShutdown()
    {
        if (isShuttingDown) return;
        isShuttingDown = true;
        
        try
        {
            // Cancel active work
            if (agent.Status == AgentStatus.Processing)
            {
                Debug.Log("Cancelling active request...");
                await agent.CancelAsync();
            }
            
            // Save state
            if (agent.Conversation?.IsDirty == true)
            {
                Debug.Log("Saving conversation...");
                await agent.SaveConversationAsync();
            }
            
            // Dispose
            Debug.Log("Disposing agent...");
            agent.Dispose();
            agent = null;
            
            Debug.Log("Shutdown complete");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Shutdown error: {ex.Message}");
        }
    }
    
    // Public method for manual shutdown
    public async void Shutdown()
    {
        await GracefulShutdown();
    }
    
    IAgentBehaviour CreateBehaviour()
    {
        return new AgentBehaviourConfig
        {
            ConversationStoreType = ConversationStoreType.LocalFile,
            Stream = true
        };
    }
    
    AgentHooks CreateHooks()
    {
        return new AgentHooks();
    }
}
```

## Best Practices

### 1. Always Dispose

```csharp
// Good
void OnDestroy()
{
    agent?.Dispose();
}

// Bad - memory leak
void OnDestroy()
{
    // Forgot to dispose
}
```

### 2. Cancel Before Dispose

```csharp
// Good
async UniTask Cleanup()
{
    await agent.CancelAsync();
    agent.Dispose();
}

// Risky - may leave pending requests
agent.Dispose();
```

### 3. Handle Exceptions

```csharp
// Good
try
{
    agent.Dispose();
}
catch (Exception ex)
{
    Debug.LogError($"Disposal error: {ex.Message}");
}

// Bad - exception may crash app
agent.Dispose();
```

### 4. Null After Dispose

```csharp
// Good
agent.Dispose();
agent = null;

// Risky - reference to disposed object
agent.Dispose();
// agent still references disposed object
```

## Next Steps

- [Initialization](initialization.md) - Initialize agent
- [Agent Status](agent-status.md) - Monitor status
- [Conversation Management](../conversation/README.md) - Save conversations
