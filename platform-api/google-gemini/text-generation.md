# 💬 Text generation

### Generate text from text-only input <a href="#generate-text-from-text" id="generate-text-from-text"></a>

The simplest way to generate text using the Gemini API is to provide the model with a single text-only input, as shown in this example:

{% tabs %}
{% tab title="C#" %}
```csharp
using Glitch9.AIDevKit.Google.GenerativeAI;

// Choose a model that's appropriate for your use case.
var model = new GenerativeModel(GeminiModel.Gemini15Flash);

var prompt = "Write a story about a magic backpack.";

var response = await model.GenerateContentAsync(prompt);

Debug.Log(response.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
import os
import google.generativeai as genai

# Access your API key as an environment variable.
genai.configure(api_key=os.environ['API_KEY'])
# Choose a model that's appropriate for your use case.
model = genai.GenerativeModel('gemini-1.5-flash')

prompt = "Write a story about a magic backpack."

response = model.generate_content(prompt)

print(response.text)
```
{% endtab %}
{% endtabs %}

In this case, the prompt ("Write a story about a magic backpack") doesn't include any output examples, system instructions, or formatting information. It's a [zero-shot](https://ai.google.dev/gemini-api/docs/models/generative-models#zero-shot-prompts) approach. For some use cases, a [one-shot](https://ai.google.dev/gemini-api/docs/models/generative-models#one-shot-prompts) or [few-shot](https://ai.google.dev/gemini-api/docs/models/generative-models#few-shot-prompts) prompt might produce output that's more aligned with user expectations. In some cases, you might also want to provide [system instructions](https://ai.google.dev/gemini-api/docs/system-instructions) to help the model understand the task or follow specific guidelines.

### Generate text from text-and-image input <a href="#generate-text-from-text-and-image" id="generate-text-from-text-and-image"></a>

The Gemini API supports multimodal inputs that combine text with media files. The following example shows how to generate text from text-and-image input:

{% tabs %}
{% tab title="C#" %}
```csharp
using Glitch9.AIDevKit.Google.GenerativeAI;

// Choose a model that's appropriate for your use case.
var model = new GenerativeModel(GeminiModel.Gemini15Flash);

var image1 = new ImageResource("Assets/image1.jpg");
var image2 = new ImageResource("Assets/image2.jpg");

var prompt = "What's different between these pictures?";

var response = await model.GenerateContentAsync(
    prompt, 
    images: new List<ImageResource> { image1, image2 });
    
Debug.Log(response.GetOutputText());
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
message = "hello world"
print(message)
```
{% endtab %}
{% endtabs %}

As with text-only prompting, multimodal prompting can involve various approaches and refinements. Depending on the output from this example, you might want to add steps to the prompt or be more specific in your instructions. To learn more, see [File prompting strategies](https://ai.google.dev/gemini-api/docs/file-prompting-strategies).

### Generate a text stream <a href="#generate-a-text-stream" id="generate-a-text-stream"></a>

By default, the model returns a response after completing the entire text generation process. You can achieve faster interactions by not waiting for the entire result, and instead use streaming to handle partial results.

The following example shows how to implement streaming using the [`streamGenerateContent`](https://ai.google.dev/api/rest/v1/models/streamGenerateContent) method to generate text from a text-only input prompt.

{% tabs %}
{% tab title="C#" %}
```csharp
using Glitch9.AIDevKit.Google.GenerativeAI;

// Choose a model that's appropriate for your use case.
var model = new GenerativeModel(GeminiModel.Gemini15Flash);

var prompt = "Write a story about a magic backpack.";

var streamHandler = new StreamHandler();
streamHandler.OnStream += OnStream;

var response = await model.GenerateContentAsync(
    prompt, 
    streamHandler: streamHandler);
    
return;

void OnStream(object sender, string chunk)
{
    Debug.Log(chunk);
}
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
import os
import google.generativeai as genai

# Access your API key as an environment variable.
genai.configure(api_key=os.environ['API_KEY'])
# Choose a model that's appropriate for your use case.
model = genai.GenerativeModel('gemini-1.5-flash')

prompt = "Write a story about a magic backpack."

response = model.generate_content(prompt, stream=True)

for chunk in response:
  print(chunk.text)
  print("_"*80)
```
{% endtab %}
{% endtabs %}

### What's next <a href="#whats-next" id="whats-next"></a>

This guide shows how to use [`generateContent`](https://ai.google.dev/api/rest/v1/models/generateContent) and [`streamGenerateContent`](https://ai.google.dev/api/rest/v1/models/streamGenerateContent) to generate text outputs from text-only and text-and-image inputs. To learn more about generating text using the Gemini API, see the following resources:

* [Prompting with media files](https://ai.google.dev/gemini-api/docs/prompting_with_media): The Gemini API supports prompting with text, image, audio, and video data, also known as multimodal prompting.
* [System instructions](https://ai.google.dev/gemini-api/docs/system-instructions): System instructions let you steer the behavior of the model based on your specific needs and use cases.
* [Safety guidance](https://ai.google.dev/gemini-api/docs/safety-guidance): Sometimes generative AI models produce unexpected outputs, such as outputs that are inaccurate, biased, or offensive. Post-processing and human evaluation are essential to limit the risk of harm from such outputs.

{% embed url="https://ai.google.dev/gemini-api/docs/text-generation" %}
Google official document
{% endembed %}
