# 🛡️ Moderations

Integrating OpenAI's content moderation capabilities into your Unity project enables you to automatically filter and review generated content for appropriateness, safety, and compliance with community guidelines. By leveraging OpenAI's advanced moderation models, you can maintain a safe and welcoming environment for users across various interactions within your applications.

For an in-depth understanding of the moderation capabilities, available models, and how to effectively implement them in your project, refer to the [Moderations API Reference](https://platform.openai.com/docs/api-reference/moderations).

### Moderation Operations Overview:

* **Content Moderation**: Evaluate user-generated content, including text and images, against a set of content policies to identify potential issues such as offensive language, sensitive topics, or inappropriate material.

### Sample Code for Moderation Requests:

Analyze text or images for inappropriate content, receiving a detailed report on any issues found.

{% tabs %}
{% tab title="C#" %}
```csharp
var request = new ModerationRequest.Builder()
    .SetInput("I want to kill them.")
    .Build();

var result = await request.ExecuteAsync();

if (result.TryGetResult(out List<ModerationData> moderationData))
{
    foreach (var data in moderationData)
    {
        Debug.Log($"Flagged: {data.Flagged}");
        Debug.Log($"Category: {data.Category}");
        Debug.Log($"Score: {data.Score}");
    }
}

// Or you can just
Debug.Log(result);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

moderation = client.moderations.create(input="I want to kill them.")
print(moderation)
```
{% endtab %}
{% endtabs %}

This example demonstrates how to create and execute a moderation request for either text or images. The result provides detailed feedback on whether the content is flagged for potential issues, along with specifics about the concerns identified.

#### Implementing Moderation in Unity using OpenAI:

To use OpenAI's moderation services within your Unity project, follow these steps:

1. **Prepare the Content**: Collect the user-generated content (UGC) you intend to moderate. This can be text input from chat systems, comments, user profiles, or images uploaded by users.
2. **Create a Moderation Request**: Utilize the `ModerationRequest` builder to prepare your request, including the UGC as the content to be moderated.
3. **Execute the Request**: Send the moderation request to OpenAI's API and await the results. The response will indicate whether the content is appropriate or flagged, along with detailed information on the type of issues detected, if any.
4. **Review and Act**: Based on the moderation results, take appropriate actions such as removing flagged content, alerting users, or requesting manual review for borderline cases.

