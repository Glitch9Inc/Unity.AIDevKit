# Tools

Enable your agent to call functions, execute commands, and integrate with external systems.

## Overview

Tools allow agents to:

- **Call functions** in your code
- **Execute shell commands** locally
- **Use hosted services** (file search, code interpreter, web search)
- **Generate content** locally (images, speech)
- **Access Google services** (search, code execution)
- **Connect via MCP** (Model Context Protocol)

## Tool Categories

### Function Tools

Agent can call C# methods in your code.

```csharp
public class WeatherTool : IToolExecutor
{
    public string Name => "get_weather";
    public string Description => "Get current weather for a location";
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        string location = toolCall.GetArgument<string>("location");
        // Fetch weather data...
        return $"Weather in {location}: Sunny, 72°F";
    }
}

// Register tool
agent.RegisterToolExecutor(new WeatherTool());
```

### Local Shell

Execute command-line tools on the local system.

```csharp
// Enable in AgentSettings
settings.EnableLocalShell = true;

// Agent can now run commands like:
// "Run ls -la to list files"
// "Execute python script.py"
```

### Computer Use (Anthropic)

Let Claude control your computer (screenshots, mouse, keyboard).

```csharp
settings.EnableComputerUse = true;
// Requires Anthropic API
```

### Hosted Tools

Server-side tools provided by AI platforms:

- **File Search** - Search through uploaded documents
- **Code Interpreter** - Execute Python code
- **Web Search** - Search the internet
- **MCP Tools** - Model Context Protocol integrations

```csharp
settings.EnableFileSearch = true;
settings.EnableCodeInterpreter = true;
```

### Local Tools

Generate content locally using AI APIs:

- **Image Generation** - Create images with DALL-E, Stable Diffusion
- **Speech Generation** - Convert text to speech
- **Speech Transcription** - Convert speech to text

```csharp
var imageTool = new ImageGenerationToolDefinition
{
    Model = "dall-e-3",
    Size = "1024x1024"
};
settings.ToolDefinitions.Add(imageTool);
```

### Google Tools

Google-specific integrations:

- **Google Search** - Search via Google
- **URL Context** - Fetch and analyze web pages
- **Code Execution** - Run code in Google's sandbox

```csharp
settings.EnableGoogleSearch = true;
settings.EnableCodeExecution = true;
```

## Tool Registration

### Register Function Tool

```csharp
// Define tool
public class CalculatorTool : IToolExecutor
{
    public string Name => "calculate";
    public string Description => "Perform mathematical calculations";
    
    [ToolParameter("expression", "The math expression to evaluate")]
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        string expression = toolCall.GetArgument<string>("expression");
        // Evaluate expression...
        return result.ToString();
    }
}

// Register with agent
agent.RegisterToolExecutor(new CalculatorTool());
```

### Register Multiple Tools

```csharp
agent.RegisterToolExecutors(new IToolExecutor[]
{
    new WeatherTool(),
    new CalculatorTool(),
    new DatabaseTool()
});
```

### Dynamic Tool Registration

```csharp
// Register at runtime based on context
if (userHasPermission)
{
    agent.RegisterToolExecutor(new AdminTool());
}
```

## Tool Call Handling

### Automatic Handling

```csharp
// Agent automatically executes registered tools
await agent.SendAsync("What's the weather in London?");
// Agent calls WeatherTool, includes result in response
```

### Manual Handling

```csharp
agent.onUnhandledToolCall += (toolCall) =>
{
    Debug.Log($"Unhandled tool: {toolCall.Function.Name}");
    
    // Execute manually
    string result = ExecuteCustomTool(toolCall);
    
    // Submit result
    agent.SubmitToolOutput(toolCall.Id, result);
};
```

### Tool Choice Strategy

```csharp
// Let agent decide
agent.ToolChoice = ToolChoice.Auto;

// Force tool use
agent.ToolChoice = ToolChoice.Required;

// Disable tools
agent.ToolChoice = ToolChoice.None;

// Force specific tool
agent.ToolChoice = ToolChoice.Function("get_weather");
```

## Unhandled Tool Call Behavior

Configure how agent handles tools not registered locally:

```csharp
// Ignore unhandled tools
behaviour.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.LogAndIgnore;

// Submit tool output manually
behaviour.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.SubmitToolOutput;
behaviour.SubmitToolOutputTimeoutSeconds = 60;

// Return refusal message
behaviour.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.ReturnRefusal;

// Throw exception
behaviour.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.Throw;
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ToolExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Register tools
        agent.RegisterToolExecutor(new WeatherTool());
        agent.RegisterToolExecutor(new TimeTool());
        
        // Configure tool behavior
        agent.ToolChoice = ToolChoice.Auto;
        agent.UnhandledToolCallBehaviour = UnhandledToolCallBehaviour.RaiseEvent;
        
        // Handle tool calls
        agent.onToolCallStarted.AddListener(OnToolCallStarted);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onUnhandledToolCall += OnUnhandledToolCall;
    }
    
    void OnToolCallStarted(ToolCall toolCall)
    {
        Debug.Log($"Calling tool: {toolCall.Function.Name}");
    }
    
    void OnToolCallCompleted(ToolCall toolCall, string result)
    {
        Debug.Log($"Tool result: {result}");
    }
    
    void OnUnhandledToolCall(ToolCall toolCall)
    {
        Debug.LogWarning($"Unhandled: {toolCall.Function.Name}");
    }
}

// Weather tool implementation
public class WeatherTool : IToolExecutor
{
    public string Name => "get_weather";
    public string Description => "Get current weather for a location";
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        string location = toolCall.GetArgument<string>("location");
        
        // Call weather API
        var weatherData = await FetchWeatherAsync(location);
        
        return $"Temperature: {weatherData.temp}°F, Condition: {weatherData.condition}";
    }
    
    async UniTask<WeatherData> FetchWeatherAsync(string location)
    {
        // Implementation...
        return new WeatherData { temp = 72, condition = "Sunny" };
    }
}
```

## Next Steps

- [Tool Overview](overview.md) - Detailed tool concepts
- [Registering Tool Executors](registering-executors.md) - Tool registration guide
- [Tool Calls](tool-calls/README.md) - Function, Shell, Computer Use
- [Unhandled Tool Calls](unhandled-tool-calls.md) - Handle missing tools
- [Submit Tool Output](submit-tool-output.md) - Manual tool execution
