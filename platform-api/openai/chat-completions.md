# 💬 Chat completions

To incorporate OpenAI's conversational AI capabilities into your Unity project, you'll be utilizing a variety of requests centered around chat interactions through the **OpenAI with unity asset**. These requests enable the integration of OpenAI's sophisticated language models, like GPT-4, into your applications, facilitating dynamic and intelligent conversations. Whether you're aiming to develop engaging NPCs, automated customer service bots, or interactive storytelling experiences, the chat functionalities offered by OpenAI can significantly enhance the interactivity of your Unity projects.

For a deep dive into the technical details and options available for these requests, refer to the [Chat API Reference](https://platform.openai.com/docs/api-reference/chat).

### Chat Operations Overview:

1. **Chat Completion**: Generate conversational responses based on input prompts. This is the core of creating dynamic dialogues where the AI provides contextually relevant answers or reactions to user inputs.
2. **Chat Streaming**: For applications requiring real-time interaction, streaming chat completions allow your application to receive responses as they are generated. This approach is ideal for simulating live conversations, enhancing the responsiveness and dynamism of chatbots or virtual assistants.

### Sample Code for Chat Completions:

#### 1. Standard Chat Completion Request

This approach sends a prompt to OpenAI and receives a single, complete response. It's straightforward and suitable for most use cases where an immediate and full response is expected.

{% tabs %}
{% tab title="C#" %}
```csharp
var request = new ChatCompletionRequest.Builder()
    .SetModel(GPTModel.GPT4_0125Preview) 
    .SetPrompt("Your prompt here")
    .SetMaxTokens(50) // Optional: Set the maximum number of tokens to generate.
    .Build();

ChatCompletion completion = await request.ExecuteAsync();

Debug.Log(completion.Choices[0].Message);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
  model="gpt-4o",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ]
)

print(completion.choices[0].message)
```
{% endtab %}
{% endtabs %}

#### 2. Streaming Chat Completion Request

For receiving responses as they're generated, use the streaming request. This method is beneficial when you want to process or display parts of the response as soon as they become available, especially for interactive applications.

{% tabs %}
{% tab title="C#" %}
```csharp
var request = new ChatCompletionRequest.Builder()
    .SetModel(GPTModel.GPT4_0125Preview) 
    .SetPrompt("Your prompt here")
    .SetMaxTokens(50) // Optional: Set the maximum number of tokens to generate.
    .SetStream(true) // Enable streaming.
    .Build();

await request.StreamAsync(streamedResponse =>
{
    Debug.Log($"Streamed chunk: {streamedResponse}");
    // Handle each piece of the response as it arrives.
});
```
{% endtab %}
{% endtabs %}

Or you can create a stream handler to handle the streamed response.

{% tabs %}
{% tab title="C#" %}
```csharp
var streamHandler = new StreamHandler();

streamHandler.OnStart += sender =>
{
    Debug.Log("Stream started");
};

streamHandler.OnStream += (sender, message) =>
{
    Debug.Log($"Streamed chunk: {message}");
    // Handle each piece of the response as it arrives.
};

streamHandler.OnDone += sender =>
{
    Debug.Log("Stream done");
};

await request.StreamAsync(streamHandler);
```
{% endtab %}
{% endtabs %}
