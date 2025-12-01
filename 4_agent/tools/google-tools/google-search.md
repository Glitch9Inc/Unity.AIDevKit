# Google Search

Use Google Search for real-time information and research.

## Overview

Google Search allows agents to:

- Search the web for current information
- Find documentation and tutorials
- Research game mechanics
- Look up technical information
- Verify facts and data

## Basic Setup

### Enable Google Search

```csharp
agent.AddGoogleTool(GoogleToolType.Search);
```

### Configure API Key

```csharp
agent.Settings.GoogleTools = new GoogleToolsSettings
{
    ApiKey = "your-google-api-key",
    SearchEngineId = "your-search-engine-id"
};
```

## Perform Searches

### Simple Search

```csharp
await agent.SendAsync("Search Google for Unity best practices");
await agent.SendAsync("Find information about C# async/await");
```

### With Specific Query

```csharp
await agent.SendAsync(@"
Search Google for:
'Unity ML-Agents tutorial 2024'
and summarize the key points
");
```

## Search Results

### Access Results

```csharp
agent.onGoogleSearchCompleted.AddListener(results =>
{
    Debug.Log($"Found {results.Count} results");
    
    foreach (var result in results)
    {
        Debug.Log($"Title: {result.Title}");
        Debug.Log($"URL: {result.Url}");
        Debug.Log($"Snippet: {result.Snippet}");
    }
});
```

### Display Results in UI

```csharp
public class SearchResultsUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject resultPrefab;
    [SerializeField] private Transform resultsContainer;
    
    void Start()
    {
        agent.onGoogleSearchCompleted.AddListener(DisplayResults);
    }
    
    void DisplayResults(List<SearchResult> results)
    {
        // Clear previous results
        foreach (Transform child in resultsContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Display new results
        foreach (var result in results)
        {
            var resultObj = Instantiate(resultPrefab, resultsContainer);
            
            var title = resultObj.transform.Find("Title").GetComponent<TMP_Text>();
            var snippet = resultObj.transform.Find("Snippet").GetComponent<TMP_Text>();
            var button = resultObj.GetComponent<Button>();
            
            title.text = result.Title;
            snippet.text = result.Snippet;
            
            button.onClick.AddListener(() => Application.OpenURL(result.Url));
        }
    }
}
```

## Game Development Use Cases

### Asset Store Search

```csharp
public async void SearchUnityAssets(string query)
{
    await agent.SendAsync($@"
Search Google for Unity Asset Store assets related to: '{query}'
Focus on:
- Asset name and price
- Rating and reviews
- Compatible Unity versions
- Publisher
");
}

// Usage
SearchUnityAssets("particle effects");
SearchUnityAssets("inventory system");
```

### Documentation Lookup

```csharp
public async void FindDocumentation(string topic)
{
    await agent.SendAsync($@"
Search for Unity documentation on: '{topic}'
Include:
- Official Unity docs
- API reference
- Code examples
- Best practices
");
}

// Usage
FindDocumentation("NavMesh Agent");
FindDocumentation("Scriptable Objects");
```

### Error Research

```csharp
public async void ResearchError(string errorMessage)
{
    await agent.SendAsync($@"
Search for solutions to this Unity error:
'{errorMessage}'

Find:
- Common causes
- Solutions
- Related forum discussions
- Code fixes
");
}

// Usage
ResearchError("NullReferenceException at line 42");
```

### Package Compatibility

```csharp
public async void CheckCompatibility(string packageName, string unityVersion)
{
    await agent.SendAsync($@"
Search if '{packageName}' is compatible with Unity {unityVersion}
Look for:
- Official compatibility information
- Known issues
- User experiences
- Alternative packages if incompatible
");
}

// Usage
CheckCompatibility("UniTask", "2022.3");
```

## Advanced Search

### Custom Search Parameters

