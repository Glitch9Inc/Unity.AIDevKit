---
description: Generate vector embeddings from text input using GENEmbed.
icon: brackets-square
---

# Embedding

returns [`Embedding`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.Embedding.html)

**GENEmbed** is used to generate vector embeddings from text input. Embeddings are numerical representations of text that capture semantic meaning, allowing for tasks such as similarity search, clustering, and classification.

**Basic Usage**

```csharp
float[] embeddingVector = await "my text input to generate embedding"
    .GENEmbed() 
    .ExecuteAsync();
    
// Use the embedding vector for further processing
```
