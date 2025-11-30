# API Reference

Complete API reference for Agent and AgentBehaviour classes.

## Agent Class

### Properties

#### Model Configuration

```csharp
public Model Model { get; set; }
public Model SpeechModel { get; set; }
public Model TranscriptionModel { get; set; }
public Model ImageModel { get; set; }
public Voice Voice { get; set; }
```

#### State

```csharp
public AgentStatus Status { get; }
public bool IsInitialized { get; }
public Response CurrentResponse { get; }
```

#### Conversation

```csharp
public string ConversationId { get; set; }
public Conversation Conversation { get; }
public List<Message> Messages { get; }
public List<ConversationItem> Items { get; }
public Message LastMessage { get; }
```

#### Settings

```csharp
public string Id { get; }
public string Name { get; set; }
public string Instructions { get; }
public ChatService ChatServiceApi { get; }
public float? Temperature { get; }
public int? MaxTokens { get; }
```

#### Audio

```csharp
public float OutputAudioVolume { get; set; }
public SystemLanguage InputAudioLanguage { get; set; }
public bool HasInputAudio { get; }
public bool HasOutputAudio { get; }
```

#### Tools

```csharp
public List<Tool> Tools { get; }
public ToolChoice ToolChoice { get; set; }
public bool HasMcpTool { get; set; }
```

### Methods

#### Lifecycle

```csharp
UniTask InitializeAsync(CancellationToken ct = default);
void Dispose();
```

#### Sending Messages

```csharp
UniTask<Response> SendAsync(string message, CancellationToken ct = default);
UniTask<Response> SendAsync(AudioClip audio, CancellationToken ct = default);
UniTask<Response> SendAsync(string message, Texture2D image, CancellationToken ct = default);
UniTask CancelAsync();
```

#### Conversation Management

```csharp
UniTask<Conversation> CreateNewConversationAsync(ConversationMetadata metadata = null);
UniTask LoadConversationAsync(string conversationId);
UniTask SaveConversationAsync();
UniTask<Conversation[]> ListConversationsAsync();
UniTask DeleteConversationAsync(string conversationId = null);
void ClearConversation(int? keepLastN = null);
```

#### Tool Management

```csharp
void RegisterToolExecutor(IToolExecutor executor);
void RegisterToolExecutors(IEnumerable<IToolExecutor> executors);
void UnregisterToolExecutor(string toolName);
UniTask SubmitToolOutputAsync(string toolCallId, string output);
```

#### Audio

```csharp
UniTask<AudioClip> GenerateSpeechAsync(string text);
UniTask<string> TranscribeAsync(AudioClip audio);
```

### Events

```csharp
event Action<string> onTextDelta;
event Action<Response> onResponseStarted;
event Action<Response> onResponseCompleted;
event Action<AgentStatus> onStatusChanged;
event Action<ToolCall> onToolCallStarted;
event Action<ToolCall, string> onToolCallCompleted;
event Action<ToolCall> onUnhandledToolCall;
event Action<string> onError;
event Action onInputAudioStarted;
event Action<AudioClip> onInputAudioCompleted;
event Action<string> onInputAudioTranscribed;
event Action onOutputAudioStarted;
event Action onOutputAudioCompleted;
```

## AgentBehaviour Class

### Inspector Fields

```csharp
[SerializeField] private AgentSettings settings;
[SerializeField] private ConversationStoreType conversationStore;
[SerializeField] private string conversationId;
[SerializeField] private InputAudioRecorder inputAudioRecorder;
[SerializeField] private OutputAudioPlayer outputAudioPlayer;
[SerializeField] private bool stream;
[SerializeField] private UnhandledToolCallBehaviour unhandledToolCallBehaviour;
[SerializeField] private int submitToolOutputTimeoutSeconds;
[SerializeField] private int mcpApprovalTimeoutSeconds;
```

### Properties

All Agent properties plus:

```csharp
public Agent Agent { get; }
public AgentSettings Settings { get; }
public ConversationStoreType ConversationStore { get; set; }
public bool Stream { get; set; }
public InputAudioRecorder InputAudioRecorder { get; set; }
public OutputAudioPlayer OutputAudioPlayer { get; set; }
```

### Methods

All Agent methods plus:

```csharp
// Unity-specific
void Start();
void OnDestroy();
```

### Unity Events

```csharp
public UnityEvent<string> onTextDelta;
public UnityEvent<Response> onResponseStarted;
public UnityEvent<Response> onResponseCompleted;
public UnityEvent<AgentStatus> onStatusChanged;
public UnityEvent<ToolCall> onToolCallStarted;
public UnityEvent<ToolCall, string> onToolCallCompleted;
public UnityEvent<string> onError;
public UnityEvent onInputAudioStarted;
public UnityEvent<AudioClip> onInputAudioCompleted;
public UnityEvent<string> onInputAudioTranscribed;
public UnityEvent onOutputAudioStarted;
public UnityEvent onOutputAudioCompleted;
```

## Supporting Classes

### AgentSettings

```csharp
public string Id { get; }
public ChatService ChatServiceApi { get; }
public string Model { get; set; }
public string Instructions { get; }
public Temperature Temperature { get; set; }
public TokenCount MaxTokens { get; set; }
public bool EnableInputAudio { get; set; }
public bool EnableOutputAudio { get; set; }
public TranscriptionParameters InputAudioParameters { get; }
public SpeechParameters OutputAudioParameters { get; }
public AgentMemorySettings Memory { get; }
public List<ToolDefinitionBase> ToolDefinitions { get; }
```

### AgentHooks

```csharp
public Action<string> TextDelta { get; set; }
public Action<Response> ResponseStarted { get; set; }
public Action<Response> ResponseCompleted { get; set; }
public Action<AgentStatus> StatusChanged { get; set; }
public ToolCallHooks ToolCall { get; set; }
public IInputAudioRecorder InputAudioRecorder { get; set; }
public IOutputAudioPlayer OutputAudioPlayer { get; set; }
public ConversationHooks Conversation { get; set; }
```

### Response

```csharp
public string Id { get; }
public string Text { get; }
public Role Role { get; }
public List<ToolCall> ToolCalls { get; }
public Usage Usage { get; }
public FinishReason FinishReason { get; }
```

### Message

```csharp
public string Id { get; }
public Role Role { get; }
public string Content { get; }
public List<ContentPart> Parts { get; }
public DateTime Timestamp { get; }
```

## Next Steps

- [Agent Properties](agent-properties.md)
- [Agent Methods](agent-methods.md)
- [AgentBehaviour Properties](agentbehaviour-properties.md)
- [AgentBehaviour Methods](agentbehaviour-methods.md)
