# Function Tool Calls

Execute C# functions through agent tool calls.

## Basic Function Tools

### Simple Function

```csharp
// Define function
int Add(int a, int b) => a + b;

// Register as tool
agent.AddTool("add", Add);

// Agent can now call: add(5, 3) → 8
```

### With Description

```csharp
agent.AddTool(
    name: "calculate_damage",
    description: "Calculate damage dealt to enemy based on attack and defense stats",
    function: (int attack, int defense) =>
    {
        int damage = Mathf.Max(0, attack - defense);
        return $"Dealt {damage} damage";
    }
);
```

## Lambda Functions

### Inline Implementation

```csharp
// Simple lambda
agent.AddTool("get_time", () => DateTime.Now.ToString("HH:mm:ss"));

// Multi-line lambda
agent.AddTool("spawn_enemies", (string type, int count) =>
{
    for (int i = 0; i < count; i++)
    {
        SpawnEnemy(type);
    }
    return $"Spawned {count} {type}";
});
```

### Closure Variables

```csharp
int score = 0;

agent.AddTool("add_score", (int points) =>
{
    score += points;
    return $"Score: {score}";
});

agent.AddTool("get_score", () => $"Current score: {score}");
```

## Method References

### Instance Methods

```csharp
public class GameManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("save_game", SaveGame);
        agent.AddTool("load_game", LoadGame);
        agent.AddTool("get_stats", GetPlayerStats);
    }
    
    string SaveGame()
    {
        // Save logic
        PlayerPrefs.Save();
        return "Game saved successfully";
    }
    
    string LoadGame(int slot)
    {
        // Load logic
        return $"Loaded save slot {slot}";
    }
    
    string GetPlayerStats()
    {
        return JsonUtility.ToJson(new
        {
            level = player.Level,
            health = player.Health
        });
    }
}
```

### Static Methods

```csharp
public static class GameUtilities
{
    public static string GetSystemInfo()
    {
        return $"OS: {SystemInfo.operatingSystem}\n" +
               $"CPU: {SystemInfo.processorType}\n" +
               $"RAM: {SystemInfo.systemMemorySize} MB";
    }
    
    public static float CalculateDistance(Vector3 a, Vector3 b)
    {
        return Vector3.Distance(a, b);
    }
}

// Register
agent.AddTool("get_system_info", GameUtilities.GetSystemInfo);
agent.AddTool("calculate_distance", GameUtilities.CalculateDistance);
```

## Async Functions

### Async/Await

```csharp
async UniTask<string> FetchDataAsync(string url)
{
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

// Register async function
agent.AddTool("fetch_data", FetchDataAsync);
```

### Long-Running Operations

```csharp
async UniTask<string> ProcessLargeDataset(string datasetName)
{
    var data = LoadDataset(datasetName);
    
    int progress = 0;
    int total = data.Count;
    
    foreach (var item in data)
    {
        await ProcessItem(item);
        progress++;
        
        if (progress % 10 == 0)
        {
            Debug.Log($"Progress: {progress}/{total}");
        }
    }
    
    return $"Processed {total} items";
}

agent.AddTool("process_dataset", ProcessLargeDataset);
```

## Parameter Types

### Primitive Types

```csharp
// Int, float, string, bool
agent.AddTool("set_volume", (float volume) =>
{
    AudioListener.volume = volume;
    return $"Volume set to {volume}";
});

agent.AddTool("toggle_feature", (bool enabled) =>
{
    gameManager.FeatureEnabled = enabled;
    return $"Feature {(enabled ? "enabled" : "disabled")}";
});
```

### Complex Types

```csharp
[System.Serializable]
public class SpawnConfig
{
    public string enemyType;
    public int count;
    public Vector3 position;
}

agent.AddTool("spawn_with_config", (SpawnConfig config) =>
{
    for (int i = 0; i < config.count; i++)
    {
        SpawnEnemyAt(config.enemyType, config.position);
    }
    return $"Spawned {config.count} {config.enemyType}";
});
```

### Optional Parameters

```csharp
agent.AddTool("search_files", (string query, int maxResults = 10) =>
{
    var results = SearchFiles(query).Take(maxResults);
    return $"Found {results.Count()} results";
});
```

## Return Types

### String (Recommended)

```csharp
agent.AddTool("get_info", () => "Information text");
```

### JSON Objects

```csharp
agent.AddTool("get_player_data", () =>
{
    var data = new
    {
        name = player.Name,
        level = player.Level,
        health = player.Health,
        position = player.transform.position
    };
    
    return JsonUtility.ToJson(data);
});
```

### Collections

```csharp
agent.AddTool("list_inventory", () =>
{
    var items = player.Inventory.GetAllItems();
    return JsonUtility.ToJson(new { items, count = items.Count });
});
```

## Error Handling

### Try-Catch

```csharp
agent.AddTool("risky_operation", (string input) =>
{
    try
    {
        var result = PerformRiskyOperation(input);
        return $"Success: {result}";
    }
    catch (ArgumentException ex)
    {
        return $"Error: Invalid input - {ex.Message}";
    }
    catch (Exception ex)
    {
        Debug.LogException(ex);
        return $"Error: {ex.Message}";
    }
});
```

