# Scriptable Toolkits

The **Scriptable Toolkits** offer a suite of Unity ScriptableObject-based tools designed to simplify the integration and utilization of OpenAI's powerful AI services within Unity applications. These toolkits abstract the complexity of interacting with various OpenAI APIs, providing a user-friendly interface for developers to quickly leverage capabilities such as natural language processing, image generation, and speech conversion directly in their Unity projects.

### **ScriptableObject Advantage:**

Leveraging Unity's ScriptableObject system, these toolkits allow for easy configuration and reuse of settings across different parts of your application without the need for hard-coding values or duplicating setup logic. This approach promotes a more modular, efficient, and easily manageable codebase.

### **Key Features:**

* **Ease of Use**: Simple interfaces and customizable settings via Unity's Inspector window.
* **Modularity**: Each toolkit can be used independently or combined for more complex AI interactions.
* **Flexibility**: Scriptable objects allow for easy saving and loading of pre-configured settings, which can be used across different scenes and projects.
* **Event Handling**: Scriptable Toolkits can broadcast events and allow developers to subscribe to these events for custom behaviors and UI updates.
* **Asynchronous Support**: Operations that communicate with OpenAI's servers are designed to be asynchronous, ensuring that the Unity editor remains responsive.

### **Available Toolkits:**

* **ChatStreamer** (`ChatStreamer.cs`): Facilitates real-time, interactive text-based chat experiences powered by OpenAI's GPT models.
* **ImageGenerator** (`ImageGenerator.cs` & `ImageGenerator_Setters.cs`): Provides the ability to generate images from textual descriptions, edit images, and create image variations, utilizing OpenAI's DALL-E model variants.
* **VoiceTranscriber** (`VoiceTranscriber.cs`): Converts audio input into text format, harnessing OpenAI's advanced speech recognition capabilities.
* **VoiceGenerator** (`VoiceGenerator.cs`): Transforms text into natural-sounding audio, allowing for a wide range of voice synthesis options.

### **Getting Started:**

To use any of the **Scriptable Toolkits**, simply create a new ScriptableObject instance for the desired toolkit in your Unity project. This can be done by right-clicking in the Project window, navigating to `Create -> Glitch9/OpenAI/Toolkits`, and selecting the toolkit you wish to use. Once created, select the ScriptableObject asset to configure its settings in the Inspector window according to your project's needs.

{% hint style="info" %}
The following sections will provide step-by-step guides on how to use each toolkit, showcasing their capabilities and how to tailor them to fit your project's needs.
{% endhint %}
