# Registering Tool Executors

Tool executors handle the execution of tools called by agents.

## IToolExecutor Interface

```csharp
public interface IToolExecutor
{
    string[] SupportedTools { get; }
    UniTask<string> ExecuteAsync(ToolCall toolCall);
}
```

## Basic Executor

### Simple Implementation

```csharp
public class BasicExecutor : IToolExecutor
{
    public string[] SupportedTools => new[] { "get_time", "get_date" };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "get_time" => DateTime.Now.ToString("HH:mm:ss"),
            "get_date" => DateTime.Now.ToString("yyyy-MM-dd"),
            _ => "Unknown tool"
        };
    }
}
```

### Register Executor

```csharp
void Start()
{
    var executor = new BasicExecutor();
    agent.RegisterExecutor(executor);
}
```

## Game Executor

### Player Operations

```csharp
public class PlayerExecutor : IToolExecutor
{
    public string[] SupportedTools => new[]
    {
        "get_health",
        "get_position",
        "get_inventory",
        "heal_player"
    };
    
    private readonly Player player;
    
    public PlayerExecutor(Player player)
    {
        this.player = player;
    }
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "get_health" => GetHealth(),
            "get_position" => GetPosition(),
            "get_inventory" => GetInventory(),
            "heal_player" => HealPlayer(toolCall),
            _ => "Unknown tool"
        };
    }
    
    string GetHealth()
    {
        return JsonUtility.ToJson(new
        {
            current = player.Health,
            max = player.MaxHealth,
            percentage = (player.Health / player.MaxHealth) * 100
        });
    }
    
    string GetPosition()
    {
        var pos = player.transform.position;
        return $"x: {pos.x:F2}, y: {pos.y:F2}, z: {pos.z:F2}";
    }
    
    string GetInventory()
    {
        var items = player.Inventory.GetAllItems();
        return JsonUtility.ToJson(items);
    }
    
    string HealPlayer(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<HealArgs>(toolCall.Function.Arguments);
        player.Heal(args.amount);
        return $"Healed {args.amount} HP. Current health: {player.Health}";
    }
    
    [System.Serializable]
    class HealArgs
    {
        public float amount;
    }
}

// Register with player reference
agent.RegisterExecutor(new PlayerExecutor(player));
```

## Async Operations

### API Calls

```csharp
public class APIExecutor : IToolExecutor
{
    public string[] SupportedTools => new[]
    {
        "fetch_weather",
        "fetch_news",
        "fetch_user_data"
    };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "fetch_weather" => await FetchWeather(toolCall),
            "fetch_news" => await FetchNews(toolCall),
            "fetch_user_data" => await FetchUserData(toolCall),
            _ => "Unknown tool"
        };
    }
    
    async UniTask<string> FetchWeather(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<WeatherArgs>(toolCall.Function.Arguments);
        
        string url = $"https://api.weather.com/v1/current?location={args.location}";
        using var request = UnityWebRequest.Get(url);
        
        await request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            return request.downloadHandler.text;
        }
        else
        {
            return $"Error: {request.error}";
        }
    }
    
    async UniTask<string> FetchNews(ToolCall toolCall)
    {
        // Similar implementation
        await UniTask.Delay(100);
        return "News data...";
    }
    
    async UniTask<string> FetchUserData(ToolCall toolCall)
    {
        // Similar implementation
        await UniTask.Delay(100);
        return "User data...";
    }
    
    [System.Serializable]
    class WeatherArgs
    {
        public string location;
        public string unit;
    }
}
```

## Database Executor

### Data Operations

```csharp
public class DatabaseExecutor : IToolExecutor
{
    public string[] SupportedTools => new[]
    {
        "query_users",
        "create_user",
        "update_user",
        "delete_user"
    };
    
    private readonly IDatabase database;
    
    public DatabaseExecutor(IDatabase database)
    {
        this.database = database;
    }
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        try
        {
            return toolCall.Function.Name switch
            {
                "query_users" => await QueryUsers(toolCall),
                "create_user" => await CreateUser(toolCall),
                "update_user" => await UpdateUser(toolCall),
                "delete_user" => await DeleteUser(toolCall),
                _ => "Unknown tool"
            };
        }
        catch (Exception ex)
        {
            Debug.LogException(ex);
            return $"Database error: {ex.Message}";
        }
    }
    
    async UniTask<string> QueryUsers(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<QueryArgs>(toolCall.Function.Arguments);
        var users = await database.QueryAsync<User>(args.query);
        return JsonUtility.ToJson(new { count = users.Count, users });
    }
    
    async UniTask<string> CreateUser(ToolCall toolCall)
    {
        var user = JsonUtility.FromJson<User>(toolCall.Function.Arguments);
        await database.InsertAsync(user);
        return $"Created user: {user.Name}";
    }
    
    async UniTask<string> UpdateUser(ToolCall toolCall)
    {
        var user = JsonUtility.FromJson<User>(toolCall.Function.Arguments);
        await database.UpdateAsync(user);
        return $"Updated user: {user.Name}";
    }
    
    async UniTask<string> DeleteUser(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<DeleteArgs>(toolCall.Function.Arguments);
        await database.DeleteAsync<User>(args.userId);
        return $"Deleted user: {args.userId}";
    }
    
    [System.Serializable]
    class QueryArgs { public string query; }
    
    [System.Serializable]
    class DeleteArgs { public int userId; }
}
```

