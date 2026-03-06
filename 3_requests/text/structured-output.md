---
icon: brackets-curly
---

# Structured Output

The `.GENStruct<T>()` method generates JSON responses that are automatically mapped to strongly-typed C# classes.

## Basic Usage

```csharp
[StrictJsonSchema("character", "A game character.")]
public class Character
{
    [JsonSchemaProperty("name", "Character name.", Required = true)]
    public string Name { get; set; }

    [JsonSchemaProperty("level", "Character level.", Required = true)]
    public int Level { get; set; }

    [JsonSchemaProperty("class", "Character class.", Required = true)]
    public string Class { get; set; }
}

Character hero = await "Generate a level 10 warrior"
    .GENStruct<Character>()
    .ExecuteAsync();

Debug.Log($"{hero.Name} - Lv.{hero.Level} {hero.Class}");
```

> **Note:** `[StrictJsonSchema]` is required on the class. When `Strict = true`, **all** properties must have `Required = true` in their `[JsonSchemaProperty]`.

## Defining Schema Classes

### 1. Simple Class

```csharp
[StrictJsonSchema("user_profile", "A user profile.")]
public class UserProfile
{
    [JsonSchemaProperty("name", "Full name of the user.")]
    public string Name { get; set; }

    [JsonSchemaProperty("age", "Age of the user.")]
    public int Age { get; set; }

    [JsonSchemaProperty("email", "Email address.")]
    public string Email { get; set; }
}

UserProfile user = await "Create profile for John, age 30"
    .GENStruct<UserProfile>()
    .ExecuteAsync();
```

### 2. Nested Objects

```csharp
[StrictJsonSchema("game_character", "A game character with stats and skills.")]
public class GameCharacter
{
    [JsonSchemaProperty("name", "Character name.")]
    public string Name { get; set; }

    [JsonSchemaProperty("stats", "Character stats.")]
    public Stats Stats { get; set; }

    [JsonSchemaProperty("skills", "List of skill names.")]
    public List<string> Skills { get; set; }
}

[StrictJsonSchema("stats", "Character stats.")]
public class Stats
{
    [JsonSchemaProperty("health", "Max health points.")]
    public int Health { get; set; }

    [JsonSchemaProperty("mana", "Max mana points.")]
    public int Mana { get; set; }

    [JsonSchemaProperty("strength", "Strength attribute.")]
    public int Strength { get; set; }
}

GameCharacter character = await "Create a mage character"
    .GENStruct<GameCharacter>()
    .ExecuteAsync();
```

### 3. Collections

```csharp
[StrictJsonSchema("item_list", "A list of items.")]
public class ItemList
{
    [JsonSchemaProperty("items", "The items in the list.")]
    public List<Item> Items { get; set; }
}

[StrictJsonSchema("item", "A single item.")]
public class Item
{
    public string Name { get; set; }
    public int Quantity { get; set; }
    public float Price { get; set; }
}

ItemList inventory = await "Generate 5 fantasy items"
    .GENStruct<ItemList>()
    .ExecuteAsync();

foreach (var item in inventory.Items)
{
    Debug.Log($"{item.Name} x{item.Quantity} - ${item.Price}");
}
```

### 4. Enums

```csharp
public enum Rarity { Common, Uncommon, Rare, Epic, Legendary }

[StrictJsonSchema("weapon", "A weapon item.")]
public class Weapon
{
    [JsonSchemaProperty("name", "Weapon name.")]
    public string Name { get; set; }

    [JsonSchemaProperty("damage", "Base damage value.")]
    public int Damage { get; set; }

    [JsonSchemaProperty("rarity", "Rarity tier.")]
    public Rarity Rarity { get; set; }
}

Weapon sword = await "Generate a legendary sword"
    .GENStruct<Weapon>()
    .ExecuteAsync();
```

## Input Types

### String Input

```csharp
Character character = await "Create a warrior character"
    .GENStruct<Character>()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("Generate a {characterClass} named {name}");
Character character = await prompt
    .GENStruct<Character>()
    .ExecuteAsync();
```

## Configuration

### Model Selection

```csharp
Character character = await "Create character"
    .GENStruct<Character>()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

### Temperature

```csharp
// More creative/varied output
Character creative = await "Create character"
    .GENStruct<Character>()
    .SetTemperature(1.0f)
    .ExecuteAsync();

// More consistent output
Character consistent = await "Create character"
    .GENStruct<Character>()
    .SetTemperature(0.0f)
    .ExecuteAsync();
```

## Common Use Cases

### 1. Data Extraction

```csharp
[StrictJsonSchema("contact_info", "Contact information.")]
public class ContactInfo
{
    [JsonSchemaProperty("name", "Full name.")]
    public string Name { get; set; }

