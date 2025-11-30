# Agent Services

Build service-oriented architecture for AI agents.

## Overview

Agent Services provide:

- Service layer architecture
- Dependency injection
- Service registration
- Lifecycle management
- Cross-cutting concerns

## Basic Concepts

### Service Architecture

```csharp
// Agent uses services through dependency injection
public class Agent
{
    private readonly IConversationService conversationService;
    private readonly IToolService toolService;
    private readonly IAudioService audioService;
    
    public Agent(
        IConversationService conversationService,
        IToolService toolService,
        IAudioService audioService)
    {
        this.conversationService = conversationService;
        this.toolService = toolService;
        this.audioService = audioService;
    }
}
```

## Service Interfaces

### Define Service Contracts

```csharp
public interface IAgentService
{
    void Initialize();
    void Shutdown();
    bool IsInitialized { get; }
}

public interface IConversationService : IAgentService
{
    Conversation CreateConversation();
    Task<Conversation> LoadAsync(string id);
    Task SaveAsync(Conversation conversation);
    Task<AgentResponse> SendMessageAsync(string message);
}

public interface IToolService : IAgentService
{
    void RegisterExecutor(IToolExecutor executor);
    Task<string> ExecuteToolAsync(ToolCall toolCall);
    List<ToolDefinition> GetAvailableTools();
}

public interface IAudioService : IAgentService
{
    Task<AudioClip> RecordAsync();
    Task PlayAsync(AudioClip clip);
    Task<string> TranscribeAsync(AudioClip clip);
    Task<AudioClip> GenerateSpeechAsync(string text);
}
```

## Service Implementation

### Conversation Service

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ConversationService : IConversationService
{
    private readonly Agent agent;
    private bool isInitialized;
    
    public bool IsInitialized => isInitialized;
    
    public ConversationService(Agent agent)
    {
        this.agent = agent;
    }
    
    public void Initialize()
    {
        Debug.Log("✓ Conversation service initialized");
        isInitialized = true;
    }
    
    public void Shutdown()
    {
        Debug.Log("Conversation service shut down");
        isInitialized = false;
    }
    
    public Conversation CreateConversation()
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        return new Conversation
        {
            Id = Guid.NewGuid().ToString(),
            CreatedAt = DateTime.Now,
            Messages = new List<ConversationMessage>()
        };
    }
    
    public async Task<Conversation> LoadAsync(string id)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        string path = Path.Combine(
            Application.persistentDataPath,
            "Conversations",
            $"{id}.json"
        );
        
        if (!File.Exists(path))
            return null;
        
        string json = await File.ReadAllTextAsync(path);
        return JsonUtility.FromJson<Conversation>(json);
    }
    
    public async Task SaveAsync(Conversation conversation)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        string json = JsonUtility.ToJson(conversation, true);
        
        string path = Path.Combine(
            Application.persistentDataPath,
            "Conversations",
            $"{conversation.Id}.json"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(path));
        await File.WriteAllTextAsync(path, json);
        
        Debug.Log($"💾 Conversation saved: {conversation.Id}");
    }
    
    public async Task<AgentResponse> SendMessageAsync(string message)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        // Add user message to conversation
        agent.ConversationController.CurrentConversation.AddMessage(
            ConversationRole.User,
            message
        );
        
        // Send to API
        AgentResponse response = await agent.SendAsync();
        
        return response;
    }
}
```

### Tool Service

```csharp
public class ToolService : IToolService
{
    private readonly Agent agent;
    private Dictionary<string, IToolExecutor> executors = new();
    private bool isInitialized;
    
    public bool IsInitialized => isInitialized;
    
    public ToolService(Agent agent)
    {
        this.agent = agent;
    }
    
    public void Initialize()
    {
        Debug.Log("✓ Tool service initialized");
        isInitialized = true;
    }
    
    public void Shutdown()
    {
        executors.Clear();
        Debug.Log("Tool service shut down");
        isInitialized = false;
    }
    
