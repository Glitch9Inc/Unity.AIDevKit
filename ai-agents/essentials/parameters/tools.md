# Tools & Tool Choice

Configure tools and tool execution strategies.

## Overview

Tool Choice controls how the agent decides to use tools:

- **Auto** - Agent decides when to use tools
- **Required** - Agent must use at least one tool
- **None** - Agent cannot use tools
- **Specific** - Force a specific tool

## Tool Choice Modes

### Auto (Recommended)

```csharp
agent.ToolChoice = ToolChoice.Auto;

// Agent decides based on context
await agent.SendAsync("What's the weather in Tokyo?");
// → Calls get_weather tool

await agent.SendAsync("Hello!");
// → Responds directly without tools
```

### Required

```csharp
agent.ToolChoice = ToolChoice.Required;

// Agent MUST use a tool
await agent.SendAsync("Help me");
// → Will use one of the available tools, even if not needed
```

### None

```csharp
agent.ToolChoice = ToolChoice.None;

// Agent cannot use tools
await agent.SendAsync("What's the weather?");
// → Responds with text, no tool calls
```

### Specific Tool

```csharp
// Force specific tool
agent.ToolChoice = new ToolChoice("get_weather");

await agent.SendAsync("Hello");
// → Will try to call get_weather even though it's not relevant
```

## Registering Tools

### Single Tool

```csharp
// Register tool executor
agent.RegisterToolExecutor(new WeatherToolExecutor());

// Now agent can call get_weather
await agent.SendAsync("What's the weather in Tokyo?");
```

### Multiple Tools

```csharp
// Register multiple tools
agent.RegisterToolExecutors(new IToolExecutor[]
{
    new WeatherToolExecutor(),
    new CalculatorToolExecutor(),
    new SearchToolExecutor()
});
```

### Tool Definitions in Settings

```csharp
// In AgentSettings, add tool definitions
settings.ToolDefinitions.Add(new FunctionToolDefinition
{
    Name = "get_weather",
    Description = "Get current weather for a location",
    Parameters = new
    {
        type = "object",
        properties = new
        {
            location = new
            {
                type = "string",
                description = "City name, e.g. Tokyo"
            },
            unit = new
            {
                type = "string",
                @enum = new[] { "celsius", "fahrenheit" }
            }
        },
        required = new[] { "location" }
    }
});
```

## Tool Choice Strategies

### Context-Aware Tool Choice

```csharp
public class SmartToolChoice : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void SendSmart(string message)
    {
        // Analyze message
        if (NeedsTools(message))
        {
            agent.ToolChoice = ToolChoice.Auto;
        }
        else
        {
            agent.ToolChoice = ToolChoice.None;
        }
        
        await agent.SendAsync(message);
    }
    
    bool NeedsTools(string message)
    {
        string lower = message.ToLower();
        
        // Check for tool keywords
        return lower.Contains("weather") ||
               lower.Contains("calculate") ||
               lower.Contains("search") ||
               lower.Contains("find");
    }
}
```

### Dynamic Tool Choice

```csharp
public ToolChoice GetToolChoice(string intent)
{
    return intent switch
    {
        "weather" => new ToolChoice("get_weather"),
        "calculation" => new ToolChoice("calculator"),
        "search" => new ToolChoice("web_search"),
        "chat" => ToolChoice.None,
        _ => ToolChoice.Auto
    };
}

// Usage
string intent = ClassifyIntent(message);
agent.ToolChoice = GetToolChoice(intent);
await agent.SendAsync(message);
```

## Built-in Tools

### File Search

```csharp
// Enable File Search
settings.EnableFileSearch = true;
settings.FileSearchFileIds = new[] { "file-abc123" };

// Agent can search uploaded files
agent.ToolChoice = ToolChoice.Auto;
await agent.SendAsync("What does the documentation say about Unity?");
```

### Code Interpreter

```csharp
// Enable Code Interpreter
settings.EnableCodeInterpreter = true;

// Agent can execute Python code
await agent.SendAsync("Calculate the first 10 Fibonacci numbers");
```

### Web Search

```csharp
// Enable Web Search (Google/Bing)
settings.EnableWebSearch = true;

// Agent can search the web
await agent.SendAsync("What are the latest Unity features?");
```

