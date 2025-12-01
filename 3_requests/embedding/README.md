---
icon: diagram-project
---

# Embedding

Generate vector embeddings for semantic search and similarity using `.GENEmbed()`.

## What are Embeddings?

Embeddings are numerical representations of text that capture semantic meaning, allowing you to:

- Find similar content
- Implement semantic search
- Cluster related items
- Measure text similarity

## Basic Usage

### Single Text

```csharp
float[] embedding = await "Hello, world!"
    .GENEmbed()
    .ExecuteAsync();

Debug.Log($"Embedding dimensions: {embedding.Length}");
```

### Multiple Texts

```csharp
string[] texts = new[] { "Text 1", "Text 2", "Text 3" };
float[][] embeddings = await texts
    .GENEmbed()
    .ExecuteAsync();
```

## Configuration

```csharp
float[] embedding = await "Search query"
    .GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Semantic Search

```csharp
public class SemanticSearch : MonoBehaviour
{
    private Dictionary<string, float[]> database = new();
    
    async void Start()
    {
        // Build embedding database
        await BuildDatabase(new[]
        {
            "How to jump in Unity",
            "Creating a character controller",
            "Audio system tutorial"
        });
    }
    
    async UniTask BuildDatabase(string[] documents)
    {
        foreach (var doc in documents)
        {
            float[] embedding = await doc.GENEmbed().ExecuteAsync();
            database[doc] = embedding;
        }
    }
    
    public async UniTask<string> Search(string query)
    {
        float[] queryEmbed = await query.GENEmbed().ExecuteAsync();
        
        string bestMatch = null;
        float bestScore = float.MinValue;
        
        foreach (var (doc, embed) in database)
        {
            float similarity = CosineSimilarity(queryEmbed, embed);
            if (similarity > bestScore)
            {
                bestScore = similarity;
                bestMatch = doc;
            }
        }
        
        return bestMatch;
    }
    
    float CosineSimilarity(float[] a, float[] b)
    {
        float dot = 0, magA = 0, magB = 0;
        for (int i = 0; i < a.Length; i++)
        {
            dot += a[i] * b[i];
            magA += a[i] * a[i];
            magB += b[i] * b[i];
        }
        return dot / (Mathf.Sqrt(magA) * Mathf.Sqrt(magB));
    }
}
```

### Example 2: Content Recommendation

```csharp
public class ContentRecommender : MonoBehaviour
{
    private Dictionary<string, float[]> contentEmbeddings = new();
    
    public async UniTask<List<string>> GetRecommendations(string userInterest)
    {
        float[] interestEmbed = await userInterest.GENEmbed().ExecuteAsync();
        
        var scores = new List<(string content, float score)>();
        
        foreach (var (content, embed) in contentEmbeddings)
        {
            float similarity = CosineSimilarity(interestEmbed, embed);
            scores.Add((content, similarity));
        }
        
        return scores
            .OrderByDescending(x => x.score)
            .Take(5)
            .Select(x => x.content)
            .ToList();
    }
}
```

## Common Models

```csharp
// OpenAI
await text.GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)  // 1536 dimensions
    .ExecuteAsync();

await text.GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Large)  // 3072 dimensions
    .ExecuteAsync();

// Google
await text.GENEmbed()
    .SetModel(GoogleModel.TextEmbedding004)
    .ExecuteAsync();
```

## Use Cases

- **Semantic Search**: Find relevant content
- **Recommendations**: Suggest similar items
- **Clustering**: Group related content
- **Duplicate Detection**: Find similar texts
- **Question Answering**: Match questions to answers

## Next Steps

- [Text Embedding](text-embedding.md) - Single text embeddings
- [Batch Embedding](batch-embedding.md) - Multiple texts
