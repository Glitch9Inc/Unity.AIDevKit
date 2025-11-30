# Submit Tool Output

Submit results from tool execution back to the agent.

## Basic Usage

### Submit Single Output

```csharp
// After executing a tool
string output = ExecuteTool(toolCall);

// Submit result
await agent.SubmitToolOutputAsync(toolCall.Id, output);
```

### Complete Flow

```csharp
void Start()
{
    agent.onToolCallStarted.AddListener(OnToolCallStarted);
}

async void OnToolCallStarted(ToolCall toolCall)
{
    // Execute tool
    string output = await ExecuteMyTool(toolCall);
    
    // Submit output
    await agent.SubmitToolOutputAsync(toolCall.Id, output);
}

async UniTask<string> ExecuteMyTool(ToolCall toolCall)
{
    // Your tool logic
    await UniTask.Delay(100);
    return "Tool result";
}
```

## Multiple Tool Outputs

### Batch Submission

```csharp
var outputs = new List<ToolOutput>();

foreach (var toolCall in toolCalls)
{
    string result = await ExecuteTool(toolCall);
    outputs.Add(new ToolOutput
    {
        ToolCallId = toolCall.Id,
        Output = result
    });
}

// Submit all at once
await agent.SubmitToolOutputsAsync(outputs);
```

### Parallel Execution

```csharp
async UniTask SubmitParallelOutputs(List<ToolCall> toolCalls)
{
    var tasks = toolCalls.Select(async toolCall =>
    {
        string output = await ExecuteTool(toolCall);
        return new ToolOutput
        {
            ToolCallId = toolCall.Id,
            Output = output
        };
    });
    
    var outputs = await UniTask.WhenAll(tasks);
    await agent.SubmitToolOutputsAsync(outputs.ToList());
}
```

## Structured Output

### JSON Output

```csharp
async UniTask<string> GetPlayerStats(ToolCall toolCall)
{
    var stats = new
    {
        health = player.Health,
        level = player.Level,
        position = new
        {
            x = player.transform.position.x,
            y = player.transform.position.y,
            z = player.transform.position.z
        },
        inventory = player.Inventory.GetItems()
    };
    
    return JsonUtility.ToJson(stats);
}
```

### Formatted Text

```csharp
string FormatWeatherOutput(WeatherData data)
{
    return $@"Weather in {data.Location}:
Temperature: {data.Temperature}°C
Conditions: {data.Conditions}
Humidity: {data.Humidity}%
Wind: {data.WindSpeed} km/h";
}
```

## Error Handling

### Submit Errors

```csharp
async void OnToolCall(ToolCall toolCall)
{
    try
    {
        string output = await ExecuteTool(toolCall);
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    catch (Exception ex)
    {
        Debug.LogException(ex);
        
        // Submit error as output
        await agent.SubmitToolOutputAsync(
            toolCall.Id,
            $"Error: {ex.Message}"
        );
    }
}
```

### Structured Errors

```csharp
string CreateErrorOutput(Exception ex)
{
    var error = new
    {
        success = false,
        error = new
        {
            message = ex.Message,
            type = ex.GetType().Name
        }
    };
    
    return JsonUtility.ToJson(error);
}

// Usage
catch (Exception ex)
{
    string errorOutput = CreateErrorOutput(ex);
    await agent.SubmitToolOutputAsync(toolCall.Id, errorOutput);
}
```

## Timeouts

### Handle Execution Timeout

```csharp
async UniTask SubmitWithTimeout(ToolCall toolCall)
{
    var cts = new CancellationTokenSource();
    cts.CancelAfterSlim(TimeSpan.FromSeconds(30));
    
    try
    {
        string output = await ExecuteTool(toolCall, cts.Token);
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    catch (OperationCanceledException)
    {
        await agent.SubmitToolOutputAsync(
            toolCall.Id,
            "Error: Tool execution timed out"
        );
    }
}
```

## Streaming Tool Output

### Progressive Updates

```csharp
async UniTask StreamToolOutput(ToolCall toolCall)
{
    var outputBuilder = new StringBuilder();
    
    await foreach (var chunk in ExecuteStreamingTool(toolCall))
    {
        outputBuilder.Append(chunk);
        
        // Optional: Send intermediate updates
        Debug.Log($"Progress: {chunk}");
    }
    
    // Submit final output
    await agent.SubmitToolOutputAsync(
        toolCall.Id,
        outputBuilder.ToString()
    );
}

async IAsyncEnumerable<string> ExecuteStreamingTool(ToolCall toolCall)
{
    for (int i = 0; i < 10; i++)
    {
        await UniTask.Delay(100);
        yield return $"Chunk {i}\n";
    }
}
```

## Large Outputs

### Handle Large Data