    [JsonSchemaProperty("email", "Email address.")]
    public string Email { get; set; }

    [JsonSchemaProperty("phone", "Phone number.")]
    public string Phone { get; set; }
}

string text = "Contact John Doe at john@example.com or 555-1234";
ContactInfo contact = await $"Extract contact info from: {text}"
    .GENStruct<ContactInfo>()
    .ExecuteAsync();
```

### 2. Form Generation

```csharp
[StrictJsonSchema("quest_data", "Quest configuration data.")]
public class QuestData
{
    [JsonSchemaProperty("title", "Quest title.")]
    public string Title { get; set; }

    [JsonSchemaProperty("description", "Quest description.")]
    public string Description { get; set; }

    [JsonSchemaProperty("reward_gold", "Gold reward amount.")]
    public int RewardGold { get; set; }

    [JsonSchemaProperty("required_level", "Minimum player level.")]
    public int RequiredLevel { get; set; }
}

QuestData quest = await "Generate a beginner quest"
    .GENStruct<QuestData>()
    .ExecuteAsync();
```

### 3. Configuration Files

```csharp
[StrictJsonSchema("game_settings", "Default game settings.")]
public class GameSettings
{
    [JsonSchemaProperty("music_volume", "Music volume (0-1).")]
    public float MusicVolume { get; set; }

    [JsonSchemaProperty("sfx_volume", "SFX volume (0-1).")]
    public float SfxVolume { get; set; }

    [JsonSchemaProperty("difficulty", "Difficulty level.")]
    public string Difficulty { get; set; }

    [JsonSchemaProperty("enable_tutorial", "Whether to show tutorial.")]
    public bool EnableTutorial { get; set; }
}

GameSettings settings = await "Generate default game settings"
    .GENStruct<GameSettings>()
    .ExecuteAsync();
```

### 4. Content Generation

```csharp
[StrictJsonSchema("story_segment", "A story chapter.")]
public class StorySegment
{
    [JsonSchemaProperty("chapter", "Chapter title.")]
    public string Chapter { get; set; }

    [JsonSchemaProperty("content", "Chapter content.")]
    public string Content { get; set; }

    [JsonSchemaProperty("choices", "Player choices.")]
    public List<string> Choices { get; set; }
}

StorySegment story = await "Generate a story chapter with 3 choices"
    .GENStruct<StorySegment>()
    .ExecuteAsync();
```

## Unity Examples

### Example 1: NPC Dialog Generator

```csharp
[StrictJsonSchema("npc_dialog", "NPC dialogue lines.")]
public class NPCDialog
{
    [JsonSchemaProperty("greeting", "Opening greeting line.")]
    public string Greeting { get; set; }

    [JsonSchemaProperty("questions", "Possible questions from the NPC.")]
    public List<string> Questions { get; set; }

    [JsonSchemaProperty("farewell", "Closing farewell line.")]
    public string Farewell { get; set; }
}

public class NPCController : MonoBehaviour
{
    private NPCDialog dialog;
    
    async void Start()
    {
        dialog = await "Generate dialog for a friendly shopkeeper"
            .GENStruct<NPCDialog>()
            .ExecuteAsync();
        
        Debug.Log(dialog.Greeting);
    }
}
```

### Example 2: Item Database Generator

```csharp
[StrictJsonSchema("item_database", "Game item database.")]
public class ItemDatabase
{
    [JsonSchemaProperty("weapons", "List of weapons.")]
    public List<WeaponData> Weapons { get; set; }

    [JsonSchemaProperty("armors", "List of armors.")]
    public List<ArmorData> Armors { get; set; }
}

[StrictJsonSchema("weapon_data", "A weapon entry.")]
public class WeaponData
{
    [JsonSchemaProperty("name", "Weapon name.")]
    public string Name { get; set; }

    [JsonSchemaProperty("damage", "Base damage.")]
    public int Damage { get; set; }

    [JsonSchemaProperty("attack_speed", "Attack speed multiplier.")]
    public float AttackSpeed { get; set; }
}

ItemDatabase db = await "Generate 10 weapons and 5 armors"
    .GENStruct<ItemDatabase>()
    .ExecuteAsync();
```

### Example 3: Level Configuration

```csharp
[StrictJsonSchema("level_config", "Level configuration.")]
public class LevelConfig
{
    [JsonSchemaProperty("level_name", "Display name for the level.")]
    public string LevelName { get; set; }

    [JsonSchemaProperty("enemy_count", "Number of enemies to spawn.")]
    public int EnemyCount { get; set; }

    [JsonSchemaProperty("spawn_points", "List of spawn point coordinates.")]
    public List<Vector3Serializable> SpawnPoints { get; set; }

    [JsonSchemaProperty("time_limit", "Level time limit in seconds.")]
    public float TimeLimit { get; set; }
}

