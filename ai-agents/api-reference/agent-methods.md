# Agent Methods

Complete reference for Agent class methods.

## Overview

The `Agent` class provides methods for initialization, conversation management, response generation, tool handling, and lifecycle control.

## Initialization Methods

### InitializeAsync

```csharp
public UniTask InitializeAsync()
```

Initializes the agent, conversation controller, and services.

**Returns:** `UniTask` that completes when initialization finishes.

**Example:**

```csharp
Agent agent = new Agent(settings, behaviour);

await agent.InitializeAsync();

Debug.Log($"Agent initialized: {agent.IsInitialized}");
```

**Throws:**

- `InvalidOperationException` - If already initialized or initializing

## Response Generation Methods

### GenerateResponseAsync (Text)

```csharp
public UniTask<Response> GenerateResponseAsync(
    string text, 
    int index = -1, 
    CancellationToken ct = default)
```

Generates a response from text input.

**Parameters:**

- `text` - User input text
- `index` - Optional message index (-1 for append)
- `ct` - Cancellation token

**Returns:** `Response` object with generated content.

**Example:**

```csharp
Response response = await agent.GenerateResponseAsync("Hello!");

Debug.Log($"Response: {response.TextContent}");
```

### GenerateResponseAsync (Message)

```csharp
public UniTask<Response> GenerateResponseAsync(
    Message message, 
    int index = -1, 
    CancellationToken ct = default)
```

Generates a response from a message object.

**Parameters:**

- `message` - Message to send
- `index` - Optional message index
- `ct` - Cancellation token

**Returns:** `Response` object.

**Example:**

```csharp
var message = new UserMessage("What is the weather?");
message.AddAttachment(imageFile);

Response response = await agent.GenerateResponseAsync(message);
```

### GenerateResponseAsync (ConversationItem)

```csharp
public UniTask<Response> GenerateResponseAsync(
    ConversationItem inputItem, 
    int index = -1, 
    CancellationToken ct = default)
```

Generates a response from any conversation item.

**Parameters:**

- `inputItem` - Conversation item to process
- `index` - Optional item index
- `ct` - Cancellation token

**Returns:** `Response` object.

**Example:**

```csharp
var toolOutput = new ToolOutput(
    callId: "call_123",
    output: "{\"temperature\": 72}"
);

Response response = await agent.GenerateResponseAsync(toolOutput);
```

**Throws:**

- `InvalidOperationException` - If not initialized or already processing

### SubmitToolOutputAsync

```csharp
public UniTask<Response> SubmitToolOutputAsync(
    ToolOutput toolOutput, 
    CancellationToken ct = default)
```

Submits tool output to continue conversation.

**Parameters:**

- `toolOutput` - Tool execution result
- `ct` - Cancellation token

**Returns:** `Response` with agent's follow-up.

**Example:**

```csharp
var toolOutput = new ToolOutput(
    callId: toolCall.Id,
    output: JsonUtility.ToJson(result)
);

Response response = await agent.SubmitToolOutputAsync(toolOutput);

Debug.Log($"Agent response: {response.TextContent}");
```

## Request Control Methods

### CancelRequest

```csharp
public void CancelRequest()
```

Cancels the current request and all ongoing operations.

**Example:**

```csharp
// Start request
var task = agent.GenerateResponseAsync("Long request...");

// Cancel after delay
await UniTask.Delay(1000);
agent.CancelRequest();
```

### CommitParametersUpdateAsync

```csharp
public UniTask CommitParametersUpdateAsync(CancellationToken ct = default)
```

Commits parameter changes to the chat service.

**Parameters:**

- `ct` - Cancellation token

**Returns:** `UniTask` that completes when committed.

**Example:**

```csharp
// Register new tool
toolExecutor.RegisterWith(agent);

// Commit changes
await agent.CommitParametersUpdateAsync();

Debug.Log("Tools updated on server");
```

## Event Callback Methods

### OnResponse

```csharp
public void OnResponse(ResponseStatus status, Response response)
```

Internal callback for response events.

**Parameters:**

- `status` - Response status
- `response` - Response data

### OnTextDelta

```csharp
public void OnTextDelta(TextDelta e)
```

Internal callback for streaming text deltas.

**Parameters:**

- `e` - Text delta event

### OnToolCall

```csharp
public void OnToolCall(ToolCallEvent e)
```

Internal callback for tool call events.

**Parameters:**

- `e` - Tool call event

### OnAudioDelta

```csharp
public void OnAudioDelta(AudioDelta delta)
```

Internal callback for audio streaming.

**Parameters:**

- `delta` - Audio delta data

## Tool Registration Methods

### RegisterTools

```csharp
internal void RegisterTools(List<Tool> tools)
```

Internal method to register tool definitions.

**Parameters:**

- `tools` - List of tools to register

**Note:** Tools are typically registered via `IFunctionCallExecutor` components attached to GameObjects. See Tool Call Handling example below.

```

## Error Handling Methods

### HandleError

```csharp
public void HandleError(Exception error)
```

Internal error handler.

**Parameters:**

- `error` - Exception to handle

### HandleUsage

```csharp
public void HandleUsage(UsageMetadata usage)
```

Internal usage metrics handler.

**Parameters:**

- `usage` - Usage metadata

## Lifecycle Methods

### Dispose

```csharp
public void Dispose()
```

Disposes the agent and releases resources.

**Example:**

```csharp
void OnDestroy()
{
    agent?.Dispose();
}
```

## Complete Examples

### Basic Request Flow

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentBasicExample : MonoBehaviour
{
    async void Start()
    {
        // Create agent
        var settings = AgentSettings.Default;
        var behaviour = new AgentBehaviourConfig();
        
        var agent = new Agent(settings, behaviour);
        
        // Initialize
        await agent.InitializeAsync();
        
        // Send request
        Response response = await agent.GenerateResponseAsync("Hello!");
        
        Debug.Log($"Response: {response.TextContent}");
        
        // Cleanup
        agent.Dispose();
    }
}
```