```csharp
async UniTask<string> HandleLargeOutput(ToolCall toolCall)
{
    string fullOutput = await GetLargeData(toolCall);
    
    // Truncate if too large
    const int maxLength = 10000;
    if (fullOutput.Length > maxLength)
    {
        fullOutput = fullOutput.Substring(0, maxLength) + 
                    "\n... (output truncated)";
    }
    
    return fullOutput;
}
```

### File-Based Output

```csharp
async UniTask<string> SaveToFile(ToolCall toolCall)
{
    string data = await GetLargeData(toolCall);
    
    // Save to file
    string filePath = Path.Combine(
        Application.persistentDataPath,
        $"tool_output_{DateTime.Now:yyyyMMddHHmmss}.txt"
    );
    
    await File.WriteAllTextAsync(filePath, data);
    
    // Return file reference
    return $"Output saved to: {filePath}\nSize: {data.Length} bytes";
}
```

## Validation

### Validate Before Submit

```csharp
async UniTask SubmitValidated(ToolCall toolCall)
{
    string output = await ExecuteTool(toolCall);
    
    if (ValidateOutput(output))
    {
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    else
    {
        await agent.SubmitToolOutputAsync(
            toolCall.Id,
            "Error: Invalid tool output"
        );
    }
}

bool ValidateOutput(string output)
{
    // Check output is valid
    if (string.IsNullOrEmpty(output))
        return false;
    
    // Check JSON if expected
    try
    {
        JsonUtility.FromJson<object>(output);
        return true;
    }
    catch
    {
        return false;
    }
}
```

## Retry Logic

### Retry on Failure

```csharp
async UniTask SubmitWithRetry(ToolCall toolCall, int maxRetries = 3)
{
    for (int attempt = 0; attempt < maxRetries; attempt++)
    {
        try
        {
            string output = await ExecuteTool(toolCall);
            await agent.SubmitToolOutputAsync(toolCall.Id, output);
            return; // Success
        }
        catch (Exception ex) when (attempt < maxRetries - 1)
        {
            Debug.LogWarning($"Attempt {attempt + 1} failed: {ex.Message}");
            await UniTask.Delay(1000 * (attempt + 1)); // Exponential backoff
        }
    }
    
    // All retries failed
    await agent.SubmitToolOutputAsync(
        toolCall.Id,
        "Error: Tool execution failed after retries"
    );
}
```

## Caching

### Cache Tool Results

```csharp
public class CachedToolExecutor : MonoBehaviour
{
    private Dictionary<string, string> cache = new();
    
    async UniTask<string> ExecuteWithCache(ToolCall toolCall)
    {
        string cacheKey = GetCacheKey(toolCall);
        
        // Check cache
        if (cache.TryGetValue(cacheKey, out string cachedOutput))
        {
            Debug.Log("📦 Using cached result");
            return cachedOutput;
        }
        
        // Execute
        string output = await ExecuteTool(toolCall);
        
        // Cache result
        cache[cacheKey] = output;
        
        return output;
    }
    
    string GetCacheKey(ToolCall toolCall)
    {
        return $"{toolCall.Function.Name}:{toolCall.Function.Arguments}";
    }
}
```

## Logging

### Log All Submissions