    public void RegisterExecutor(IToolExecutor executor)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        executors[executor.ToolName] = executor;
        Debug.Log($"Registered tool executor: {executor.ToolName}");
    }
    
    public async Task<string> ExecuteToolAsync(ToolCall toolCall)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        if (!executors.TryGetValue(toolCall.FunctionName, out var executor))
        {
            throw new InvalidOperationException($"No executor found for: {toolCall.FunctionName}");
        }
        
        Debug.Log($"🔧 Executing tool: {toolCall.FunctionName}");
        
        string result = await executor.ExecuteAsync(toolCall.Arguments);
        
        Debug.Log($"✓ Tool completed: {toolCall.FunctionName}");
        
        return result;
    }
    
    public List<ToolDefinition> GetAvailableTools()
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        return executors.Values
            .Select(e => e.GetToolDefinition())
            .ToList();
    }
}
```

## Service Registry

### Register and Resolve Services

```csharp
public class ServiceRegistry
{
    private static ServiceRegistry instance;
    public static ServiceRegistry Instance => instance ??= new ServiceRegistry();
    
    private Dictionary<Type, object> services = new();
    
    public void Register<TService>(TService service) where TService : IAgentService
    {
        services[typeof(TService)] = service;
        Debug.Log($"✓ Service registered: {typeof(TService).Name}");
    }
    
    public TService Resolve<TService>() where TService : IAgentService
    {
        if (services.TryGetValue(typeof(TService), out var service))
        {
            return (TService)service;
        }
        
        throw new InvalidOperationException($"Service not registered: {typeof(TService).Name}");
    }
    
    public bool TryResolve<TService>(out TService service) where TService : IAgentService
    {
        if (services.TryGetValue(typeof(TService), out var obj))
        {
            service = (TService)obj;
            return true;
        }
        
        service = default;
        return false;
    }
    
    public void InitializeAll()
    {
        foreach (var service in services.Values.Cast<IAgentService>())
        {
            service.Initialize();
        }
        
        Debug.Log($"✓ Initialized {services.Count} services");
    }
    
    public void ShutdownAll()
    {
        foreach (var service in services.Values.Cast<IAgentService>())
        {
            service.Shutdown();
        }
        
        services.Clear();
        Debug.Log("All services shut down");
    }
}
```

## Service Container

### Dependency Injection Container

```csharp
public class AgentServiceContainer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private ServiceRegistry registry;
    
    void Awake()
    {
        registry = ServiceRegistry.Instance;
        RegisterServices();
        InitializeServices();
    }
    
    void RegisterServices()
    {
        // Register core services
        registry.Register<IConversationService>(new ConversationService(agent.Agent));
        registry.Register<IToolService>(new ToolService(agent.Agent));
        registry.Register<IAudioService>(new AudioService(agent.Agent));
        
        // Register custom services
        registry.Register<IAnalyticsService>(new AnalyticsService());
        registry.Register<IStorageService>(new StorageService());
        
        Debug.Log("✓ All services registered");
    }
    
    void InitializeServices()
    {
        registry.InitializeAll();
    }
    
    void OnDestroy()
    {
        registry.ShutdownAll();
    }
}
```

## Custom Services

### Analytics Service

```csharp
public interface IAnalyticsService : IAgentService
{
    void TrackEvent(string eventName, Dictionary<string, object> properties = null);
    void TrackError(string error);
}

public class AnalyticsService : IAnalyticsService
{
    private bool isInitialized;
    
    public bool IsInitialized => isInitialized;
    
    public void Initialize()
    {
        Debug.Log("✓ Analytics service initialized");
        isInitialized = true;
    }
    
    public void Shutdown()
    {
        Debug.Log("Analytics service shut down");
        isInitialized = false;
    }
    
    public void TrackEvent(string eventName, Dictionary<string, object> properties = null)
    {
        if (!isInitialized)
            return;
        
        Debug.Log($"📊 Event: {eventName}");
        
        if (properties != null)
        {
            foreach (var kvp in properties)
            {
                Debug.Log($"  {kvp.Key}: {kvp.Value}");
            }
        }
        
        // Send to analytics backend
    }
    
    public void TrackError(string error)
    {
        if (!isInitialized)
            return;
        
        Debug.LogError($"📊 Error tracked: {error}");
        
        // Send to error tracking service
    }
}
```

### Storage Service

```csharp
public interface IStorageService : IAgentService
{
    Task SaveAsync<T>(string key, T data);
    Task<T> LoadAsync<T>(string key);
    Task<bool> ExistsAsync(string key);
    Task DeleteAsync(string key);
}

public class StorageService : IStorageService
{
    private string basePath;
    private bool isInitialized;
    
    public bool IsInitialized => isInitialized;
    
    public void Initialize()
    {
        basePath = Path.Combine(Application.persistentDataPath, "Storage");
        Directory.CreateDirectory(basePath);
        
        Debug.Log($"✓ Storage service initialized: {basePath}");
        isInitialized = true;
    }
    
