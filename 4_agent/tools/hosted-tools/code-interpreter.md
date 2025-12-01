# Code Interpreter (Hosted Tool)

Execute Python code in a secure sandboxed environment using OpenAI's Code Interpreter.

## Overview

Code Interpreter allows agents to:

- Run Python code
- Perform data analysis
- Create visualizations
- Process files
- Generate charts and graphs
- Solve complex calculations

## Basic Setup

### Enable Code Interpreter

```csharp
// Add code interpreter tool
agent.AddTool(ToolType.CodeInterpreter);
```

## Simple Calculations

### Math Operations

```csharp
await agent.SendAsync("Calculate the factorial of 20");
await agent.SendAsync("What is the 100th Fibonacci number?");
await agent.SendAsync("Solve the quadratic equation: x² + 5x + 6 = 0");
```

### Complex Math

```csharp
await agent.SendAsync(@"
Calculate:
1. The sum of all prime numbers between 1 and 1000
2. The average of those primes
3. The standard deviation
");
```

## Data Analysis

### Analyze CSV Data

```csharp
// Upload CSV file
string fileId = await agent.UploadFileAsync("player_stats.csv");

// Ask for analysis
await agent.SendAsync("Analyze the player stats CSV and show key statistics");
await agent.SendAsync("What are the top 10 players by score?");
await agent.SendAsync("Calculate the average playtime");
```

### Statistical Analysis

```csharp
await agent.SendAsync(@"
Analyze the data in the uploaded file:
1. Calculate mean, median, and mode
2. Find outliers
3. Show distribution
4. Generate summary statistics
");
```

## Visualizations

### Create Charts

```csharp
await agent.SendAsync("Create a bar chart of player scores from the CSV");
await agent.SendAsync("Generate a line graph showing level progression over time");
await agent.SendAsync("Make a pie chart of player classes distribution");
```

### Download Visualizations

```csharp
agent.onResponseCompleted.AddListener(async response =>
{
    if (response.OutputFiles != null && response.OutputFiles.Count > 0)
    {
        foreach (var file in response.OutputFiles)
        {
            Debug.Log($"Generated file: {file.FileName}");
            
            // Download file
            byte[] data = await agent.DownloadFileAsync(file.Id);
            
            // Save locally
            string savePath = Path.Combine(
                Application.persistentDataPath,
                file.FileName
            );
            File.WriteAllBytes(savePath, data);
            
            Debug.Log($"Saved to: {savePath}");
        }
    }
});
```

## File Processing

### Process Uploaded Files

```csharp
// Upload data file
string fileId = await agent.UploadFileAsync("game_data.json");

// Process it
await agent.SendAsync("Parse the JSON file and extract all weapon stats");
await agent.SendAsync("Convert the data to CSV format");
await agent.SendAsync("Find all items with rarity >= 4");
```

### Transform Data

```csharp
await agent.SendAsync(@"
Process the uploaded Excel file:
1. Remove duplicate entries
2. Sort by date
3. Calculate monthly totals
4. Export as CSV
");
```

## Game-Specific Use Cases

### Balance Analysis

```csharp
public class GameBalanceAnalyzer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void Start()
    {
        agent.AddTool(ToolType.CodeInterpreter);
    }
    
    public async void AnalyzeWeaponBalance()
    {
        // Upload weapon data
        string fileId = await agent.UploadFileAsync("weapons.json");
        
        await agent.SendAsync(@"
Analyze weapon balance:
1. Calculate DPS for each weapon
2. Find weapons that are too powerful or weak
3. Suggest balance adjustments
4. Create a comparison chart
");
    }
    
    public async void AnalyzePlayerProgression()
    {
        string fileId = await agent.UploadFileAsync("player_progression.csv");
        
        await agent.SendAsync(@"
Analyze player progression data:
1. Calculate average time per level
2. Identify difficulty spikes
3. Show player retention by level
4. Generate progression curve chart
");
    }
}
```

### Economy Analysis

```csharp
public async void AnalyzeGameEconomy()
{
    // Upload economy data
    await agent.UploadFileAsync("transactions.csv");
    await agent.UploadFileAsync("item_prices.json");
    
    await agent.SendAsync(@"
Analyze the game economy:
1. Calculate currency inflation rate
2. Find items with unusual pricing
3. Analyze transaction patterns
4. Suggest price adjustments
5. Create price history charts
");
}
```

## Advanced Usage

### Custom Python Code

```csharp
await agent.SendAsync(@"
Run this Python code:

```python
import numpy as np
import matplotlib.pyplot as plt

