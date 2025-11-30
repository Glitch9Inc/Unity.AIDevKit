# Tools Overview

Tools extend agent capabilities by allowing them to interact with external systems, APIs, and functions.

## What are Tools?

Tools enable agents to:

- **Search files** - Find and read documents
- **Execute code** - Run Python code in sandboxed environment
- **Search web** - Query search engines
- **Call functions** - Invoke custom C# methods
- **Use MCP** - Connect to Model Context Protocol servers
- **Generate images** - Create images with DALL-E
- **Access APIs** - Call external services

## Tool Types

### 1. Hosted Tools (API-based)

Managed by AI providers (OpenAI, Anthropic, Google):

```csharp
// File Search (OpenAI/Google)
agent.AddTool(ToolType.FileSearch);

// Code Interpreter (OpenAI)
agent.AddTool(ToolType.CodeInterpreter);

// Web Search (Google)
agent.AddTool(ToolType.GoogleSearch);
```

### 2. Local Tools (Unity-side)

Run locally in your Unity application:

```csharp
// Function calling
agent.AddTool("GetPlayerStats", GetPlayerStats);

// Local image generation
agent.AddTool(ToolType.ImageGeneration);

// Speech generation
agent.AddTool(ToolType.SpeechGeneration);
```

### 3. MCP Tools

Model Context Protocol servers:

```csharp
// Connect to MCP server
var mcp = agent.MCPController;
await mcp.ConnectAsync("filesystem-server");

// Tools auto-registered
```

## Basic Usage

### Add Single Tool

```csharp
// Add by type
agent.AddTool(ToolType.FileSearch);

// Add function with lambda
agent.AddTool("calculate", (int a, int b) => a + b);

// Add function with method
agent.AddTool("getWeather", GetWeather);
```

### Add Multiple Tools

```csharp
// Add multiple at once
agent.AddTools(
    ToolType.FileSearch,
    ToolType.CodeInterpreter,
    ToolType.WebSearch
);
```

### Remove Tools

```csharp
// Remove specific tool
agent.RemoveTool("calculate");

// Remove all tools
agent.ClearTools();
```

## Tool Choice

Control when and which tools are used:

```csharp
// Let agent decide (default)
agent.ToolChoice = ToolChoice.Auto;

// Must use a tool
agent.ToolChoice = ToolChoice.Required;

// Never use tools
agent.ToolChoice = ToolChoice.None;

// Force specific tool
agent.ToolChoice = new ToolChoice("web_search");
```

## Tool Execution Flow

```
User Message
     ↓
Agent Processes
     ↓
Decides Tool Needed ──→ ToolChoice.None ──→ Direct Response
     ↓
Tool Call Created
     ↓
Execute Tool ──→ Local ──→ Unity Method
     │           Remote ──→ API Call
     │           MCP ──→ MCP Server
     ↓
Tool Output
     ↓
Agent Processes Output
     ↓
Final Response
```

## Tool Events

### Listen for Tool Calls

```csharp
void Start()
{
    agent.onToolCallStarted.AddListener(OnToolCallStarted);
    agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
}

void OnToolCallStarted(ToolCall toolCall)
{
    Debug.Log($"Tool: {toolCall.Function.Name}");
    Debug.Log($"Arguments: {toolCall.Function.Arguments}");
}

void OnToolCallCompleted(ToolCall toolCall, string output)
{
    Debug.Log($"Output: {output}");
}
```

### Handle Tool Errors

```csharp
agent.onToolCallError.AddListener((toolCall, error) =>
{
    Debug.LogError($"Tool '{toolCall.Function.Name}' failed: {error}");
    
    // Show user feedback
    ShowErrorMessage($"Failed to execute {toolCall.Function.Name}");
});
```

## Tool Executors

### Built-in Executors

```csharp
// File operations
var fileExecutor = new FileSearchExecutor();
agent.RegisterExecutor(fileExecutor);

// Web operations
var webExecutor = new WebSearchExecutor();
agent.RegisterExecutor(webExecutor);
```

### Custom Executor

```csharp
public class GameExecutor : IToolExecutor
{
    public string[] SupportedTools => new[] { "get_score", "save_game" };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "get_score" => GetScore().ToString(),
            "save_game" => SaveGame() ? "Saved" : "Failed",
            _ => "Unknown tool"
        };
    }
    
    int GetScore() => PlayerPrefs.GetInt("Score", 0);
    
    bool SaveGame()
    {
        // Save logic
        return true;
    }
}

// Register
agent.RegisterExecutor(new GameExecutor());
```

## Tool Schema

### Define Tool Signature

```csharp
var weatherTool = new Tool
{
    Type = "function",
    Function = new FunctionTool
    {
        Name = "get_weather",
        Description = "Get current weather for a location",
        Parameters = new JsonObject
        {
            ["type"] = "object",
            ["properties"] = new JsonObject
            {
                ["location"] = new JsonObject
                {
                    ["type"] = "string",
                    ["description"] = "City name, e.g. 'San Francisco'"
                },
                ["unit"] = new JsonObject
                {
                    ["type"] = "string",
                    ["enum"] = new JsonArray { "celsius", "fahrenheit" }
                }
            },
            ["required"] = new JsonArray { "location" }
        }
    }
};

agent.AddTool(weatherTool, GetWeather);
```