```csharp
public class ToolOutputLogger : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async UniTask SubmitWithLogging(ToolCall toolCall, string output)
    {
        // Log before submit
        Debug.Log($"📤 Submitting output for: {toolCall.Function.Name}");
        Debug.Log($"   Output length: {output.Length} chars");
        Debug.Log($"   Output preview: {GetPreview(output, 100)}");
        
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
        
        stopwatch.Stop();
        Debug.Log($"✓ Submitted in {stopwatch.ElapsedMilliseconds}ms");
    }
    
    string GetPreview(string text, int maxLength)
    {
        if (text.Length <= maxLength)
            return text;
        
        return text.Substring(0, maxLength) + "...";
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using System.Text;

public class ToolOutputManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int timeoutSeconds = 30;
    [SerializeField] private int maxRetries = 3;
    
    private Dictionary<string, string> cache = new();
    
    void Start()
    {
        agent.onToolCallStarted.AddListener(OnToolCallStarted);
    }
    
    async void OnToolCallStarted(ToolCall toolCall)
    {
        Debug.Log($"🔧 Tool called: {toolCall.Function.Name}");
        
        // Check cache
        string cacheKey = GetCacheKey(toolCall);
        if (cache.TryGetValue(cacheKey, out string cachedOutput))
        {
            Debug.Log("📦 Using cached result");
            await SubmitOutput(toolCall.Id, cachedOutput);
            return;
        }
        
        // Execute with timeout and retry
        await ExecuteAndSubmit(toolCall);
    }
    
    async UniTask ExecuteAndSubmit(ToolCall toolCall)
    {
        for (int attempt = 0; attempt < maxRetries; attempt++)
        {
            var cts = new CancellationTokenSource();
            cts.CancelAfterSlim(TimeSpan.FromSeconds(timeoutSeconds));
            
            try
            {
                // Execute with cancellation
                string output = await ExecuteTool(toolCall, cts.Token);
                
                // Validate
                if (!ValidateOutput(output))
                {
                    throw new Exception("Invalid output format");
                }
                
                // Cache
                string cacheKey = GetCacheKey(toolCall);
                cache[cacheKey] = output;
                
                // Submit
                await SubmitOutput(toolCall.Id, output);
                return; // Success
            }
            catch (OperationCanceledException)
            {
                Debug.LogWarning($"Attempt {attempt + 1}: Timeout");
                if (attempt == maxRetries - 1)
                {
                    await SubmitError(toolCall.Id, "Tool execution timed out");
                }
            }
            catch (Exception ex)
            {
                Debug.LogWarning($"Attempt {attempt + 1}: {ex.Message}");
                if (attempt == maxRetries - 1)
                {
                    await SubmitError(toolCall.Id, ex.Message);
                }
                else
                {
                    await UniTask.Delay(1000 * (attempt + 1)); // Backoff
                }
            }
        }
    }
    
    async UniTask<string> ExecuteTool(ToolCall toolCall, CancellationToken ct)
    {
        return toolCall.Function.Name switch
        {
            "get_weather" => await GetWeather(toolCall, ct),
            "search_files" => await SearchFiles(toolCall, ct),
            "analyze_data" => await AnalyzeData(toolCall, ct),
            _ => "Tool not implemented"
        };
    }
    
    async UniTask<string> GetWeather(ToolCall toolCall, CancellationToken ct)
    {
        await UniTask.Delay(500, cancellationToken: ct);
        
        return JsonUtility.ToJson(new
        {
            location = "Tokyo",
            temperature = 22,
            conditions = "Sunny"
        });
    }
    
    async UniTask<string> SearchFiles(ToolCall toolCall, CancellationToken ct)
    {
        await UniTask.Delay(1000, cancellationToken: ct);
        return "Found 5 files matching query";
    }
    
    async UniTask<string> AnalyzeData(ToolCall toolCall, CancellationToken ct)
    {
        var output = new StringBuilder();
        
        for (int i = 0; i < 10; i++)
        {
            await UniTask.Delay(200, cancellationToken: ct);
            output.AppendLine($"Analyzing chunk {i + 1}/10...");
        }
        
        output.AppendLine("Analysis complete");
        return output.ToString();
    }
    
    async UniTask SubmitOutput(string toolCallId, string output)
    {
        Debug.Log($"📤 Submitting output ({output.Length} chars)");
        
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        await agent.SubmitToolOutputAsync(toolCallId, output);
        stopwatch.Stop();
        
        Debug.Log($"✓ Submitted in {stopwatch.ElapsedMilliseconds}ms");
    }
    
    async UniTask SubmitError(string toolCallId, string errorMessage)
    {
        Debug.LogError($"❌ Submitting error: {errorMessage}");
        
        var error = new
        {
            success = false,
            error = errorMessage
        };
        
        await agent.SubmitToolOutputAsync(
            toolCallId,
            JsonUtility.ToJson(error)
        );
    }
    
    bool ValidateOutput(string output)
    {
        if (string.IsNullOrEmpty(output))
            return false;
        
        // Add more validation as needed
        return true;
    }
    
    string GetCacheKey(ToolCall toolCall)
    {
        return $"{toolCall.Function.Name}:{toolCall.Function.Arguments}";
    }
}
```

## Best Practices

### 1. Always Submit

```csharp
// ❌ Bad - no output submitted
async void OnToolCall(ToolCall toolCall)
{
    ExecuteTool(toolCall); // Fire and forget
}

// ✅ Good - always submit
async void OnToolCall(ToolCall toolCall)
{
    string output = await ExecuteTool(toolCall);
    await agent.SubmitToolOutputAsync(toolCall.Id, output);
}
```

### 2. Handle Errors

```csharp
// Always submit even on error
try
{
    output = await ExecuteTool(toolCall);
}
catch (Exception ex)
{
    output = $"Error: {ex.Message}";
}
finally
{
    await agent.SubmitToolOutputAsync(toolCall.Id, output);
}
```

### 3. Provide Context

```csharp
// ❌ Bad
return "Done";

// ✅ Good
return "Successfully processed 42 items in 1.5 seconds";
```

## Next Steps

- [Unhandled Tool Calls](unhandled-tool-calls.md)
- [Function Calls](tool-calls/function.md)
- [Tool Events](../events-hooks/tool-calls.md)