```csharp
public async void AdvancedSearch(string query, SearchOptions options)
{
    string searchQuery = query;
    
    if (options.DateRestrict != null)
    {
        searchQuery += $" after:{options.DateRestrict}";
    }
    
    if (options.SiteSearch != null)
    {
        searchQuery += $" site:{options.SiteSearch}";
    }
    
    if (options.ExcludeTerms != null)
    {
        searchQuery += $" -{options.ExcludeTerms}";
    }
    
    await agent.SendAsync($"Search Google for: {searchQuery}");
}

[System.Serializable]
public class SearchOptions
{
    public string DateRestrict;  // e.g., "2024-01-01"
    public string SiteSearch;    // e.g., "unity.com"
    public string ExcludeTerms;  // e.g., "deprecated"
}

// Usage
AdvancedSearch("Unity networking", new SearchOptions
{
    DateRestrict = "2024-01-01",
    SiteSearch = "docs.unity3d.com"
});
```

### Image Search

```csharp
public async void SearchImages(string query)
{
    await agent.SendAsync($@"
Search Google Images for: '{query}'
Find high-quality reference images suitable for game development
");
}

// Usage
SearchImages("medieval armor reference");
SearchImages("cyberpunk environment concept art");
```

## Research Assistant

### Game Mechanics Research

```csharp
public class GameResearchAssistant : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void ResearchMechanic(string mechanic)
    {
        await agent.SendAsync($@"
Research the '{mechanic}' game mechanic:

1. Search for examples in popular games
2. Find implementation tutorials
3. Look for best practices
4. Identify common pitfalls
5. Suggest Unity assets or packages

Provide a comprehensive summary with links
");
    }
}

// Usage
assistant.ResearchMechanic("inventory system");
assistant.ResearchMechanic("dialogue system");
assistant.ResearchMechanic("skill tree");
```

### Technology Comparison

```csharp
public async void CompareTechnologies(string tech1, string tech2)
{
    await agent.SendAsync($@"
Compare '{tech1}' vs '{tech2}' for Unity game development:

Search for:
- Performance differences
- Ease of use
- Community support
- Documentation quality
- Use cases
- Pros and cons

Provide a detailed comparison
");
}

// Usage
CompareTechnologies("UniTask", "Coroutines");
CompareTechnologies("Addressables", "AssetBundles");
```

## Caching and Rate Limiting

### Cache Search Results

```csharp
public class SearchCache : MonoBehaviour
{
    private Dictionary<string, CachedSearch> cache = new();
    private TimeSpan cacheExpiry = TimeSpan.FromHours(1);
    
    public async UniTask<List<SearchResult>> GetOrSearch(AgentBehaviour agent, string query)
    {
        if (cache.TryGetValue(query, out var cached))
        {
            if (DateTime.Now - cached.Timestamp < cacheExpiry)
            {
                Debug.Log("Using cached search results");
                return cached.Results;
            }
        }
        
        // Perform new search
        var results = await PerformSearch(agent, query);
        
        cache[query] = new CachedSearch
        {
            Results = results,
            Timestamp = DateTime.Now
        };
        
        return results;
    }
    
    async UniTask<List<SearchResult>> PerformSearch(AgentBehaviour agent, string query)
    {
        // Implementation
        return new List<SearchResult>();
    }
    
    struct CachedSearch
    {
        public List<SearchResult> Results;
        public DateTime Timestamp;
    }
}
```

### Rate Limiting

```csharp
public class SearchRateLimiter : MonoBehaviour
{
    [SerializeField] private int maxSearchesPerMinute = 10;
    
    private Queue<DateTime> searchTimestamps = new();
    
    public async UniTask<bool> CanSearch()
    {
        // Remove old timestamps
        while (searchTimestamps.Count > 0 &&
               DateTime.Now - searchTimestamps.Peek() > TimeSpan.FromMinutes(1))
        {
            searchTimestamps.Dequeue();
        }
        
        // Check limit
        if (searchTimestamps.Count >= maxSearchesPerMinute)
        {
            Debug.LogWarning("Search rate limit reached");
            return false;
        }
        
        searchTimestamps.Enqueue(DateTime.Now);
        return true;
    }
}
```

## Error Handling

### Handle Search Errors