## Unity Services Executor

### Scene & GameObject Operations

```csharp
public class UnityServicesExecutor : IToolExecutor
{
    public string[] SupportedTools => new[]
    {
        "load_scene",
        "find_objects",
        "spawn_prefab",
        "destroy_objects"
    };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "load_scene" => await LoadScene(toolCall),
            "find_objects" => FindObjects(toolCall),
            "spawn_prefab" => SpawnPrefab(toolCall),
            "destroy_objects" => DestroyObjects(toolCall),
            _ => "Unknown tool"
        };
    }
    
    async UniTask<string> LoadScene(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<SceneArgs>(toolCall.Function.Arguments);
        
        var operation = SceneManager.LoadSceneAsync(args.sceneName);
        await operation;
        
        return $"Loaded scene: {args.sceneName}";
    }
    
    string FindObjects(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<FindArgs>(toolCall.Function.Arguments);
        var objects = GameObject.FindObjectsOfType<GameObject>()
            .Where(go => go.name.Contains(args.namePattern))
            .ToList();
        
        return $"Found {objects.Count} objects matching '{args.namePattern}'";
    }
    
    string SpawnPrefab(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<SpawnArgs>(toolCall.Function.Arguments);
        
        var prefab = Resources.Load<GameObject>(args.prefabName);
        if (prefab == null)
        {
            return $"Error: Prefab '{args.prefabName}' not found";
        }
        
        var position = new Vector3(args.x, args.y, args.z);
        GameObject.Instantiate(prefab, position, Quaternion.identity);
        
        return $"Spawned {args.prefabName} at {position}";
    }
    
    string DestroyObjects(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<FindArgs>(toolCall.Function.Arguments);
        var objects = GameObject.FindObjectsOfType<GameObject>()
            .Where(go => go.name.Contains(args.namePattern))
            .ToList();
        
        foreach (var obj in objects)
        {
            GameObject.Destroy(obj);
        }
        
        return $"Destroyed {objects.Count} objects";
    }
    
    [System.Serializable]
    class SceneArgs { public string sceneName; }
    
    [System.Serializable]
    class FindArgs { public string namePattern; }
    
    [System.Serializable]
    class SpawnArgs
    {
        public string prefabName;
        public float x, y, z;
    }
}
```

## Composite Executor

### Multiple Executors

```csharp
public class CompositeExecutor : IToolExecutor
{
    private readonly List<IToolExecutor> executors = new();
    
    public string[] SupportedTools
    {
        get
        {
            return executors
                .SelectMany(e => e.SupportedTools)
                .Distinct()
                .ToArray();
        }
    }
    
    public void AddExecutor(IToolExecutor executor)
    {
        executors.Add(executor);
    }
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        var executor = executors.FirstOrDefault(e =>
            e.SupportedTools.Contains(toolCall.Function.Name));
        
        if (executor == null)
        {
            return $"No executor found for tool: {toolCall.Function.Name}";
        }
        
        return await executor.ExecuteAsync(toolCall);
    }
}

// Usage
var composite = new CompositeExecutor();
composite.AddExecutor(new PlayerExecutor(player));
composite.AddExecutor(new APIExecutor());
composite.AddExecutor(new DatabaseExecutor(database));

agent.RegisterExecutor(composite);
```

## Error Handling

### Try-Catch Pattern

```csharp
public class SafeExecutor : IToolExecutor
{
    public string[] SupportedTools => new[] { "risky_operation" };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        try
        {
            return await PerformOperation(toolCall);
        }
        catch (ArgumentException ex)
        {
            Debug.LogWarning($"Invalid arguments: {ex.Message}");
            return $"Error: Invalid arguments - {ex.Message}";
        }
        catch (UnauthorizedAccessException ex)
        {
            Debug.LogError($"Access denied: {ex.Message}");
            return "Error: Permission denied";
        }
        catch (Exception ex)
        {
            Debug.LogException(ex);
            return $"Error: {ex.Message}";
        }
    }
    
    async UniTask<string> PerformOperation(ToolCall toolCall)
    {
        // Operation that might throw
        await UniTask.Delay(100);
        return "Success";
    }
}
```

## Timeout Handling

