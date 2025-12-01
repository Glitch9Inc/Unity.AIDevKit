# Code Execution

Execute code using Google's code execution tool.

## Overview

Code Execution allows agents to:

- Run Python code in Google's sandbox
- Perform calculations and data analysis
- Process game data
- Generate procedural content
- Test algorithms

## Basic Setup

### Enable Code Execution

```csharp
agent.AddGoogleTool(GoogleToolType.CodeExecution);
```

## Execute Code

### Simple Calculations

```csharp
await agent.SendAsync("Calculate the fibonacci sequence up to 100");
await agent.SendAsync("Generate 10 random numbers between 1 and 100");
```

### Data Processing

```csharp
await agent.SendAsync(@"
Process this player data and calculate statistics:
Player scores: 100, 250, 175, 300, 425, 150

Calculate:
- Average score
- Median score
- Standard deviation
- Top 3 scores
");
```

## Game Data Analysis

### Balance Calculations

```csharp
public async void AnalyzeWeaponBalance(List<WeaponData> weapons)
{
    string weaponData = string.Join("\n", weapons.Select(w =>
        $"{w.Name}: Damage={w.Damage}, Speed={w.Speed}, Range={w.Range}"));
    
    await agent.SendAsync($@"
Analyze weapon balance:

{weaponData}

Calculate:
1. DPS (damage per second) for each weapon
2. Effective range scores
3. Balance recommendations
4. Identify overpowered/underpowered weapons
");
}
```

### Economy Simulation

```csharp
public async void SimulateEconomy(EconomySettings settings)
{
    await agent.SendAsync($@"
Simulate game economy with these settings:
- Starting gold: {settings.StartingGold}
- Daily income: {settings.DailyIncome}
- Item costs: {string.Join(", ", settings.ItemCosts)}
- Inflation rate: {settings.InflationRate}%

Simulate 30 days and provide:
1. Gold progression graph
2. Purchasing power over time
3. Balance recommendations
");
}
```

### Loot Table Generation

```csharp
public async void GenerateLootTable(int playerLevel)
{
    await agent.SendAsync($@"
Generate a loot table for player level {playerLevel}:

Requirements:
- 5 common items (50% drop rate)
- 3 rare items (30% drop rate)
- 2 epic items (15% drop rate)
- 1 legendary item (5% drop rate)

Ensure total drop rates = 100%
Calculate expected drops per 100 kills
");
}
```

## Procedural Generation

### Level Layout

```csharp
public async void GenerateLevelLayout(int width, int height)
{
    await agent.SendAsync($@"
Generate a {width}x{height} dungeon layout using Python:

Rules:
- 30% walls, 70% walkable space
- At least one path from start to end
- 3-5 rooms
- Treasure room and boss room

Output as a 2D array (0=walkable, 1=wall, 2=treasure, 3=boss)
");
}
```

### Quest Rewards

```csharp
public async void CalculateQuestReward(int questLevel, int difficulty)
{
    await agent.SendAsync($@"
Calculate quest reward for:
- Quest level: {questLevel}
- Difficulty: {difficulty}/10

Formula: base_reward * level_multiplier * difficulty_modifier

Provide:
- Gold reward
- XP reward
- Loot tier
");
}
```

## Statistical Analysis

### Player Progression Analysis

```csharp
public async void AnalyzeProgression(List<PlayerLevelData> data)
{
    string csvData = "Level,XP Required,Time to Complete\n";
    csvData += string.Join("\n", data.Select(d =>
        $"{d.Level},{d.XPRequired},{d.TimeToComplete}"));
    
    await agent.SendAsync($@"
Analyze player progression curve:

{csvData}

Calculate:
1. Average time per level
2. XP curve slope
3. Identify difficulty spikes
4. Recommend curve adjustments
");
}
```

### Combat Metrics

```csharp
public async void AnalyzeCombatMetrics(CombatLog[] combatLogs)
{
    string logData = string.Join("\n", combatLogs.Select(log =>
        $"Duration: {log.Duration}s, Damage: {log.TotalDamage}, Deaths: {log.Deaths}"));
    
    await agent.SendAsync($@"
Analyze combat encounters:

{logData}

Calculate:
1. Average encounter duration
2. DPS trends
3. Survival rate
4. Difficulty rating
");
}
```

## Probability Calculations

### Gacha/Loot Box Rates

```csharp
public async void CalculateGachaRates(GachaSettings settings)
{
    await agent.SendAsync($@"
Calculate gacha probabilities:

Item Rates:
- Common: {settings.CommonRate}%
- Rare: {settings.RareRate}%
- Epic: {settings.EpicRate}%
- Legendary: {settings.LegendaryRate}%

Calculate:
1. Expected pulls for each rarity
2. Cost to guarantee legendary (with pity system at {settings.PityCount} pulls)
3. Probability distributions
");
}
```

### Critical Hit Analysis

```csharp
public async void AnalyzeCriticalHits(float critChance, float critMultiplier)
{
    await agent.SendAsync($@"
Analyze critical hit system:
- Crit chance: {critChance}%
- Crit multiplier: {critMultiplier}x

Calculate:
1. Average damage increase
2. DPS impact
3. Optimal crit chance/multiplier balance
");
}
```

## Algorithm Testing

### Pathfinding Comparison

```csharp
public async void ComparePathfinding(int gridSize)
{
    await agent.SendAsync($@"
Compare pathfinding algorithms on a {gridSize}x{gridSize} grid:

Algorithms: A*, Dijkstra, BFS

Generate random obstacles and compare:
1. Path length
2. Execution time
3. Memory usage
4. Optimal use cases
");
}
```