### Streaming Response

```csharp
public class StreamingExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    [SerializeField] private TMP_Text outputText;
    
    void Start()
    {
        var agent = agentBehaviour.Agent;
        
        // Subscribe to text deltas
        agent.hooks.TextDelta = new TextDeltaHandler
        {
            OnEvent = delta =>
            {
                outputText.text += delta.Text;
            }
        };
    }
    
    public async void SendMessage(string text)
    {
        outputText.text = "";
        
        Response response = await agentBehaviour.Agent.GenerateResponseAsync(text);
        
        Debug.Log("✓ Streaming completed");
    }
}
```

### Tool Call Handling

```csharp
public class ToolCallExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    
    async void Start()
    {
        var agent = agentBehaviour.Agent;
        
        await agent.InitializeAsync();
        
        // Executor auto-registered if attached to GameObject
        // Or manually register:
        var executor = GetComponent<WeatherExecutor>();
        agentBehaviour.RegisterToolCallExecutor<FunctionCall, FunctionOutput>(executor);
        
        // Commit tools
        await agent.CommitParametersUpdateAsync();
        
        // Send request that triggers tool
        Response response = await agent.GenerateResponseAsync(
            "What's the weather in Tokyo?"
        );
        
        Debug.Log($"Response: {response.TextContent}");
    }
}

public class WeatherExecutor : MonoBehaviour, IFunctionCallExecutor
{
    public bool CanExecute(string toolName) 
        => toolName == "get_weather";
    
    public async UniTask<FunctionOutput> ExecuteAsync(
        FunctionCall toolCall, 
        CancellationToken ct = default)
    {
        var args = JsonUtility.FromJson<WeatherArgs>(toolCall.Arguments);
        
        // Fetch weather data
        var weather = await FetchWeatherAsync(args.location);
        
        return new FunctionOutput
        {
            CallId = toolCall.CallId,
            Output = JsonUtility.ToJson(weather)
        };
    }
    
    async UniTask<WeatherData> FetchWeatherAsync(string location)
    {
        // Implementation
        return new WeatherData { Temperature = 72, Condition = "Sunny" };
    }
}

[System.Serializable]
public class WeatherArgs
{
    public string location;
}

[System.Serializable]
public class WeatherData
{
    public int Temperature;
    public string Condition;
}
```

### Error Handling

```csharp
public class ErrorHandlingExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    
    void Start()
    {
        var agent = agentBehaviour.Agent;
        
        // Subscribe to errors
        agent.hooks.Error = new ErrorHandler
        {
            OnError = error =>
            {
                Debug.LogError($"Agent error: {error.Message}");
                ShowErrorDialog(error.Message);
            }
        };
    }
    
    public async void SendMessage(string text)
    {
        try
        {
            Response response = await agentBehaviour.Agent.GenerateResponseAsync(text);
            DisplayResponse(response);
        }
        catch (AIApiException ex)
        {
            Debug.LogError($"API error: {ex.Message}");
            ShowRetryButton();
        }
        catch (Exception ex)
        {
            Debug.LogError($"Unexpected error: {ex.Message}");
        }
    }
}
```

### Request Cancellation

```csharp
public class CancellationExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    [SerializeField] private Button cancelButton;
    
    private CancellationTokenSource cts;
    
    void Start()
    {
        cancelButton.onClick.AddListener(CancelRequest);
    }
    
    public async void SendLongRequest()
    {
        cts = new CancellationTokenSource();
        cancelButton.interactable = true;
        
        try
        {
            Response response = await agentBehaviour.Agent.GenerateResponseAsync(
                "Write a long story...",
                ct: cts.Token
            );
            
            Debug.Log($"Response: {response.TextContent}");
        }
        catch (OperationCanceledException)
        {
            Debug.Log("Request cancelled by user");
        }
        finally
        {
            cancelButton.interactable = false;
            cts?.Dispose();
        }
    }
    
    void CancelRequest()
    {
        agentBehaviour.Agent.CancelRequest();
        cts?.Cancel();
    }
}
```

### Parameter Updates

```csharp
public class ParameterUpdateExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agentBehaviour;
    
    public async void UpdateModelAndTools()
    {
        var agent = agentBehaviour.Agent;
        
        // Change model
        agent.Model = Model.GPT4Turbo;
        
        // Change temperature
        // (via settings, as Temperature is read-only)
        agent.Settings.Temperature = 0.8f;
        
        // Register new tools
        var newExecutor = GetComponent<CalculatorExecutor>();
        agentBehaviour.RegisterToolCallExecutor<FunctionCall, FunctionOutput>(newExecutor);
        
        // Commit all changes
        await agent.CommitParametersUpdateAsync();
        
        Debug.Log("✓ Parameters updated");
    }
}
```

## Next Steps

- [Agent Properties](agent-properties.md)
- [AgentBehaviour Properties](agentbehaviour-properties.md)
- [AgentBehaviour Methods](agentbehaviour-methods.md)