[StrictJsonSchema("vector3", "3D coordinate.")]
public class Vector3Serializable
{
    [JsonSchemaProperty("x", "X coordinate.")]
    public float X { get; set; }

    [JsonSchemaProperty("y", "Y coordinate.")]
    public float Y { get; set; }

    [JsonSchemaProperty("z", "Z coordinate.")]
    public float Z { get; set; }
    
    public Vector3 ToVector3() => new Vector3(X, Y, Z);
}

LevelConfig config = await "Generate level 5 configuration"
    .GENStruct<LevelConfig>()
    .ExecuteAsync();
```

### Example 4: Character Sheet

```csharp
[StrictJsonSchema("character_sheet", "A full character sheet.")]
public class CharacterSheet
{
    [JsonSchemaProperty("name", "Character name.")]
    public string Name { get; set; }

    [JsonSchemaProperty("race", "Character race.")]
    public string Race { get; set; }

    [JsonSchemaProperty("class", "Character class.")]
    public string Class { get; set; }

    [JsonSchemaProperty("level", "Character level.")]
    public int Level { get; set; }

    [JsonSchemaProperty("attributes", "Attribute set.")]
    public AttributeSet Attributes { get; set; }

    [JsonSchemaProperty("skills", "List of skills.")]
    public List<Skill> Skills { get; set; }

    [JsonSchemaProperty("equipment", "Equipped items.")]
    public Equipment Equipment { get; set; }
}

[StrictJsonSchema("attribute_set", "D&D-style attributes.")]
public class AttributeSet
{
    [JsonSchemaProperty("strength")] public int Strength { get; set; }
    [JsonSchemaProperty("dexterity")] public int Dexterity { get; set; }
    [JsonSchemaProperty("constitution")] public int Constitution { get; set; }
    [JsonSchemaProperty("intelligence")] public int Intelligence { get; set; }
    [JsonSchemaProperty("wisdom")] public int Wisdom { get; set; }
    [JsonSchemaProperty("charisma")] public int Charisma { get; set; }
}

CharacterSheet sheet = await "Generate a level 5 elf ranger"
    .GENStruct<CharacterSheet>()
    .ExecuteAsync();
```

## Best Practices

### ✅ Do

```csharp
// ✅ Use [StrictJsonSchema] with clear names and descriptions
[StrictJsonSchema("item", "A game item.")]
public class Item
{
    [JsonSchemaProperty("name", "Item name.")]
    public string Name { get; set; }

    [JsonSchemaProperty("quantity", "Stack count.")]
    public int Quantity { get; set; }
}

// ✅ Provide context in prompt
Character c = await "Generate a medieval warrior character with stats"
    .GENStruct<Character>()
    .ExecuteAsync();

// ✅ Handle null values
if (character?.Stats != null)
{
    Debug.Log($"HP: {character.Stats.Health}");
}
```

### ❌ Don't

```csharp
// ❌ Missing [StrictJsonSchema] attribute — will throw at runtime
public class Item
{
    public string Name { get; set; }
    public int Quantity { get; set; }
}

// ❌ Vague prompts
Character c = await "character"
    .GENStruct<Character>()
    .ExecuteAsync();

// ❌ Complex nested structures (may fail)
public class OverlyComplex
{
    public Dictionary<string, List<Dictionary<int, List<string>>>> Data { get; set; }
}
```

## Validation

Always validate generated data:

```csharp
try
{
    Character character = await "Generate character"
        .GENStruct<Character>()
        .ExecuteAsync();
    
    // Validate
    if (string.IsNullOrEmpty(character.Name))
        throw new Exception("Invalid character: missing name");
    
    if (character.Level < 1 || character.Level > 100)
        throw new Exception("Invalid character: level out of range");
    
    // Use validated data
    Debug.Log($"Valid character: {character.Name}");
}
catch (Exception ex)
{
    Debug.LogError($"Character generation failed: {ex.Message}");
}
```

## Provider Support

| Provider | Support | Models |
|----------|---------|--------|
| OpenAI | ✅ Full | GPT-4o, GPT-4 |
| Anthropic | ✅ Full | Claude 3.5 |
| Google Gemini | ⚠️ Partial | Gemini 1.5 Pro |
| OpenRouter | ✅ Varies | Model-dependent |

## Error Handling

```csharp
try
{
    Character character = await "Generate character"
        .GENStruct<Character>()
        .ExecuteAsync();
}
catch (JsonException ex)
{
    Debug.LogError($"JSON parsing failed: {ex.Message}");
}
catch (AIApiException ex)
{
    Debug.LogError($"API error: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Next Steps

- [Chat Completions](chat-completions.md) - General text generation
- [Code Generation](code-generation.md) - Generate code
- [Responses API](responses-api.md) - Advanced features