    public void Shutdown()
    {
        Debug.Log("Storage service shut down");
        isInitialized = false;
    }
    
    public async Task SaveAsync<T>(string key, T data)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        string json = JsonUtility.ToJson(data, true);
        string path = Path.Combine(basePath, $"{key}.json");
        
        await File.WriteAllTextAsync(path, json);
        
        Debug.Log($"💾 Saved: {key}");
    }
    
    public async Task<T> LoadAsync<T>(string key)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        string path = Path.Combine(basePath, $"{key}.json");
        
        if (!File.Exists(path))
            return default;
        
        string json = await File.ReadAllTextAsync(path);
        return JsonUtility.FromJson<T>(json);
    }
    
    public Task<bool> ExistsAsync(string key)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        string path = Path.Combine(basePath, $"{key}.json");
        return Task.FromResult(File.Exists(path));
    }
    
    public Task DeleteAsync(string key)
    {
        if (!isInitialized)
            throw new InvalidOperationException("Service not initialized");
        
        string path = Path.Combine(basePath, $"{key}.json");
        
        if (File.Exists(path))
        {
            File.Delete(path);
            Debug.Log($"🗑️ Deleted: {key}");
        }
        
        return Task.CompletedTask;
    }
}
```

## Service Usage

### Using Services in Code

```csharp
public class AgentManager : MonoBehaviour
{
    private IConversationService conversationService;
    private IToolService toolService;
    private IAnalyticsService analyticsService;
    
    void Start()
    {
        // Resolve services
        var registry = ServiceRegistry.Instance;
        
        conversationService = registry.Resolve<IConversationService>();
        toolService = registry.Resolve<IToolService>();
        analyticsService = registry.Resolve<IAnalyticsService>();
        
        Debug.Log("✓ Services resolved");
    }
    
    public async void SendMessage(string message)
    {
        // Track event
        analyticsService.TrackEvent("message_sent", new Dictionary<string, object>
        {
            { "length", message.Length }
        });
        
        // Send message
        AgentResponse response = await conversationService.SendMessageAsync(message);
        
        Debug.Log($"Response: {response.TextContent}");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentServiceManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private ServiceRegistry registry;
    private IConversationService conversationService;
    private IToolService toolService;
    private IAnalyticsService analyticsService;
    private IStorageService storageService;
    
    void Awake()
    {
        InitializeServices();
    }
    
    void InitializeServices()
    {
        registry = ServiceRegistry.Instance;
        
        // Register services
        registry.Register<IConversationService>(new ConversationService(agent.Agent));
        registry.Register<IToolService>(new ToolService(agent.Agent));
        registry.Register<IAudioService>(new AudioService(agent.Agent));
        registry.Register<IAnalyticsService>(new AnalyticsService());
        registry.Register<IStorageService>(new StorageService());
        
        // Initialize all
        registry.InitializeAll();
        
        // Resolve services
        conversationService = registry.Resolve<IConversationService>();
        toolService = registry.Resolve<IToolService>();
        analyticsService = registry.Resolve<IAnalyticsService>();
        storageService = registry.Resolve<IStorageService>();
        
        Debug.Log("✓ Agent services initialized");
    }
    
    public async void SaveConversation(string id)
    {
        var conversation = agent.ConversationController.CurrentConversation;
        
        if (conversation != null)
        {
            await storageService.SaveAsync($"conversation_{id}", conversation);
            
            analyticsService.TrackEvent("conversation_saved", new Dictionary<string, object>
            {
                { "id", id },
                { "messages", conversation.Messages.Count }
            });
        }
    }
    
    public async void LoadConversation(string id)
    {
        var conversation = await storageService.LoadAsync<Conversation>($"conversation_{id}");
        
        if (conversation != null)
        {
            agent.ConversationController.CurrentConversation = conversation;
            
            analyticsService.TrackEvent("conversation_loaded", new Dictionary<string, object>
            {
                { "id", id },
                { "messages", conversation.Messages.Count }
            });
            
            Debug.Log($"✓ Loaded conversation: {conversation.Messages.Count} messages");
        }
    }
    
    void OnDestroy()
    {
        registry.ShutdownAll();
    }
}
```

## Next Steps

- [Controllers](controllers.md)
- [Event Router](event-router.md)
- [Custom Chat Services](custom-chat-services.md)
