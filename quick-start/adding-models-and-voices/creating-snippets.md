---
description: Create code snippets with one click.
icon: square-code
---

# Creating Snippets

Once you've added your models or voices to the library,\
you can create reusable code snippets using the **"Create Snippets"** button located at the bottom of the **Model Library** or **Voice Library** window.

{% hint style="info" %}
**What is a Snippet?**

A **snippet** is a small block of pre-written code that makes it easy to call a specific model or voice without typing the configuration manually each time.\
It’s especially useful for automating prompts, speech generation, or completions in your game or editor tools.

> Think of snippets as **ready-to-use shortcuts** for your favorite models and voices.
{% endhint %}

***

#### How to Create a Snippet

<figure><img src="../../.gitbook/assets/image (106).png" alt="" width="563"><figcaption></figcaption></figure>

1. **Add** at least one model or voice to your library (if you haven’t already)
2. Click the **“Create Snippets”** button
3. Snippets will be automatically generated and saved as C# files in your project (usually under `Assets/AIDevKit/Snippets/`)

***

#### Using Snippets

Once created, you can call your saved model or voice like this:

```csharp
"My prompt here"
    .GENResponse()
    .SetModel(OpenAIModels.GPT4o)
    .ExecuteAsync();

"Hello!"
    .GENSpeech()
    .SetVoice(ElevenLabsVoices.Rachel)
    .ExecuteAsync();
```

These snippets are especially useful for:

* Auto-complete
* Reducing errors
* Keeping your AI calls consistent across your project

