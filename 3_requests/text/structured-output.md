---
icon: brackets-curly
---

# Structured Output

The `.GENStruct<T>()` method generates JSON responses that are automatically mapped to strongly-typed C# classes.

## Basic Usage

```csharp
[JsonSchema]
public class Character
{
    public string Name { get; set; }
    public int Level { get; set; }
    public string Class { get; set; }
}

Character hero = await "Generate a level 10 warrior"
    .GENStruct<Character>()
    .ExecuteAsync();

Debug.Log($"{hero.Name} - Lv.{hero.Level} {hero.Class}");
```

## Defining Schema Classes

### 1. Simple Class

```csharp
[JsonSchema]
public class UserProfile
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Email { get; set; }
}

UserProfile user = await "Create profile for John, age 30"
    .GENStruct<UserProfile>()
    .ExecuteAsync();
```

### 2. Nested Objects

```csharp
[JsonSchema]
public class GameCharacter
{
    public string Name { get; set; }
    public Stats Stats { get; set; }
    public List<string> Skills { get; set; }
}

[JsonSchema]
public class Stats
{
    public int Health { get; set; }
    public int Mana { get; set; }
    public int Strength { get; set; }
}

GameCharacter character = await "Create a mage character"
    .GENStruct<GameCharacter>()
    .ExecuteAsync();
```

### 3. Collections

```csharp
[JsonSchema]
public class ItemList
{
    public List<Item> Items { get; set; }
}

[JsonSchema]
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

[JsonSchema]
public class Weapon
{
    public string Name { get; set; }
    public int Damage { get; set; }
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
[JsonSchema]
public class ContactInfo
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
}

string text = "Contact John Doe at john@example.com or 555-1234";
ContactInfo contact = await $"Extract contact info from: {text}"
    .GENStruct<ContactInfo>()
    .ExecuteAsync();
```

### 2. Form Generation

```csharp
[JsonSchema]
public class QuestData
{
    public string Title { get; set; }
    public string Description { get; set; }
    public int RewardGold { get; set; }
    public int RequiredLevel { get; set; }
}

QuestData quest = await "Generate a beginner quest"
    .GENStruct<QuestData>()
    .ExecuteAsync();
```

### 3. Configuration Files

```csharp
[JsonSchema]
public class GameSettings
{
    public float MusicVolume { get; set; }
    public float SfxVolume { get; set; }
    public string Difficulty { get; set; }
    public bool EnableTutorial { get; set; }
}

GameSettings settings = await "Generate default game settings"
    .GENStruct<GameSettings>()
    .ExecuteAsync();
```

### 4. Content Generation

```csharp
[JsonSchema]
public class StorySegment
{
    public string Chapter { get; set; }
    public string Content { get; set; }
    public List<string> Choices { get; set; }
}

StorySegment story = await "Generate a story chapter with 3 choices"
    .GENStruct<StorySegment>()
    .ExecuteAsync();
```

## Unity Examples

### Example 1: NPC Dialog Generator

```csharp
[JsonSchema]
public class NPCDialog
{
    public string Greeting { get; set; }
    public List<string> Questions { get; set; }
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
[JsonSchema]
public class ItemDatabase
{
    public List<WeaponData> Weapons { get; set; }
    public List<ArmorData> Armors { get; set; }
}

[JsonSchema]
public class WeaponData
{
    public string Name { get; set; }
    public int Damage { get; set; }
    public float AttackSpeed { get; set; }
}

ItemDatabase db = await "Generate 10 weapons and 5 armors"
    .GENStruct<ItemDatabase>()
    .ExecuteAsync();
```

### Example 3: Level Configuration

```csharp
[JsonSchema]
public class LevelConfig
{
    public string LevelName { get; set; }
    public int EnemyCount { get; set; }
    public List<Vector3Serializable> SpawnPoints { get; set; }
    public float TimeLimit { get; set; }
}

[JsonSchema]
public class Vector3Serializable
{
    public float X { get; set; }
    public float Y { get; set; }
    public float Z { get; set; }
    
    public Vector3 ToVector3() => new Vector3(X, Y, Z);
}

LevelConfig config = await "Generate level 5 configuration"
    .GENStruct<LevelConfig>()
    .ExecuteAsync();
```

### Example 4: Character Sheet

```csharp
[JsonSchema]
public class CharacterSheet
{
    public string Name { get; set; }
    public string Race { get; set; }
    public string Class { get; set; }
    public int Level { get; set; }
    public AttributeSet Attributes { get; set; }
    public List<Skill> Skills { get; set; }
    public Equipment Equipment { get; set; }
}

[JsonSchema]
public class AttributeSet
{
    public int Strength { get; set; }
    public int Dexterity { get; set; }
    public int Constitution { get; set; }
    public int Intelligence { get; set; }
    public int Wisdom { get; set; }
    public int Charisma { get; set; }
}

CharacterSheet sheet = await "Generate a level 5 elf ranger"
    .GENStruct<CharacterSheet>()
    .ExecuteAsync();
```

## Best Practices

### ✅ Do

```csharp
// ✅ Use clear property names
[JsonSchema]
public class Item
{
    public string Name { get; set; }
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
// ❌ Unclear property names
public class Item
{
    public string N { get; set; }
    public int Q { get; set; }
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