## Custom Tools

### Define Custom Tool

```csharp
public class CustomToolExecutor : IToolExecutor
{
    public string ToolName => "custom_tool";
    
    public async UniTask<string> ExecuteAsync(
        string arguments,
        CancellationToken ct = default)
    {
        // Parse arguments
        var args = JsonConvert.DeserializeObject<CustomArgs>(arguments);
        
        // Execute tool logic
        string result = await DoCustomWorkAsync(args);
        
        // Return result as JSON
        return JsonConvert.SerializeObject(new { result });
    }
}

// Register
agent.RegisterToolExecutor(new CustomToolExecutor());
```

### Tool with Parameters

```csharp
public class ParameterizedTool : IToolExecutor
{
    public string ToolName => "complex_tool";
    
    public class Args
    {
        public string Required { get; set; }
        public string Optional { get; set; }
        public int Number { get; set; }
        public bool Flag { get; set; }
    }
    
    public async UniTask<string> ExecuteAsync(
        string arguments,
        CancellationToken ct = default)
    {
        var args = JsonConvert.DeserializeObject<Args>(arguments);
        
        // Validate required parameters
        if (string.IsNullOrEmpty(args.Required))
        {
            return JsonConvert.SerializeObject(new
            {
                error = "Required parameter missing"
            });
        }
        
        // Execute with parameters
        var result = ProcessWithParams(args);
        
        return JsonConvert.SerializeObject(result);
    }
}
```

## Tool Events

### Handle Tool Calls

```csharp
void Start()
{
    agent.onToolCallStarted.AddListener(OnToolCallStarted);
    agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
    agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
}

void OnToolCallStarted(ToolCall toolCall)
{
    Debug.Log($"Tool started: {toolCall.Function.Name}");
    ShowToolIndicator(toolCall.Function.Name);
}

void OnToolCallCompleted(ToolCall toolCall, string output)
{
    Debug.Log($"Tool completed: {toolCall.Function.Name}");
    HideToolIndicator();
}

void OnUnhandledToolCall(ToolCall toolCall)
{
    Debug.LogWarning($"Unhandled tool: {toolCall.Function.Name}");
}
```

### Tool Call Progress

```csharp
public class ToolProgress : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text statusText;
    
    void Start()
    {
        agent.onToolCallStarted.AddListener(OnToolStarted);
        agent.onToolCallCompleted.AddListener(OnToolCompleted);
    }
    
    void OnToolStarted(ToolCall toolCall)
    {
        statusText.text = $"Using {toolCall.Function.Name}...";
    }
    
    void OnToolCompleted(ToolCall toolCall, string output)
    {
        statusText.text = $"✓ {toolCall.Function.Name} completed";
        
        // Clear after delay
        Invoke(nameof(ClearStatus), 2f);
    }
    
    void ClearStatus()
    {
        statusText.text = "";
    }
}
```

## Tool Timeouts

### Configure Timeout

```csharp
// In AgentBehaviour
agent.SubmitToolOutputTimeoutSeconds = 30;

// If tool doesn't complete in 30 seconds, it will timeout
```

### Handle Timeout

```csharp
agent.onError.AddListener(OnError);

void OnError(string error)
{
    if (error.Contains("timeout"))
    {
        Debug.LogWarning("Tool execution timed out");
        ShowMessage("Tool took too long to respond");
    }
}
```

## Parallel Tool Calls

### Multiple Tools at Once

```csharp
// Some models can call multiple tools in parallel
agent.ToolChoice = ToolChoice.Auto;

await agent.SendAsync("Get weather for Tokyo, London, and New York");

// Agent may call get_weather three times in parallel:
// - get_weather(location="Tokyo")
// - get_weather(location="London")
// - get_weather(location="New York")
```

### Handle Parallel Calls

```csharp
void Start()
{
    agent.onResponseCompleted.AddListener(OnResponseCompleted);
}

void OnResponseCompleted(Response response)
{
    if (response.ToolCalls != null && response.ToolCalls.Count > 1)
    {
        Debug.Log($"Parallel tool calls: {response.ToolCalls.Count}");
        
        foreach (var toolCall in response.ToolCalls)
        {
            Debug.Log($"  - {toolCall.Function.Name}");
        }
    }
}
```

