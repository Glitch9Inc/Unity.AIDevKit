---
icon: vector-square
---

# Text Embedding

Generate vector embeddings for single text inputs using `.GENEmbed()`.

## Basic Usage

```csharp
float[] embedding = await "Hello, world!"
    .GENEmbed()
    .ExecuteAsync();

Debug.Log($"Embedding dimensions: {embedding.Length}");
```

## Configuration

### Model Selection

```csharp
// OpenAI - Small (1536 dimensions, faster)
float[] embedding = await "Search query"
    .GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)
    .ExecuteAsync();

// OpenAI - Large (3072 dimensions, more accurate)
float[] embedding = await "Search query"
    .GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Large)
    .ExecuteAsync();

// Google
float[] embedding = await "Search query"
    .GENEmbed()
    .SetModel(GoogleModel.TextEmbedding004)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Document Search Engine

```csharp
public class DocumentSearchEngine : MonoBehaviour
{
    [System.Serializable]
    public class Document
    {
        public string id;
        public string content;
        public float[] embedding;
    }
    
    private List<Document> documents = new();
    
    public async UniTask IndexDocument(string id, string content)
    {
        float[] embedding = await content
            .GENEmbed()
            .SetModel(OpenAIModel.TextEmbedding3Small)
            .ExecuteAsync();
        
        documents.Add(new Document
        {
            id = id,
            content = content,
            embedding = embedding
        });
    }
    
    public async UniTask<List<Document>> Search(string query, int topK = 5)
    {
        float[] queryEmbedding = await query
            .GENEmbed()
            .SetModel(OpenAIModel.TextEmbedding3Small)
            .ExecuteAsync();
        
        var scores = documents
            .Select(doc => new
            {
                doc,
                score = CosineSimilarity(queryEmbedding, doc.embedding)
            })
            .OrderByDescending(x => x.score)
            .Take(topK)
            .Select(x => x.doc)
            .ToList();
        
        return scores;
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

### Example 2: FAQ Matcher

```csharp
public class FAQMatcher : MonoBehaviour
{
    [System.Serializable]
    public class FAQ
    {
        public string question;
        public string answer;
        public float[] embedding;
    }
    
    private List<FAQ> faqs = new();
    
    async void Start()
    {
        await LoadFAQs();
    }
    
    async UniTask LoadFAQs()
    {
        var faqData = new[]
        {
            ("How do I jump?", "Press the Space key to jump"),
            ("How do I save?", "Press Esc and click Save Game"),
            ("How do I pause?", "Press Esc to pause the game")
        };
        
        foreach (var (question, answer) in faqData)
        {
            float[] embedding = await question
                .GENEmbed()
                .ExecuteAsync();
            
            faqs.Add(new FAQ
            {
                question = question,
                answer = answer,
                embedding = embedding
            });
        }
    }
    
    public async UniTask<string> GetAnswer(string userQuestion)
    {
        float[] queryEmbed = await userQuestion
            .GENEmbed()
            .ExecuteAsync();
        
        FAQ bestMatch = null;
        float bestScore = float.MinValue;
        
        foreach (var faq in faqs)
        {
            float similarity = CosineSimilarity(queryEmbed, faq.embedding);
            if (similarity > bestScore)
            {
                bestScore = similarity;
                bestMatch = faq;
            }
        }
        
        return bestMatch?.answer ?? "Sorry, I don't understand.";
    }
}
```

### Example 3: Content Deduplication

```csharp
public class ContentDeduplicator : MonoBehaviour
{
    private const float SIMILARITY_THRESHOLD = 0.95f;
    
    public async UniTask<bool> IsDuplicate(string newContent, List<string> existingContent)
    {
        float[] newEmbed = await newContent.GENEmbed().ExecuteAsync();
        
        foreach (var existing in existingContent)
        {
            float[] existEmbed = await existing.GENEmbed().ExecuteAsync();
            float similarity = CosineSimilarity(newEmbed, existEmbed);
            
            if (similarity >= SIMILARITY_THRESHOLD)
                return true;
        }
        
        return false;
    }
}
```

### Example 4: Smart Categorization

```csharp
public class SmartCategorizer : MonoBehaviour
{
    private Dictionary<string, float[]> categories = new();
    
    async void Start()
    {
        // Define categories
        categories["Combat"] = await "Fighting, weapons, battles".GENEmbed().ExecuteAsync();
        categories["Exploration"] = await "Discovery, travel, adventure".GENEmbed().ExecuteAsync();
        categories["Puzzle"] = await "Logic, problem solving, thinking".GENEmbed().ExecuteAsync();
    }
    
    public async UniTask<string> Categorize(string content)
    {
        float[] contentEmbed = await content.GENEmbed().ExecuteAsync();
        
        string bestCategory = null;
        float bestScore = float.MinValue;
        
        foreach (var (category, embed) in categories)
        {
            float similarity = CosineSimilarity(contentEmbed, embed);
            if (similarity > bestScore)
            {
                bestScore = similarity;
                bestCategory = category;
            }
        }
        
        return bestCategory;
    }
}
```

## Provider Support

### OpenAI

```csharp
// Small model (1536 dimensions)
float[] embedding = await text
    .GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)
    .ExecuteAsync();

// Large model (3072 dimensions)
float[] embedding = await text
    .GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Large)
    .ExecuteAsync();
```

### Google

```csharp
float[] embedding = await text
    .GENEmbed()
    .SetModel(GoogleModel.TextEmbedding004)
    .ExecuteAsync();
```

## Similarity Calculation

### Cosine Similarity

Most common method for comparing embeddings:

```csharp
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
```

Returns value between -1 and 1:

- **1.0**: Identical
- **0.5+**: Similar
- **0.0**: Unrelated
- **-1.0**: Opposite

### Euclidean Distance

Alternative method:

```csharp
float EuclideanDistance(float[] a, float[] b)
{
    float sum = 0;
    for (int i = 0; i < a.Length; i++)
    {
        float diff = a[i] - b[i];
        sum += diff * diff;
    }
    return Mathf.Sqrt(sum);
}
```

Lower distance = more similar.

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache embeddings
Dictionary<string, float[]> cache = new();

// ✅ Use appropriate model
// Small for speed, Large for accuracy
await text.GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)
    .ExecuteAsync();

// ✅ Normalize text before embedding
string normalized = text.ToLower().Trim();
float[] embed = await normalized.GENEmbed().ExecuteAsync();

// ✅ Batch when possible (see Batch Embedding)
```

### ❌ Bad Practices

```csharp
// ❌ Don't embed in Update()
void Update()
{
    await text.GENEmbed().ExecuteAsync();  // NO!
}

// ❌ Don't generate unnecessarily
// Cache and reuse embeddings

// ❌ Don't compare embeddings from different models
// Always use same model for comparison
```

## Use Cases

| Use Case | Description |
|----------|-------------|
| **Semantic Search** | Find relevant documents |
| **FAQ Matching** | Match questions to answers |
| **Deduplication** | Detect similar content |
| **Categorization** | Classify content |
| **Recommendations** | Suggest similar items |
| **Clustering** | Group related items |

## Performance Tips

```csharp
// ✅ Good - cache embeddings
Dictionary<string, float[]> embedCache = new();

async UniTask<float[]> GetCachedEmbedding(string text)
{
    if (!embedCache.ContainsKey(text))
    {
        embedCache[text] = await text.GENEmbed().ExecuteAsync();
    }
    return embedCache[text];
}

// ✅ Good - use smaller model when appropriate
await text.GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)  // Faster
    .ExecuteAsync();
```

## Next Steps

- [Batch Embedding](batch-embedding.md) - Process multiple texts