```csharp
public class TimeoutExecutor : IToolExecutor
{
    public string[] SupportedTools => new[] { "long_operation" };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        var cts = new CancellationTokenSource();
        cts.CancelAfterSlim(TimeSpan.FromSeconds(30));
        
        try
        {
            return await PerformOperation(toolCall, cts.Token);
        }
        catch (OperationCanceledException)
        {
            return "Error: Operation timed out";
        }
    }
    
    async UniTask<string> PerformOperation(ToolCall toolCall, CancellationToken ct)
    {
        // Long operation with cancellation support
        await UniTask.Delay(5000, cancellationToken: ct);
        return "Completed";
    }
}
```

## Best Practices

### 1. Validate Arguments

```csharp
string ValidateAndExecute(ToolCall toolCall)
{
    try
    {
        var args = JsonUtility.FromJson<MyArgs>(toolCall.Function.Arguments);
        
        if (string.IsNullOrEmpty(args.requiredField))
        {
            return "Error: requiredField is required";
        }
        
        if (args.value < 0 || args.value > 100)
        {
            return "Error: value must be between 0 and 100";
        }
        
        // Execute
        return Execute(args);
    }
    catch (Exception ex)
    {
        return $"Error: Invalid arguments - {ex.Message}";
    }
}
```

### 2. Return Structured Results

```csharp
string GetStructuredResult()
{
    var result = new
    {
        success = true,
        message = "Operation completed",
        data = new
        {
            itemsProcessed = 42,
            duration = 1.5f
        }
    };
    
    return JsonUtility.ToJson(result);
}
```

### 3. Log Operations

```csharp
public async UniTask<string> ExecuteAsync(ToolCall toolCall)
{
    Debug.Log($"[{toolCall.Function.Name}] Starting...");
    
    var stopwatch = System.Diagnostics.Stopwatch.StartNew();
    
    try
    {
        var result = await PerformOperation(toolCall);
        stopwatch.Stop();
        
        Debug.Log($"[{toolCall.Function.Name}] Completed in {stopwatch.ElapsedMilliseconds}ms");
        return result;
    }
    catch (Exception ex)
    {
        stopwatch.Stop();
        Debug.LogError($"[{toolCall.Function.Name}] Failed after {stopwatch.ElapsedMilliseconds}ms: {ex.Message}");
        throw;
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class GameToolExecutor : IToolExecutor
{
    public string[] SupportedTools => new[]
    {
        "get_player_stats",
        "spawn_enemy",
        "change_difficulty",
        "save_game",
        "load_game"
    };
    
    private readonly GameManager gameManager;
    private readonly Player player;
    
    public GameToolExecutor(GameManager gameManager, Player player)
    {
        this.gameManager = gameManager;
        this.player = player;
    }
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        Debug.Log($"🔧 Executing: {toolCall.Function.Name}");
        
        try
        {
            return toolCall.Function.Name switch
            {
                "get_player_stats" => GetPlayerStats(),
                "spawn_enemy" => SpawnEnemy(toolCall),
                "change_difficulty" => ChangeDifficulty(toolCall),
                "save_game" => await SaveGame(),
                "load_game" => await LoadGame(toolCall),
                _ => "Unknown tool"
            };
        }
        catch (Exception ex)
        {
            Debug.LogException(ex);
            return $"Error: {ex.Message}";
        }
    }
    
    string GetPlayerStats()
    {
        return JsonUtility.ToJson(new
        {
            health = player.Health,
            maxHealth = player.MaxHealth,
            level = player.Level,
            experience = player.Experience,
            position = player.transform.position
        });
    }
    
    string SpawnEnemy(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<SpawnArgs>(toolCall.Function.Arguments);
        
        for (int i = 0; i < args.count; i++)
        {
            gameManager.SpawnEnemy(args.enemyType);
        }
        
        return $"Spawned {args.count} {args.enemyType}";
    }
    
    string ChangeDifficulty(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<DifficultyArgs>(toolCall.Function.Arguments);
        gameManager.SetDifficulty(args.difficulty);
        return $"Difficulty changed to {args.difficulty}";
    }
    
    async UniTask<string> SaveGame()
    {
        await gameManager.SaveAsync();
        return "Game saved successfully";
    }
    
    async UniTask<string> LoadGame(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<LoadArgs>(toolCall.Function.Arguments);
        await gameManager.LoadAsync(args.saveSlot);
        return $"Loaded save slot {args.saveSlot}";
    }
    
    [System.Serializable]
    class SpawnArgs
    {
        public string enemyType;
        public int count;
    }
    
    [System.Serializable]
    class DifficultyArgs
    {
        public string difficulty;
    }
    
    [System.Serializable]
    class LoadArgs
    {
        public int saveSlot;
    }
}
```

## Next Steps

- [Function Calls](tool-calls/function.md)
- [Hosted Tools](hosted-tools/)
- [Unhandled Tool Calls](unhandled-tool-calls.md)