### Validation

```csharp
agent.AddTool("set_difficulty", (int level) =>
{
    if (level < 1 || level > 10)
    {
        return "Error: Difficulty must be between 1 and 10";
    }
    
    gameManager.SetDifficulty(level);
    return $"Difficulty set to {level}";
});
```

## Game-Specific Functions

### Player Actions

```csharp
public class PlayerActions : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Player player;
    
    void Start()
    {
        RegisterPlayerTools();
    }
    
    void RegisterPlayerTools()
    {
        agent.AddTool("heal_player", HealPlayer);
        agent.AddTool("give_item", GiveItem);
        agent.AddTool("teleport", TeleportPlayer);
        agent.AddTool("set_health", SetHealth);
    }
    
    string HealPlayer(int amount)
    {
        int oldHealth = player.Health;
        player.Heal(amount);
        int newHealth = player.Health;
        
        return $"Healed {newHealth - oldHealth} HP (now at {newHealth})";
    }
    
    string GiveItem(string itemName, int quantity)
    {
        bool success = player.Inventory.AddItem(itemName, quantity);
        
        if (success)
        {
            return $"Added {quantity}x {itemName} to inventory";
        }
        else
        {
            return $"Failed to add {itemName} (inventory full?)";
        }
    }
    
    string TeleportPlayer(float x, float y, float z)
    {
        Vector3 position = new Vector3(x, y, z);
        player.transform.position = position;
        
        return $"Teleported to {position}";
    }
    
    string SetHealth(int health)
    {
        health = Mathf.Clamp(health, 0, player.MaxHealth);
        player.Health = health;
        
        return $"Health set to {health}/{player.MaxHealth}";
    }
}
```

### World Interaction

```csharp
public class WorldTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("find_nearest", FindNearest);
        agent.AddTool("get_distance", GetDistance);
        agent.AddTool("list_objects", ListObjects);
    }
    
    string FindNearest(string tag)
    {
        var objects = GameObject.FindGameObjectsWithTag(tag);
        if (objects.Length == 0)
        {
            return $"No objects found with tag '{tag}'";
        }
        
        var player = GameObject.FindGameObjectWithTag("Player");
        var nearest = objects
            .OrderBy(obj => Vector3.Distance(player.transform.position, obj.transform.position))
            .First();
        
        float distance = Vector3.Distance(player.transform.position, nearest.transform.position);
        
        return $"Nearest {tag}: {nearest.name} at {distance:F1}m";
    }
    
    string GetDistance(string objectName)
    {
        var obj = GameObject.Find(objectName);
        if (obj == null)
        {
            return $"Object '{objectName}' not found";
        }
        
        var player = GameObject.FindGameObjectWithTag("Player");
        float distance = Vector3.Distance(player.transform.position, obj.transform.position);
        
        return $"Distance to {objectName}: {distance:F1}m";
    }
    
    string ListObjects(string tag)
    {
        var objects = GameObject.FindGameObjectsWithTag(tag);
        
        if (objects.Length == 0)
        {
            return $"No objects with tag '{tag}'";
        }
        
        var names = objects.Select(obj => obj.name);
        return $"Found {objects.Length} objects: {string.Join(", ", names)}";
    }
}
```

## UI Functions

### UI Manipulation

```csharp
public class UITools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject inventoryPanel;
    [SerializeField] private GameObject settingsPanel;
    
    void Start()
    {
        agent.AddTool("show_inventory", () =>
        {
            inventoryPanel.SetActive(true);
            return "Inventory opened";
        });
        
        agent.AddTool("show_settings", () =>
        {
            settingsPanel.SetActive(true);
            return "Settings opened";
        });
        
        agent.AddTool("show_message", ShowMessage);
    }
    
    string ShowMessage(string title, string message)
    {
        // Show message dialog
        DialogManager.Instance.Show(title, message);
        return "Message displayed";
    }
}
```

## Data Access Functions

### Database Queries

```csharp
public class DataTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    private IDatabase database;
    
    void Start()
    {
        agent.AddTool("query_users", QueryUsers);
        agent.AddTool("get_user", GetUser);
        agent.AddTool("count_records", CountRecords);
    }
    
    async UniTask<string> QueryUsers(string searchTerm)
    {
        var users = await database.QueryAsync<User>(
            $"SELECT * FROM Users WHERE Name LIKE '%{searchTerm}%'"
        );
        
        return JsonUtility.ToJson(new
        {
            count = users.Count,
            users = users.Take(10) // Limit results
        });
    }
    
    async UniTask<string> GetUser(int userId)
    {
        var user = await database.GetAsync<User>(userId);
        
        if (user == null)
        {
            return $"User {userId} not found";
        }
        
        return JsonUtility.ToJson(user);
    }
    
    async UniTask<string> CountRecords(string tableName)
    {
        int count = await database.CountAsync(tableName);
        return $"{tableName}: {count} records";
    }
}
```

## Testing Functions

### Debug Tools