# Generate sample data
x = np.linspace(0, 10, 100)
y = np.sin(x)

# Create plot
plt.figure(figsize=(10, 6))
plt.plot(x, y)
plt.title('Sine Wave')
plt.xlabel('X')
plt.ylabel('Y')
plt.grid(True)
plt.savefig('sine_wave.png')
```

Save the plot as an image.
");

```

### Data Science Libraries

```csharp
// Code Interpreter has access to:
// - NumPy (numerical computing)
// - Pandas (data analysis)
// - Matplotlib (plotting)
// - Scikit-learn (machine learning)
// - SciPy (scientific computing)

await agent.SendAsync(@"
Use pandas and scikit-learn to:
1. Load the CSV data
2. Clean and normalize it
3. Perform k-means clustering
4. Visualize the clusters
");
```

## File Management

### Handle Output Files

```csharp
public class CodeInterpreterFileHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string outputPath = "OutputFiles";
    
    void Start()
    {
        agent.AddTool(ToolType.CodeInterpreter);
        agent.onResponseCompleted.AddListener(HandleOutputFiles);
    }
    
    async void HandleOutputFiles(Response response)
    {
        if (response.OutputFiles == null || response.OutputFiles.Count == 0)
            return;
        
        Debug.Log($"📁 {response.OutputFiles.Count} file(s) generated");
        
        // Ensure output directory exists
        Directory.CreateDirectory(outputPath);
        
        foreach (var file in response.OutputFiles)
        {
            try
            {
                // Download file
                byte[] data = await agent.DownloadFileAsync(file.Id);
                
                // Save file
                string savePath = Path.Combine(outputPath, file.FileName);
                File.WriteAllBytes(savePath, data);
                
                Debug.Log($"✓ Saved: {savePath}");
                
                // Load as texture if image
                if (IsImageFile(file.FileName))
                {
                    LoadImage(savePath);
                }
            }
            catch (Exception ex)
            {
                Debug.LogError($"Failed to download {file.FileName}: {ex.Message}");
            }
        }
    }
    
    bool IsImageFile(string fileName)
    {
        string ext = Path.GetExtension(fileName).ToLower();
        return ext == ".png" || ext == ".jpg" || ext == ".jpeg";
    }
    
    void LoadImage(string path)
    {
        byte[] imageData = File.ReadAllBytes(path);
        Texture2D texture = new Texture2D(2, 2);
        texture.LoadImage(imageData);
        
        // Display image
        DisplayImage(texture);
    }
    
    void DisplayImage(Texture2D texture)
    {
        // Show in UI
        Debug.Log($"📊 Image loaded: {texture.width}x{texture.height}");
    }
}
```

## Error Handling

### Execution Errors

```csharp
agent.onToolCallError.AddListener((toolCall, error) =>
{
    if (toolCall.Function.Name == "code_interpreter")
    {
        Debug.LogError($"Code execution failed: {error}");
        
        // Parse error message
        if (error.Contains("SyntaxError"))
        {
            ShowMessage("Python syntax error in generated code");
        }
        else if (error.Contains("MemoryError"))
        {
            ShowMessage("Code execution ran out of memory");
        }
        else if (error.Contains("TimeoutError"))
        {
            ShowMessage("Code execution timed out");
        }
        else
        {
            ShowMessage("Code execution failed");
        }
    }
});
```

## Limitations

### Resource Limits

```csharp
// Code Interpreter limitations:
// - Maximum execution time: ~120 seconds
// - Memory: Limited (varies)
// - Disk space: ~2GB for input/output files
// - No network access
// - No file system access outside sandbox
```

### Handle Timeouts

```csharp
public async UniTask ExecuteWithTimeout(string code, int timeoutSeconds = 120)
{
    var cts = new CancellationTokenSource();
    cts.CancelAfter(TimeSpan.FromSeconds(timeoutSeconds));
    
    try
    {
        await agent.SendAsync(code, cancellationToken: cts.Token);
    }
    catch (OperationCanceledException)
    {
        Debug.LogWarning("Code execution timed out");
        ShowMessage("Operation took too long and was canceled");
    }
}
```

## Best Practices

### 1. Be Specific

```csharp
// ❌ Vague
await agent.SendAsync("Analyze this data");

// ✅ Specific
await agent.SendAsync(@"
Analyze player_stats.csv:
1. Calculate average score by level
2. Show distribution of play times
3. Identify top 10% performers
4. Generate bar chart comparing classes
");
```

### 2. Handle Large Data

