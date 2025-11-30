# Controllers

Manage agent subsystems with controller pattern.

## Overview

Controllers provide:

- Subsystem management
- Lifecycle control
- State coordination
- Event handling
- Resource management

## Core Controllers

### Conversation Controller

```csharp
public class ConversationController
{
    private Agent agent;
    private Conversation currentConversation;
    
    public Conversation CurrentConversation
    {
        get => currentConversation;
        set => currentConversation = value;
    }
    
    public ConversationController(Agent agent)
    {
        this.agent = agent;
        currentConversation = new Conversation();
    }
    
    public void AddMessage(ConversationRole role, string content)
    {
        var message = new ConversationMessage
        {
            Role = role,
            Content = content,
            Timestamp = DateTime.Now
        };
        
        currentConversation.Messages.Add(message);
        
        Debug.Log($"Message added: {role} - {content}");
    }
    
    public void ClearMessages()
    {
        currentConversation.Messages.Clear();
        Debug.Log("Conversation cleared");
    }
    
    public List<ConversationMessage> GetMessages()
    {
        return currentConversation.Messages;
    }
    
    public string GetConversationText()
    {
        var sb = new StringBuilder();
        
        foreach (var message in currentConversation.Messages)
        {
            sb.AppendLine($"{message.Role}: {message.Content}");
        }
        
        return sb.ToString();
    }
}
```

### Tool Controller

```csharp
public class ToolController
{
    private Agent agent;
    private Dictionary<string, IToolExecutor> executors = new();
    private List<ToolCall> pendingToolCalls = new();
    
    public ToolController(Agent agent)
    {
        this.agent = agent;
    }
    
    public void RegisterExecutor(IToolExecutor executor)
    {
        executors[executor.ToolName] = executor;
        Debug.Log($"Tool registered: {executor.ToolName}");
    }
    
    public void UnregisterExecutor(string toolName)
    {
        if (executors.Remove(toolName))
        {
            Debug.Log($"Tool unregistered: {toolName}");
        }
    }
    
    public async Task<string> ExecuteToolCallAsync(ToolCall toolCall)
    {
        if (!executors.TryGetValue(toolCall.FunctionName, out var executor))
        {
            throw new InvalidOperationException($"No executor for: {toolCall.FunctionName}");
        }
        
        pendingToolCalls.Add(toolCall);
        
        try
        {
            Debug.Log($"🔧 Executing: {toolCall.FunctionName}");
            
            string result = await executor.ExecuteAsync(toolCall.Arguments);
            
            Debug.Log($"✓ Completed: {toolCall.FunctionName}");
            
            return result;
        }
        finally
        {
            pendingToolCalls.Remove(toolCall);
        }
    }
    
    public List<ToolDefinition> GetToolDefinitions()
    {
        return executors.Values
            .Select(e => e.GetToolDefinition())
            .ToList();
    }
    
    public bool HasPendingToolCalls()
    {
        return pendingToolCalls.Count > 0;
    }
}
```

### Audio Controller

```csharp
public class AudioController
{
    private Agent agent;
    private InputRecorder inputRecorder;
    private OutputPlayer outputPlayer;
    private Transcriber transcriber;
    private TextToSpeech textToSpeech;
    
    public InputRecorder InputRecorder => inputRecorder;
    public OutputPlayer OutputPlayer => outputPlayer;
    public Transcriber Transcriber => transcriber;
    public TextToSpeech TextToSpeech => textToSpeech;
    
    public AudioController(Agent agent)
    {
        this.agent = agent;
        
        inputRecorder = new InputRecorder();
        outputPlayer = new OutputPlayer();
        transcriber = new Transcriber(agent);
        textToSpeech = new TextToSpeech(agent);
    }
    
    public async Task<string> RecordAndTranscribeAsync()
    {
        // Start recording
        inputRecorder.StartRecording();
        
        await UniTask.WaitUntil(() => !inputRecorder.IsRecording);
        
        AudioClip clip = inputRecorder.GetRecordedAudio();
        
        // Transcribe
        string text = await transcriber.TranscribeAsync(clip);
        
        return text;
    }
    
    public async Task GenerateAndPlayAsync(string text)
    {
        // Generate speech
        AudioClip clip = await textToSpeech.GenerateSpeechAsync(text);
        
        // Play audio
        outputPlayer.PlayAudio(clip);
    }
    
    public void StopAll()
    {
        inputRecorder.StopRecording();
        outputPlayer.Stop();
    }
}
```

## Parameter Controller

### Manage Agent Parameters

