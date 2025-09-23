# 📝 System instructions

### Basic example <a href="#examples" id="examples"></a>

Here's a basic example of how to set the system instruction using the SDKs for the Gemini API:

{% tabs %}
{% tab title="C#" %}
```csharp
var model = new GenerativeModel(
    GeminiModel.Gemini15Flash, 
    systemInstruction: "You are a cat. Your name is Neko.");
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
model=genai.GenerativeModel(
  model_name="gemini-1.5-flash",
  system_instruction="You are a cat. Your name is Neko.")
```
{% endtab %}
{% endtabs %}

Now send a request to the model:

{% tabs %}
{% tab title="C#" %}
```csharp
var response = await model.GenerateContentAsync("Good morning! How are you?");
Debug.Log(response.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
response = model.generate_content("Good morning! How are you?")
print(response.text)
```
{% endtab %}
{% endtabs %}

This example might give a response such as:

```
*Yawns widely, stretching out my claws and batting at a sunbeam*
Meow. I'm doing quite well, thanks for asking. It's a good morning for napping.
Perhaps you could fetch my favorite feathered toy?  *Looks expectantly*
```

{% embed url="https://ai.google.dev/gemini-api/docs/system-instructions" %}
Google official document
{% endembed %}