```csharp
// For large datasets, process in chunks
await agent.SendAsync(@"
Process the large CSV file in chunks:
1. Read 1000 rows at a time
2. Calculate statistics for each chunk
3. Combine results
4. Show final summary
");
```

### 3. Validate Results

```csharp
agent.onResponseCompleted.AddListener(response =>
{
    // Check if code was executed
    bool codeExecuted = response.ToolCalls?.Any(tc => 
        tc.Function.Name == "code_interpreter") ?? false;
    
    if (codeExecuted)
    {
        Debug.Log("✓ Code executed successfully");
        
        // Validate output files
        if (response.OutputFiles != null)
        {
            Debug.Log($"Generated {response.OutputFiles.Count} file(s)");
        }
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class CodeInterpreterManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string dataPath = "GameData";
    [SerializeField] private string outputPath = "Analysis";
    
    async void Start()
    {
        await SetupCodeInterpreter();
    }
    
    async UniTask SetupCodeInterpreter()
    {
        agent.AddTool(ToolType.CodeInterpreter);
        agent.onResponseCompleted.AddListener(HandleResponse);
        
        Debug.Log("✓ Code Interpreter ready");
    }
    
    public async void AnalyzePlayerData()
    {
        Debug.Log("📊 Analyzing player data...");
        
        try
        {
            // Upload data file
            string filePath = Path.Combine(dataPath, "player_stats.csv");
            string fileId = await agent.UploadFileAsync(filePath);
            
            Debug.Log($"✓ File uploaded: {fileId}");
            
            // Request analysis
            await agent.SendAsync(@"
Analyze the player statistics CSV file:

1. Calculate these metrics:
   - Average score per player
   - Average playtime
   - Score distribution
   - Most common player level

2. Find:
   - Top 10 players by score
   - Players with unusual patterns
   - Progression bottlenecks

3. Generate:
   - Histogram of score distribution
   - Line chart of average score by level
   - Scatter plot of playtime vs score

Save all charts as PNG files.
");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Analysis failed: {ex.Message}");
        }
    }
    
    public async void BalanceWeapons()
    {
        Debug.Log("⚖️ Analyzing weapon balance...");
        
        try
        {
            string filePath = Path.Combine(dataPath, "weapons.json");
            await agent.UploadFileAsync(filePath);
            
            await agent.SendAsync(@"
Analyze weapon balance from the JSON file:

1. Calculate for each weapon:
   - DPS (damage per second)
   - TTK (time to kill) against standard enemy
   - Cost effectiveness (DPS per resource cost)

2. Identify:
   - Overpowered weapons (DPS > 2σ above mean)
   - Underpowered weapons (DPS < 2σ below mean)
   - Balanced weapons

3. Suggest:
   - Damage adjustments for unbalanced weapons
   - Cost adjustments
   - Fire rate changes

4. Create:
   - Bar chart comparing weapon DPS
   - Scatter plot of damage vs fire rate
   - Recommendation table

Export results as CSV and charts as PNG.
");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Balance analysis failed: {ex.Message}");
        }
    }
    
    async void HandleResponse(Response response)
    {
        // Handle output files
        if (response.OutputFiles != null && response.OutputFiles.Count > 0)
        {
            Debug.Log($"📁 {response.OutputFiles.Count} file(s) generated");
            
            Directory.CreateDirectory(outputPath);
            
            foreach (var file in response.OutputFiles)
            {
                try
                {
                    byte[] data = await agent.DownloadFileAsync(file.Id);
                    string savePath = Path.Combine(outputPath, file.FileName);
                    File.WriteAllBytes(savePath, data);
                    
                    Debug.Log($"✓ Saved: {savePath}");
                    
                    // Load images
                    if (file.FileName.EndsWith(".png"))
                    {
                        LoadAndDisplayImage(savePath);
                    }
                }
                catch (Exception ex)
                {
                    Debug.LogError($"Failed to save {file.FileName}: {ex.Message}");
                }
            }
        }
    }
    
    void LoadAndDisplayImage(string path)
    {
        byte[] imageData = File.ReadAllBytes(path);
        Texture2D texture = new Texture2D(2, 2);
        texture.LoadImage(imageData);
        
        Debug.Log($"📊 Chart loaded: {texture.width}x{texture.height}");
        
        // Display in UI (implementation depends on your UI system)
        DisplayChart(texture);
    }
    
    void DisplayChart(Texture2D texture)
    {
        // Implement chart display in your UI
    }
}
```

## Next Steps

- [File Search](file-search.md)
- [Web Search](web-search.md)
- [Image Generation](image-generation.md)