```csharp
public class ParameterController
{
    private Agent agent;
    private AgentParameters currentParameters;
    
    public AgentParameters CurrentParameters
    {
        get => currentParameters;
        set => currentParameters = value;
    }
    
    public ParameterController(Agent agent)
    {
        this.agent = agent;
        currentParameters = new AgentParameters();
    }
    
    public void SetModel(string model)
    {
        currentParameters.Model = model;
        Debug.Log($"Model set: {model}");
    }
    
    public void SetTemperature(float temperature)
    {
        currentParameters.Temperature = temperature;
        Debug.Log($"Temperature set: {temperature}");
    }
    
    public void SetMaxTokens(int maxTokens)
    {
        currentParameters.MaxTokens = maxTokens;
        Debug.Log($"Max tokens set: {maxTokens}");
    }
    
    public void SetVoice(string voice)
    {
        currentParameters.Voice = voice;
        Debug.Log($"Voice set: {voice}");
    }
    
    public void EnableTools(bool enable)
    {
        currentParameters.ToolsEnabled = enable;
        Debug.Log($"Tools {(enable ? "enabled" : "disabled")}");
    }
    
    public AgentParameters GetParameters()
    {
        return currentParameters;
    }
    
    public void ResetToDefaults()
    {
        currentParameters = new AgentParameters();
        Debug.Log("Parameters reset to defaults");
    }
}
```

## Custom Controllers

### Create Custom Controller

```csharp
public interface IAgentController
{
    void Initialize();
    void Update();
    void Shutdown();
}

public class CustomAgentController : IAgentController
{
    private Agent agent;
    private bool isInitialized;
    
    public CustomAgentController(Agent agent)
    {
        this.agent = agent;
    }
    
    public void Initialize()
    {
        Debug.Log("✓ Custom controller initialized");
        isInitialized = true;
    }
    
    public void Update()
    {
        if (!isInitialized)
            return;
        
        // Update logic
    }
    
    public void Shutdown()
    {
        Debug.Log("Custom controller shut down");
        isInitialized = false;
    }
}
```

### Analytics Controller

```csharp
public class AnalyticsController : IAgentController
{
    private Agent agent;
    private Dictionary<string, int> eventCounts = new();
    private List<AnalyticsEvent> events = new();
    
    public AnalyticsController(Agent agent)
    {
        this.agent = agent;
    }
    
    public void Initialize()
    {
        RegisterEvents();
        Debug.Log("✓ Analytics controller initialized");
    }
    
    void RegisterEvents()
    {
        agent.onAgentInitialized.AddListener(OnAgentInitialized);
        agent.onMessageAdded.AddListener(OnMessageAdded);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
    }
    
    void OnAgentInitialized()
    {
        TrackEvent("agent_initialized");
    }
    
    void OnMessageAdded(ConversationMessage message)
    {
        TrackEvent("message_added", new Dictionary<string, object>
        {
            { "role", message.Role.ToString() },
            { "length", message.Content.Length }
        });
    }
    
    void OnToolCallCompleted(ToolCall toolCall, string result)
    {
        TrackEvent("tool_call_completed", new Dictionary<string, object>
        {
            { "function", toolCall.FunctionName },
            { "duration", (DateTime.Now - toolCall.RequestedAt).TotalMilliseconds }
        });
    }
    
    public void TrackEvent(string eventName, Dictionary<string, object> properties = null)
    {
        // Count events
        if (!eventCounts.ContainsKey(eventName))
        {
            eventCounts[eventName] = 0;
        }
        eventCounts[eventName]++;
        
        // Store event
        events.Add(new AnalyticsEvent
        {
            Name = eventName,
            Properties = properties,
            Timestamp = DateTime.Now
        });
        
        Debug.Log($"📊 Event tracked: {eventName} (#{eventCounts[eventName]})");
    }
    
    public Dictionary<string, int> GetEventCounts()
    {
        return new Dictionary<string, int>(eventCounts);
    }
    
    public void Update()
    {
        // Periodic flush to server
    }
    
    public void Shutdown()
    {
        FlushEvents();
        Debug.Log("Analytics controller shut down");
    }
    
    void FlushEvents()
    {
        Debug.Log($"📊 Flushing {events.Count} events");
        events.Clear();
    }
}

public class AnalyticsEvent
{
    public string Name { get; set; }
    public Dictionary<string, object> Properties { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### Cache Controller

```csharp
public class CacheController : IAgentController
{
    private Agent agent;
    private Dictionary<string, CachedResponse> cache = new();
    private int maxCacheSize = 100;
    
    public CacheController(Agent agent)
    {
        this.agent = agent;
    }
    
    public void Initialize()
    {
        LoadCache();
        Debug.Log("✓ Cache controller initialized");
    }
    
    public bool TryGetCached(string prompt, out string response)
    {
        if (cache.TryGetValue(prompt, out var cached))
        {
            if (!cached.IsExpired())
            {
                response = cached.Response;
                cached.HitCount++;
                
                Debug.Log($"💾 Cache hit: {prompt.Substring(0, Math.Min(50, prompt.Length))}...");
                return true;
            }
            else
            {
                // Remove expired
                cache.Remove(prompt);
            }
        }
        
        response = null;
        return false;
    }
    
    public void CacheResponse(string prompt, string response)
    {
        // Enforce max size
        if (cache.Count >= maxCacheSize)
        {
            RemoveLeastUsed();
        }
        
        cache[prompt] = new CachedResponse
        {
            Response = response,
            CachedAt = DateTime.Now,
            ExpiresAt = DateTime.Now.AddHours(1),
            HitCount = 0
        };
        
        Debug.Log($"💾 Cached response: {prompt.Substring(0, Math.Min(50, prompt.Length))}...");
    }
    
