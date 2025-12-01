# Agent vs AgentBehaviour

Understanding the distinction between `Agent` and `AgentBehaviour` is crucial for effective use of AI Dev Kit.

## Comparison Table

| Feature | Agent | AgentBehaviour |
|---------|-------|----------------|
| **Type** | Pure C# class | Unity MonoBehaviour |
| **Unity Dependency** | None | Requires Unity |
| **Initialization** | Manual via constructor | Automatic (Start/Awake) |
| **Configuration** | Code-based | Inspector + Code |
| **Serialization** | Manual | Automatic (Unity) |
| **Lifecycle** | Manual Dispose() | Automatic (OnDestroy) |
| **Scene Integration** | No | Yes (GameObject) |
| **Best For** | Backend, testing, libraries | UI, gameplay, scenes |

## Agent Class

### Characteristics

- **Pure C#** - No Unity dependencies
- **Explicit initialization** - Full control over creation
- **Flexible** - Can be used anywhere in C# code
- **Testable** - Easy to unit test

### Example Usage

```csharp
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentService
{
    private Agent agent;
    
    public async UniTask InitializeAsync(AgentSettings settings)
    {
        agent = new Agent(
            settings: settings,
            behaviour: new CustomBehaviour(),
            hooks: new AgentHooks
            {
                TextDelta = OnTextDelta,
                ResponseCompleted = OnResponseCompleted,
                StatusChanged = OnStatusChanged
            },
            logger: new CustomLogger()
        );
        
        await agent.InitializeAsync();
    }
    
    public async UniTask<Response> SendMessageAsync(string message)
    {
        return await agent.SendAsync(message);
    }
    
    public void Shutdown()
    {
        agent?.Dispose();
    }
    
    private void OnTextDelta(string delta) { }
    private void OnResponseCompleted(Response response) { }
    private void OnStatusChanged(AgentStatus status) { }
}
```

### When to Use Agent

✅ **Good for:**

- Backend services and APIs
- Server-side implementations
- Unit testing
- Non-Unity C# projects
- Custom initialization logic
- Dependency injection patterns

❌ **Not ideal for:**

- Quick Unity scene prototypes
- Inspector-based configuration
- GameObject-attached logic

## AgentBehaviour Class

### Characteristics

- **Unity MonoBehaviour** - Attaches to GameObjects
- **Automatic lifecycle** - Handles Start/Destroy
- **Inspector-friendly** - Drag-and-drop configuration
- **Event-driven** - UnityEvents for UI binding

### Example Usage

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ChatManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Agent auto-initializes
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        agent.onStatusChanged.AddListener(OnStatusChanged);
    }
    
    public async void SendMessage(string message)
    {
        await agent.SendAsync(message);
    }
    
    private void OnTextDelta(string delta)
    {
        // Update UI
    }
    
    private void OnResponseCompleted(Response response)
    {
        // Handle completion
    }
    
    private void OnStatusChanged(AgentStatus status)
    {
        // Update status UI
    }
}
```

### When to Use AgentBehaviour

✅ **Good for:**

- Unity scene-based projects
- Rapid prototyping
- UI development
- GameObject integration
- Inspector configuration
- Tutorial/sample projects

❌ **Not ideal for:**

- Non-Unity projects
- Backend services
- Complex initialization
- Unit testing

## Key Differences

### 1. Initialization

**Agent:**

```csharp
// Explicit construction
var agent = new Agent(
    settings: agentSettings,
    behaviour: behaviourImpl,
    hooks: agentHooks,
    services: agentServices
);

// Explicit initialization
await agent.InitializeAsync();
```

**AgentBehaviour:**

```csharp
// Automatic in Unity lifecycle
void Start()
{
    // Already initialized if AutoInit = true
    // Or manually:
    await agentBehaviour.InitializeAsync();
}
```

### 2. Configuration

**Agent:**

```csharp
// Code-based configuration
var settings = ScriptableObject.CreateInstance<AgentSettings>();
settings.ChatServiceApi = ChatService.ChatCompletions;
settings.Model = "gpt-4";

var agent = new Agent(settings, behaviour);
```

**AgentBehaviour:**

```csharp
// Inspector-based configuration
// Just drag and drop AgentSettings asset
// Configure in Inspector
```

### 3. Cleanup

**Agent:**

```csharp
// Manual cleanup
agent.Dispose();
```

**AgentBehaviour:**

```csharp
// Automatic cleanup
void OnDestroy()
{
    // AgentBehaviour handles this
}
```

### 4. Event Handling

**Agent:**

```csharp
// Code-based hooks
var hooks = new AgentHooks
{
    TextDelta = (delta) => Debug.Log(delta),
    StatusChanged = (status) => UpdateUI(status)
};
```

**AgentBehaviour:**

```csharp
// UnityEvents (can bind in Inspector)
agentBehaviour.onTextDelta.AddListener(OnTextReceived);

// Or in Inspector: drag methods to events
```

## Mixed Usage

You can use both together:

```csharp
public class HybridManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour uiAgent;
    private Agent backgroundAgent;
    
    async void Start()
    {
        // UI agent uses AgentBehaviour
        uiAgent.onTextDelta.AddListener(UpdateChatUI);
        
        // Background agent uses Agent class
        backgroundAgent = new Agent(
            settings: LoadBackgroundSettings(),
            behaviour: new BackgroundBehaviour()
        );
        
        await backgroundAgent.InitializeAsync();
    }
    
    private void OnDestroy()
    {
        backgroundAgent?.Dispose();
    }
}
```

## Architecture Decision Guide

```
Need Unity Inspector? ─┬─ Yes → Use AgentBehaviour
                       └─ No ──┬─ Need GameObjects? ─┬─ Yes → Use AgentBehaviour
                               └─ No ───────────────┴─ Use Agent
```

## Common Patterns

### Pattern 1: UI with AgentBehaviour

```csharp
// Simple chat UI
public class ChatUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public void OnSendClicked(string message)
    {
        agent.SendAsync(message).Forget();
    }
}
```

### Pattern 2: Service with Agent

```csharp
// Backend service
public class AIService
{
    private readonly Agent agent;
    
    public AIService(AgentSettings settings)
    {
        agent = new Agent(settings, new ServiceBehaviour());
    }
    
    public async Task<string> ProcessAsync(string input)
    {
        var response = await agent.SendAsync(input);
        return response.Text;
    }
}
```

### Pattern 3: Mixed Approach

```csharp
// AgentBehaviour in scene, accessed by service
public class GameController : MonoBehaviour
{
    [SerializeField] private AgentBehaviour sceneAgent;
    private NPCService npcService;
    
    void Start()
    {
        // Service accesses Agent from AgentBehaviour
        npcService = new NPCService(sceneAgent.Agent);
    }
}

public class NPCService
{
    private readonly Agent agent;
    
    public NPCService(Agent agent)
    {
        this.agent = agent;
    }
}
```

## Summary

- **Agent** = Pure C# class for maximum flexibility
- **AgentBehaviour** = Unity wrapper for ease of use
- Use **AgentBehaviour** for most Unity projects
- Use **Agent** for backend, testing, or non-Unity code
- Both share the same underlying functionality

## Next Steps

- [Quick Start](quick-start.md) - Create your first agent
- [Agent Configuration](../configuration/README.md) - Configure settings
- [Agent Lifecycle](../lifecycle/README.md) - Understand initialization and disposal