### Sorting Performance

```csharp
public async void TestSortingAlgorithms(int dataSize)
{
    await agent.SendAsync($@"
Test sorting algorithms with {dataSize} items:

Compare: QuickSort, MergeSort, HeapSort

Provide:
1. Execution time
2. Memory usage
3. Best/worst case performance
4. Recommendation for game use
");
}
```

## Data Transformation

### CSV Processing

```csharp
public async void ProcessGameData(string csvData)
{
    await agent.SendAsync($@"
Process this game data CSV:

{csvData}

Tasks:
1. Clean invalid entries
2. Calculate summary statistics
3. Identify outliers
4. Generate visualizable format
");
}
```

### JSON Transformation

```csharp
public async void TransformJsonData(string jsonData, string targetFormat)
{
    await agent.SendAsync($@"
Transform this JSON data:

{jsonData}

Convert to {targetFormat} format suitable for Unity
");
}
```

## Error Handling

### Handle Execution Errors

```csharp
agent.onCodeExecutionError.AddListener(error =>
{
    Debug.LogError($"Code execution failed: {error}");
    
    if (error.Contains("timeout"))
    {
        ShowMessage("Code execution timed out");
    }
    else if (error.Contains("syntax"))
    {
        ShowMessage("Code syntax error");
    }
    else if (error.Contains("memory"))
    {
        ShowMessage("Out of memory");
    }
    else
    {
        ShowMessage("Execution failed");
    }
});
```

## Results Processing

### Parse Execution Results

```csharp
agent.onCodeExecutionCompleted.AddListener(result =>
{
    Debug.Log($"Code executed successfully");
    Debug.Log($"Output: {result.Output}");
    
    // Parse numeric results
    if (float.TryParse(result.Output, out float value))
    {
        ProcessNumericResult(value);
    }
    
    // Parse array results
    if (result.Output.Contains("["))
    {
        ProcessArrayResult(result.Output);
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class CodeExecutor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField codeInput;
    [SerializeField] private Button executeButton;
    [SerializeField] private TMP_Text outputDisplay;
    
    async void Start()
    {
        await SetupCodeExecution();
        
        executeButton.onClick.AddListener(OnExecuteClicked);
    }
    
    async UniTask SetupCodeExecution()
    {
        // Add Code Execution tool
        agent.AddGoogleTool(GoogleToolType.CodeExecution);
        
        // Listen for results
        agent.onCodeExecutionCompleted.AddListener(OnExecutionCompleted);
        agent.onCodeExecutionError.AddListener(OnExecutionError);
        
        Debug.Log("✓ Code execution ready");
    }
    
    async void OnExecuteClicked()
    {
        string code = codeInput.text.Trim();
        
        if (string.IsNullOrEmpty(code))
        {
            ShowMessage("Please enter code to execute");
            return;
        }
        
        Debug.Log($"▶️ Executing code...");
        executeButton.interactable = false;
        outputDisplay.text = "Executing...";
        
        await agent.SendAsync($@"
Execute this Python code:

```python
{code}
```

Provide the output
");
    }

    void OnExecutionCompleted(CodeExecutionResult result)
    {
        Debug.Log($"✓ Execution completed");
        Debug.Log($"Output: {result.Output}");
        
        outputDisplay.text = result.Output;
        executeButton.interactable = true;
    }
    
    void OnExecutionError(string error)
    {
        Debug.LogError($"Execution error: {error}");
        outputDisplay.text = $"<color=red>Error: {error}</color>";
        executeButton.interactable = true;
    }
    
    void ShowMessage(string message)
    {
        outputDisplay.text = message;
    }
    
    // Helper methods for common calculations
    public async void CalculateDPS(float damage, float attackSpeed)
    {
        await agent.SendAsync($@"
Calculate DPS (Damage Per Second):

- Damage: {damage}
- Attack Speed: {attackSpeed} attacks/second

Formula: DPS = Damage × Attack Speed
");
    }

    public async void CalculateDropRates(Dictionary<string, float> rates)
    {
        string ratesData = string.Join("\n", rates.Select(r =>
            $"{r.Key}: {r.Value}%"));
        
        await agent.SendAsync($@"
Validate and calculate loot drop rates:

{ratesData}

Verify:

1. Total = 100%
2. Calculate expected drops per 100 attempts
3. Recommend adjustments if imbalanced
");
    }

    public async void SimulateCombat(CombatStats attacker, CombatStats defender, int rounds)
    {
        await agent.SendAsync($@"
Simulate {rounds} rounds of combat:

Attacker:

- HP: {attacker.HP}
- Damage: {attacker.Damage}
- Crit Chance: {attacker.CritChance}%
- Crit Multiplier: {attacker.CritMultiplier}x

Defender:

- HP: {defender.HP}
- Defense: {defender.Defense}
- Evasion: {defender.Evasion}%

Calculate:

1. Win rate
2. Average combat duration
3. Damage distribution
4. Balance assessment
");
    }
}

[System.Serializable]
public class CombatStats
{
    public int HP;
    public int Damage;
    public float CritChance;
    public float CritMultiplier;
    public int Defense;
    public float Evasion;
}

```

## Next Steps

- [Google Search](google-search.md)
- [URL Context](url-context.md)
- [Code Interpreter (Hosted)](../hosted-tools/code-interpreter.md)