```csharp
agent.AddTool("debug_info", () =>
{
    return $@"FPS: {1f / Time.deltaTime:F0}
Memory: {System.GC.GetTotalMemory(false) / 1024 / 1024}MB
Active GameObjects: {FindObjectsOfType<GameObject>().Length}
Time: {Time.time:F2}s";
});

agent.AddTool("log_message", (string message) =>
{
    Debug.Log($"[Agent] {message}");
    return "Message logged";
});

agent.AddTool("trigger_event", (string eventName) =>
{
    EventBus.Trigger(eventName);
    return $"Triggered event: {eventName}";
});
```

## Best Practices

### 1. Clear Naming

```csharp
// ❌ Bad
agent.AddTool("do", DoSomething);

// ✅ Good
agent.AddTool("spawn_enemy_at_location", SpawnEnemyAtLocation);
```

### 2. Descriptive Returns

```csharp
// ❌ Bad
return "ok";

// ✅ Good
return "Successfully spawned 5 enemies at (10, 0, 15)";
```

### 3. Input Validation

```csharp
agent.AddTool("set_difficulty", (int level) =>
{
    // Validate input
    if (level < 1 || level > 5)
    {
        return "Error: Difficulty must be between 1 and 5";
    }
    
    // Execute
    SetDifficulty(level);
    return $"Difficulty set to {level}";
});
```

### 4. Meaningful Errors

```csharp
try
{
    return ExecuteOperation();
}
catch (FileNotFoundException ex)
{
    return $"Error: File not found - {ex.FileName}";
}
catch (UnauthorizedAccessException)
{
    return "Error: Permission denied";
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class GameFunctionTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Player player;
    [SerializeField] private GameManager gameManager;
    
    void Start()
    {
        RegisterAllTools();
    }
    
    void RegisterAllTools()
    {
        // Player tools
        agent.AddTool("get_player_stats", GetPlayerStats);
        agent.AddTool("heal_player", HealPlayer);
        agent.AddTool("teleport_player", TeleportPlayer);
        
        // Game tools
        agent.AddTool("spawn_enemy", SpawnEnemy);
        agent.AddTool("set_difficulty", SetDifficulty);
        agent.AddTool("save_game", SaveGame);
        agent.AddTool("load_game", LoadGame);
        
        // Query tools
        agent.AddTool("find_nearest", FindNearest);
        agent.AddTool("count_enemies", CountEnemies);
        
        Debug.Log("✓ Registered 9 function tools");
    }
    
    // Player tools
    string GetPlayerStats()
    {
        return JsonUtility.ToJson(new
        {
            name = player.Name,
            health = player.Health,
            maxHealth = player.MaxHealth,
            level = player.Level,
            experience = player.Experience,
            position = player.transform.position
        });
    }
    
    string HealPlayer(int amount)
    {
        int healed = player.Heal(amount);
        return $"Healed {healed} HP. Current health: {player.Health}/{player.MaxHealth}";
    }
    
    string TeleportPlayer(float x, float y, float z)
    {
        player.transform.position = new Vector3(x, y, z);
        return $"Teleported to ({x}, {y}, {z})";
    }
    
    // Game tools
    string SpawnEnemy(string enemyType, int count)
    {
        if (count < 1 || count > 10)
        {
            return "Error: Count must be between 1 and 10";
        }
        
        for (int i = 0; i < count; i++)
        {
            gameManager.SpawnEnemy(enemyType);
        }
        
        return $"Spawned {count} {enemyType}";
    }
    
    string SetDifficulty(string difficulty)
    {
        if (!gameManager.IsValidDifficulty(difficulty))
        {
            return $"Error: Invalid difficulty. Valid options: Easy, Normal, Hard";
        }
        
        gameManager.SetDifficulty(difficulty);
        return $"Difficulty set to {difficulty}";
    }
    
    async UniTask<string> SaveGame()
    {
        try
        {
            await gameManager.SaveAsync();
            return "Game saved successfully";
        }
        catch (Exception ex)
        {
            return $"Failed to save: {ex.Message}";
        }
    }
    
    async UniTask<string> LoadGame(int slot)
    {
        try
        {
            await gameManager.LoadAsync(slot);
            return $"Loaded save slot {slot}";
        }
        catch (Exception ex)
        {
            return $"Failed to load: {ex.Message}";
        }
    }
    
    // Query tools
    string FindNearest(string tag)
    {
        var objects = GameObject.FindGameObjectsWithTag(tag);
        if (objects.Length == 0)
        {
            return $"No objects found with tag '{tag}'";
        }
        
        var nearest = objects
            .OrderBy(obj => Vector3.Distance(player.transform.position, obj.transform.position))
            .First();
        
        float distance = Vector3.Distance(player.transform.position, nearest.transform.position);
        return $"{nearest.name} at {distance:F1}m";
    }
    
    string CountEnemies()
    {
        int count = GameObject.FindGameObjectsWithTag("Enemy").Length;
        return $"Active enemies: {count}";
    }
}
```

## Next Steps

- [Local Shell](local-shell.md)
- [Computer Use](computer-use.md)
- [Registering Executors](../registering-executors.md)
