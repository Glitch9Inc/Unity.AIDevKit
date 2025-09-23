# 🔎 Embeddings

Integrating OpenAI's Embeddings into your Unity project allows you to tap into advanced machine learning capabilities for understanding and processing natural language data. These embeddings transform text into high-dimensional vectors, enabling a wide range of applications like semantic search, text clustering, and more sophisticated natural language understanding tasks.

For a comprehensive understanding of the Embeddings API, including available models, parameter options, and usage scenarios, please refer to the [Embeddings API Reference](https://platform.openai.com/docs/api-reference/embeddings).

### Embeddings Operations Overview:

* **Semantic Text Analysis**: Convert text into vector space to understand its underlying meaning and context.
* **Text Similarity and Clustering**: Compare and cluster text data based on semantic similarity, ideal for organizing large datasets or creating recommendation systems.

### Sample Code for Embeddings Requests:

To use embeddings in your project, you'll need to create an `EmbeddingRequest` and execute it asynchronously. Here's how you can do it:

#### Semantic Analysis Request:

Generate embeddings for given text to analyze its semantics. You need to specify the text and choose an appropriate model.

{% tabs %}
{% tab title="C#" %}
```csharp
var request = new EmbeddingRequest.Builder()
    .SetModel(EmbeddingModel.TextEmbeddingAda002)
    .SetInput("The food was delicious and the waiter...")
    .SetEncodingFormat(EncodingFormat.Float)
    .Build();

var result = await request.ExecuteAsync();

float[] embeddings = result.EmbeddingVector;
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.embeddings.create(
  model="text-embedding-ada-002",
  input="The food was delicious and the waiter...",
  encoding_format="float"
)
```
{% endtab %}
{% endtabs %}

This request will return a high-dimensional vector representation of your text, which you can then use for further analysis or comparison against other vectors.

By integrating Embeddings into your Unity applications, you unlock powerful capabilities for advanced text analysis and interpretation, enhancing the intelligence and interactivity of your projects.