```csharp
agent.onGoogleSearchError.AddListener(error =>
{
    Debug.LogError($"Search failed: {error}");
    
    if (error.Contains("quota"))
    {
        ShowMessage("Search quota exceeded. Try again later.");
    }
    else if (error.Contains("invalid_key"))
    {
        ShowMessage("Invalid Google API key. Check settings.");
    }
    else if (error.Contains("rate_limit"))
    {
        ShowMessage("Too many searches. Please wait.");
    }
    else
    {
        ShowMessage("Search failed. Please try again.");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class GoogleSearchManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField searchInput;
    [SerializeField] private Button searchButton;
    [SerializeField] private Transform resultsContainer;
    [SerializeField] private GameObject resultPrefab;
    
    private Dictionary<string, List<SearchResult>> cache = new();
    private Queue<DateTime> searchTimestamps = new();
    private const int MAX_SEARCHES_PER_MINUTE = 10;
    
    async void Start()
    {
        await SetupGoogleSearch();
        
        searchButton.onClick.AddListener(OnSearchClicked);
    }
    
    async UniTask SetupGoogleSearch()
    {
        // Configure Google Tools
        agent.Settings.GoogleTools = new GoogleToolsSettings
        {
            ApiKey = GetGoogleApiKey(),
            SearchEngineId = GetSearchEngineId()
        };
        
        // Add Google Search tool
        agent.AddGoogleTool(GoogleToolType.Search);
        
        // Listen for results
        agent.onGoogleSearchCompleted.AddListener(DisplayResults);
        agent.onGoogleSearchError.AddListener(OnSearchError);
        
        Debug.Log("✓ Google Search ready");
    }
    
    async void OnSearchClicked()
    {
        string query = searchInput.text.Trim();
        
        if (string.IsNullOrEmpty(query))
        {
            ShowMessage("Please enter a search query");
            return;
        }
        
        // Check rate limit
        if (!await CanSearch())
        {
            ShowMessage("Too many searches. Please wait.");
            return;
        }
        
        // Check cache
        if (cache.TryGetValue(query, out var cachedResults))
        {
            Debug.Log("Using cached results");
            DisplayResults(cachedResults);
            return;
        }
        
        // Perform search
        Debug.Log($"🔍 Searching: {query}");
        searchButton.interactable = false;
        
        await agent.SendAsync($"Search Google for: {query}");
    }
    
    async UniTask<bool> CanSearch()
    {
        // Clean old timestamps
        while (searchTimestamps.Count > 0 &&
               DateTime.Now - searchTimestamps.Peek() > TimeSpan.FromMinutes(1))
        {
            searchTimestamps.Dequeue();
        }
        
        // Check limit
        if (searchTimestamps.Count >= MAX_SEARCHES_PER_MINUTE)
        {
            return false;
        }
        
        searchTimestamps.Enqueue(DateTime.Now);
        return true;
    }
    
    void DisplayResults(List<SearchResult> results)
    {
        Debug.Log($"✓ Found {results.Count} results");
        
        // Clear previous
        foreach (Transform child in resultsContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Display new results
        foreach (var result in results)
        {
            CreateResultItem(result);
        }
        
        // Cache results
        string query = searchInput.text.Trim();
        cache[query] = results;
        
        searchButton.interactable = true;
    }
    
    void CreateResultItem(SearchResult result)
    {
        var resultObj = Instantiate(resultPrefab, resultsContainer);
        
        var titleText = resultObj.transform.Find("Title").GetComponent<TMP_Text>();
        var urlText = resultObj.transform.Find("URL").GetComponent<TMP_Text>();
        var snippetText = resultObj.transform.Find("Snippet").GetComponent<TMP_Text>();
        var button = resultObj.GetComponent<Button>();
        
        titleText.text = result.Title;
        urlText.text = result.Url;
        snippetText.text = result.Snippet;
        
        button.onClick.AddListener(() =>
        {
            Debug.Log($"Opening: {result.Url}");
            Application.OpenURL(result.Url);
        });
    }
    
    void OnSearchError(string error)
    {
        Debug.LogError($"Search error: {error}");
        ShowMessage($"Search failed: {error}");
        searchButton.interactable = true;
    }
    
    void ShowMessage(string message)
    {
        Debug.Log(message);
        // Show in UI
    }
    
    string GetGoogleApiKey()
    {
        return PlayerPrefs.GetString("GoogleApiKey", "");
    }
    
    string GetSearchEngineId()
    {
        return PlayerPrefs.GetString("GoogleSearchEngineId", "");
    }
}
```

## Next Steps

- [URL Context](url-context.md)
- [Code Execution](code-execution.md)
- [Web Search (Hosted)](../hosted-tools/web-search.md)