    void RemoveLeastUsed()
    {
        var leastUsed = cache.OrderBy(kvp => kvp.Value.HitCount).First();
        cache.Remove(leastUsed.Key);
        
        Debug.Log($"💾 Removed least used cache entry");
    }
    
    public void ClearCache()
    {
        cache.Clear();
        Debug.Log("💾 Cache cleared");
    }
    
    public void Update()
    {
        // Periodic cleanup
    }
    
    public void Shutdown()
    {
        SaveCache();
        Debug.Log("Cache controller shut down");
    }
    
    void LoadCache()
    {
        // Load from persistent storage
    }
    
    void SaveCache()
    {
        // Save to persistent storage
    }
}

public class CachedResponse
{
    public string Response { get; set; }
    public DateTime CachedAt { get; set; }
    public DateTime ExpiresAt { get; set; }
    public int HitCount { get; set; }
    
    public bool IsExpired()
    {
        return DateTime.Now > ExpiresAt;
    }
}
```

## Controller Manager

### Manage Multiple Controllers

```csharp
public class ControllerManager
{
    private Agent agent;
    private List<IAgentController> controllers = new();
    
    public ControllerManager(Agent agent)
    {
        this.agent = agent;
    }
    
    public void RegisterController(IAgentController controller)
    {
        controllers.Add(controller);
        controller.Initialize();
        
        Debug.Log($"Controller registered: {controller.GetType().Name}");
    }
    
    public T GetController<T>() where T : IAgentController
    {
        return controllers.OfType<T>().FirstOrDefault();
    }
    
    public void UpdateAll()
    {
        foreach (var controller in controllers)
        {
            controller.Update();
        }
    }
    
    public void ShutdownAll()
    {
        foreach (var controller in controllers)
        {
            controller.Shutdown();
        }
        
        controllers.Clear();
        Debug.Log("All controllers shut down");
    }
}
```

## Controller Lifecycle

### Initialize and Shutdown

```csharp
public class AgentWithControllers : MonoBehaviour
{
    private Agent agent;
    private ControllerManager controllerManager;
    
    void Awake()
    {
        agent = new Agent();
        controllerManager = new ControllerManager(agent);
        
        // Register controllers
        controllerManager.RegisterController(new AnalyticsController(agent));
        controllerManager.RegisterController(new CacheController(agent));
        
        Debug.Log("✓ Controllers initialized");
    }
    
    void Update()
    {
        controllerManager.UpdateAll();
    }
    
    void OnDestroy()
    {
        controllerManager.ShutdownAll();
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentControllerSystem : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private ControllerManager controllerManager;
    private ConversationController conversationController;
    private ToolController toolController;
    private AudioController audioController;
    private ParameterController parameterController;
    private AnalyticsController analyticsController;
    private CacheController cacheController;
    
    void Awake()
    {
        InitializeControllers();
    }
    
    void InitializeControllers()
    {
        controllerManager = new ControllerManager(agent.Agent);
        
        // Core controllers
        conversationController = new ConversationController(agent.Agent);
        toolController = new ToolController(agent.Agent);
        audioController = new AudioController(agent.Agent);
        parameterController = new ParameterController(agent.Agent);
        
        // Custom controllers
        analyticsController = new AnalyticsController(agent.Agent);
        cacheController = new CacheController(agent.Agent);
        
        // Register custom controllers
        controllerManager.RegisterController(analyticsController);
        controllerManager.RegisterController(cacheController);
        
        Debug.Log("✓ All controllers initialized");
    }
    
    public async void SendMessageWithCache(string message)
    {
        // Try cache first
        if (cacheController.TryGetCached(message, out string cachedResponse))
        {
            Debug.Log($"Using cached response: {cachedResponse}");
            return;
        }
        
        // Add message
        conversationController.AddMessage(ConversationRole.User, message);
        
        // Send request
        AgentResponse response = await agent.Agent.SendAsync();
        
        // Cache response
        cacheController.CacheResponse(message, response.TextContent);
        
        Debug.Log($"Response: {response.TextContent}");
    }
    
    public void SetParameters(string model, float temperature, int maxTokens)
    {
        parameterController.SetModel(model);
        parameterController.SetTemperature(temperature);
        parameterController.SetMaxTokens(maxTokens);
        
        Debug.Log("✓ Parameters updated");
    }
    
    public void RegisterTool(IToolExecutor executor)
    {
        toolController.RegisterExecutor(executor);
        Debug.Log($"✓ Tool registered: {executor.ToolName}");
    }
    
    public Dictionary<string, int> GetAnalytics()
    {
        return analyticsController.GetEventCounts();
    }
    
    void Update()
    {
        controllerManager.UpdateAll();
    }
    
    void OnDestroy()
    {
        controllerManager.ShutdownAll();
    }
}
```

## Next Steps

- [Agent Services](agent-services.md)
- [Event Router](event-router.md)
- [Custom Chat Services](custom-chat-services.md)