## Unhandled Tools

### Behavior Options

```csharp
// Skip unhandled tools
agent.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.Skip;

// Throw error
agent.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.ThrowError;

// Ask user for approval/input
agent.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.AskUser;
```

### Handle Unregistered Tools

```csharp
agent.onUnhandledToolCall += async (toolCall) =>
{
    Debug.LogWarning($"Unhandled tool: {toolCall.Function.Name}");
    
    // Provide default response
    string output = JsonConvert.SerializeObject(new
    {
        error = $"Tool {toolCall.Function.Name} is not available"
    });
    
    await agent.SubmitToolOutputAsync(toolCall.Id, output);
};
```

## Best Practices

### 1. Start with Auto

```csharp
// Let agent decide
agent.ToolChoice = ToolChoice.Auto;

// Agent is smart about when to use tools
```

### 2. Provide Clear Tool Descriptions

```csharp
new FunctionToolDefinition
{
    Name = "get_weather",
    Description = "Get current weather and forecast for a specific location. " +
                 "Returns temperature, conditions, humidity, and wind speed.",
    Parameters = // ...
}
```

### 3. Validate Tool Results

```csharp
public class ValidatedToolExecutor : IToolExecutor
{
    public async UniTask<string> ExecuteAsync(string arguments, CancellationToken ct)
    {
        try
        {
            var result = await ExecuteToolLogic(arguments);
            
            // Validate result
            if (IsValidResult(result))
            {
                return JsonConvert.SerializeObject(result);
            }
            else
            {
                return JsonConvert.SerializeObject(new
                {
                    error = "Invalid tool result"
                });
            }
        }
        catch (Exception ex)
        {
            return JsonConvert.SerializeObject(new
            {
                error = ex.Message
            });
        }
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using System.Collections.Generic;

public class ToolManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Tool Settings")]
    [SerializeField] private bool allowTools = true;
    [SerializeField] private ToolChoiceMode defaultMode = ToolChoiceMode.Auto;
    
    void Start()
    {
        // Register tools
        RegisterTools();
        
        // Configure tool choice
        ConfigureToolChoice();
        
        // Setup events
        agent.onToolCallStarted.AddListener(OnToolStarted);
        agent.onToolCallCompleted.AddListener(OnToolCompleted);
        agent.onUnhandledToolCall.AddListener(OnUnhandledTool);
    }
    
    void RegisterTools()
    {
        var tools = new List<IToolExecutor>
        {
            new WeatherToolExecutor(),
            new CalculatorToolExecutor(),
            new SearchToolExecutor()
        };
        
        agent.RegisterToolExecutors(tools);
        
        Debug.Log($"✓ Registered {tools.Count} tools");
    }
    
    void ConfigureToolChoice()
    {
        if (!allowTools)
        {
            agent.ToolChoice = ToolChoice.None;
            return;
        }
        
        agent.ToolChoice = defaultMode switch
        {
            ToolChoiceMode.Auto => ToolChoice.Auto,
            ToolChoiceMode.Required => ToolChoice.Required,
            ToolChoiceMode.None => ToolChoice.None,
            _ => ToolChoice.Auto
        };
    }
    
    void OnToolStarted(ToolCall toolCall)
    {
        Debug.Log($"🔧 Tool: {toolCall.Function.Name}");
        Debug.Log($"   Args: {toolCall.Function.Arguments}");
    }
    
    void OnToolCompleted(ToolCall toolCall, string output)
    {
        Debug.Log($"✓ Tool completed: {toolCall.Function.Name}");
        Debug.Log($"   Output: {output.Substring(0, System.Math.Min(100, output.Length))}...");
    }
    
    void OnUnhandledTool(ToolCall toolCall)
    {
        Debug.LogWarning($"⚠ Unhandled tool: {toolCall.Function.Name}");
    }
    
    enum ToolChoiceMode
    {
        Auto,
        Required,
        None
    }
}
```

## Next Steps

- [Registering Tool Executors](../tools/registering-executors.md)
- [Tool Calls](../tools/tool-calls/README.md)
- [Unhandled Tool Calls](../tools/unhandled-tool-calls.md)
- [Hosted Tools](../tools/hosted-tools/README.md)