## Parallel Tool Calls

Some models support calling multiple tools at once:

```csharp
agent.ParallelToolCalls = true;

await agent.SendAsync("Get weather for Tokyo, London, and New York");

// Agent may call get_weather 3 times in parallel
```

## Tool Timeout

```csharp
// Set timeout for tool execution
agent.ToolTimeout = TimeSpan.FromSeconds(30);

// Tools taking longer will be canceled
```

## Common Patterns

### 1. Game Commands

```csharp
agent.AddTool("spawn_enemy", (string enemyType, int count) =>
{
    for (int i = 0; i < count; i++)
    {
        SpawnEnemy(enemyType);
    }
    return $"Spawned {count} {enemyType}";
});

agent.AddTool("set_difficulty", (string difficulty) =>
{
    GameManager.Instance.SetDifficulty(difficulty);
    return $"Difficulty set to {difficulty}";
});
```

### 2. Data Retrieval

```csharp
agent.AddTool("get_player_stats", () =>
{
    return JsonUtility.ToJson(new
    {
        health = player.Health,
        level = player.Level,
        experience = player.Experience
    });
});

agent.AddTool("get_inventory", () =>
{
    var items = inventory.GetAllItems();
    return JsonUtility.ToJson(items);
});
```

### 3. API Integration

```csharp
agent.AddTool("fetch_user_data", async (string userId) =>
{
    using var request = UnityWebRequest.Get($"https://api.example.com/users/{userId}");
    await request.SendWebRequest();
    return request.downloadHandler.text;
});
```

## Tool Discovery

```csharp
// List all registered tools
var tools = agent.GetRegisteredTools();
foreach (var tool in tools)
{
    Debug.Log($"Tool: {tool.Function.Name}");
    Debug.Log($"  Description: {tool.Function.Description}");
}
```

## Best Practices

### 1. Clear Descriptions

```csharp
// ❌ Bad
agent.AddTool("do_thing", () => "done");

// ✅ Good
agent.AddTool(
    name: "save_game",
    description: "Save the current game progress to disk",
    function: () => SaveGame()
);
```

### 2. Input Validation

```csharp
agent.AddTool("set_volume", (float volume) =>
{
    if (volume < 0f || volume > 1f)
    {
        return "Error: Volume must be between 0 and 1";
    }
    
    AudioListener.volume = volume;
    return $"Volume set to {volume}";
});
```

### 3. Error Handling

```csharp
agent.AddTool("complex_operation", async () =>
{
    try
    {
        var result = await PerformOperation();
        return $"Success: {result}";
    }
    catch (Exception ex)
    {
        Debug.LogException(ex);
        return $"Error: {ex.Message}";
    }
});
```

### 4. Return Meaningful Results

```csharp
// ❌ Bad
agent.AddTool("search", (string query) => "ok");

// ✅ Good
agent.AddTool("search", (string query) =>
{
    var results = Search(query);
    return $"Found {results.Count} results: {string.Join(", ", results)}";
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ToolsExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        SetupTools();
        ListenToEvents();
    }
    
    void SetupTools()
    {
        // Hosted tools
        agent.AddTools(
            ToolType.FileSearch,
            ToolType.WebSearch
        );
        
        // Game functions
        agent.AddTool("get_score", GetScore);
        agent.AddTool("spawn_enemy", SpawnEnemy);
        agent.AddTool("save_game", SaveGame);
        
        // Control tool usage
        agent.ToolChoice = ToolChoice.Auto;
        agent.ParallelToolCalls = true;
        agent.ToolTimeout = TimeSpan.FromSeconds(30);
        
        Debug.Log($"✓ Registered {agent.GetRegisteredTools().Count} tools");
    }
    
    void ListenToEvents()
    {
        agent.onToolCallStarted.AddListener(toolCall =>
        {
            Debug.Log($"🔧 Calling tool: {toolCall.Function.Name}");
        });
        
        agent.onToolCallCompleted.AddListener((toolCall, output) =>
        {
            Debug.Log($"✓ Tool completed: {output}");
        });
        
        agent.onToolCallError.AddListener((toolCall, error) =>
        {
            Debug.LogError($"❌ Tool failed: {error}");
        });
    }
    
    // Tool implementations
    string GetScore()
    {
        int score = PlayerPrefs.GetInt("Score", 0);
        return $"Current score: {score}";
    }
    
    string SpawnEnemy(string enemyType, int count)
    {
        for (int i = 0; i < count; i++)
        {
            // Spawn logic
            Debug.Log($"Spawning {enemyType}");
        }
        return $"Spawned {count} {enemyType}";
    }
    
    string SaveGame()
    {
        try
        {
            // Save logic
            PlayerPrefs.Save();
            return "Game saved successfully";
        }
        catch (Exception ex)
        {
            return $"Failed to save: {ex.Message}";
        }
    }
}
```

## Next Steps

- [Registering Executors](registering-executors.md)
- [Function Calls](tool-calls/function.md)
- [Hosted Tools](hosted-tools/)
- [Local Tools](local-tools/)
- [Google Tools](google-tools/)
