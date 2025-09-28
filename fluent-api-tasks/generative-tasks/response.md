---
description: Generate a response using a Large Language Model (LLM)
icon: message-pen
---

# Text

returns[`ChatCompletion`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.ChatCompletion.html)

Response generation is one of the core uses of generative AI. In the GENTask system, the [**`GENResponse`**](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.GENResponseTask.html) is used to generate text (for example, completing a prompt or answering a question). This task sends a prompt to a Large Language Model (LLM) and returns a response.

| Input Modalities                       | Output                                                                                     |
| -------------------------------------- | ------------------------------------------------------------------------------------------ |
| <p>Text<br>Image<br>Audio<br>Video</p> | <p>Text<br>Text (Streaming)<br>Tool Calls<br>Speech<br>Image (Rarely)<br>File (Rarely)</p> |

**Basic Usage**

```csharp
string prompt = "Explain quantum computing in simple terms.";

string result = await prompt
    .GENCompletion()
    .SetModel(OpenAIIModel.GPT4o)
    .ExecuteAsync();

Debug.Log(result);
```

**Streaming**

```csharp
// Streaming
string prompt = "Explain quantum computing in simple terms.";

await prompt
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .OnReceiveText(delta => Debug.Log(delta))
    .OnReceiveDone(completion => Debug.Log("Finished."))
    .StreamAsync();
```

**Vision Request (Image Recognition)**

```csharp
// Vision Request
Texture2D myTexture = myImage.sprite.texture;
string prompt = "Tell me about this image.";

string result = await prompt
    .GENCompletion()
    .Attach(myTexture)
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();

Debug.Log(result);
```

***

## Structured Outputs

returns [`StructuredOutput<T>`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.StructuredOutput-1.html)

[**`GENStruct<T>`**](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.GENStructTask-1.html) lets you turn natural language prompts into **strongly-typed Unity C# objects** by guiding the AI to return JSON that matches your class definition.

It's perfect for generating game data like:

* 📦 Items
* 🎮 Quests
* 🗨️ NPC Dialogue Trees
* 📋 Skill Definitions
* 🧠 AI Configuration Profiles

| Input Modalities                       | Output                         |
| -------------------------------------- | ------------------------------ |
| <p>Text<br>Image<br>Audio<br>Video</p> | Structured Outputs (JSON Mode) |

***

#### Required Annotations

These attributes are mandatory. Without them, the task will fail to register the schema.

| Provider                       | Required Attributes                                                                                                               |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **OpenAI, Ollama, OpenRouter** | <p><code>[StrictJsonSchema(...)]</code> <em>(on the class)</em><br><code>[JsonSchema(...)]</code> <em>(on each property)</em></p> |
| **Gemini**                     | <p><code>[JsonSchema(...)]</code> <em>(on the class)</em><br><code>[JsonSchema(...)]</code> <em>(on each property)</em></p>       |

**Generating an RPG Item**

```csharp
using Glitch9.AIDevKit.OpenAI;
using Glitch9.IO.Json.Schema;
using System.Collections.Generic;

[StrictJsonSchema("item_response", Description = "RPG-style inventory item", Strict = true)]
public class GameItem
{
    [JsonSchemaProperty("id", Description = "Unique item ID", Required = true)]
    public string Id { get; set; }

    [JsonSchemaProperty("name", Description = "Name of the item", Required = true)]
    public string Name { get; set; }

    [JsonSchemaProperty("rarity", Description = "Item rarity", Enum = new[] { "Common", "Uncommon", "Rare", "Epic", "Legendary" })]
    public string Rarity { get; set; }

    [JsonSchemaProperty("effects", Description = "List of effects applied when used")]
    public List<string> Effects { get; set; }
}


GameItem item = await "Generate a legendary sword with magical effects"
    .GENStruct<GameItem>()
    .SetModel(OpenAIModel.GPT4o)
    .SetTemperature(0.4f)
    .ExecuteAsync();

Debug.Log(item.Name + " - " + item.Rarity); // "Blade of Eternity - Legendary"
```

**Generating a Simple Quest**

```csharp
[StrictJsonSchema("quest_response", Description = "Basic RPG quest", Strict = true)]
public class Quest
{
    [JsonSchemaProperty("title", Description = "Quest name", Required = true)]
    public string Title { get; set; }

    [JsonSchemaProperty("description", Description = "What the player must do", Required = true)]
    public string Description { get; set; }

    [JsonSchemaProperty("objective", Description = "Main goal of the quest")]
    public string Objective { get; set; }

    [JsonSchemaProperty("rewardXP", Description = "XP rewarded upon completion")]
    public int RewardXP { get; set; }
}

Quest quest = await "Create a side quest for a fantasy game where a farmer asks you to find his lost chicken"
    .GENStruct<Quest>()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```
