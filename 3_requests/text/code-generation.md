---
icon: code
---

# Code Generation

The `.GENCode()` method provides specialized code generation capabilities optimized for programming tasks.

## Basic Usage

```csharp
string code = await "Create a C# class for a player character"
    .GENCode()
    .ExecuteAsync();
```

## Input Types

### String Input

```csharp
string code = await "Unity script that rotates an object"
    .GENCode()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("Create a {language} {type} for {purpose}");
string code = await prompt
    .GENCode()
    .ExecuteAsync();
```

## Configuration

### Language Selection

Specify the programming language in your prompt:

```csharp
string csharp = await "Create a C# singleton pattern"
    .GENCode()
    .ExecuteAsync();

string python = await "Create a Python data parser"
    .GENCode()
    .ExecuteAsync();

string javascript = await "Create a JavaScript async function"
    .GENCode()
    .ExecuteAsync();
```

### Code Style

Request specific coding styles:

```csharp
string code = await @"
Create a Unity MonoBehaviour for player movement.
Use:
- Modern C# features (properties, null coalescing)
- XML documentation comments
- Defensive programming
"
    .GENCode()
    .ExecuteAsync();
```

### Complexity Level

```csharp
// Simple version
string simple = await "Basic player movement script"
    .GENCode()
    .SetTemperature(0.3f)
    .ExecuteAsync();

// Advanced version
string advanced = await "Advanced player movement with physics and animation"
    .GENCode()
    .SetTemperature(0.7f)
    .ExecuteAsync();
```

## Common Use Cases

### 1. Generate New Code

```csharp
string newClass = await "Create a C# inventory system class"
    .GENCode()
    .ExecuteAsync();

Debug.Log(newClass);
```

### 2. Refactor Existing Code

```csharp
string refactored = await $@"
Refactor this code to be more maintainable:

{existingCode}
"
    .GENCode()
    .ExecuteAsync();
```

### 3. Explain Code

```csharp
string explanation = await $@"
Explain what this code does:

{complexCode}
"
    .GENCode()
    .ExecuteAsync();
```

### 4. Fix Bugs

```csharp
string fixed = await $@"
Fix the bugs in this code:

{buggyCode}

The error message is: {errorMessage}
"
    .GENCode()
    .ExecuteAsync();
```

### 5. Add Features

```csharp
string enhanced = await $@"
Add save/load functionality to this code:

{existingCode}
"
    .GENCode()
    .ExecuteAsync();
```

### 6. Convert Between Languages

```csharp
string converted = await $@"
Convert this Python code to C#:

{pythonCode}
"
    .GENCode()
    .ExecuteAsync();
```

## Unity-Specific Examples

### Example 1: MonoBehaviour Component

```csharp
string playerScript = await @"
Create a Unity MonoBehaviour for a 2D player character with:
- WASD movement
- Jump with Space
- Ground check using raycast
- Serialized fields for speed and jump force
"
    .GENCode()
    .ExecuteAsync();

Debug.Log(playerScript);
```

### Example 2: ScriptableObject

```csharp
string itemData = await @"
Create a Unity ScriptableObject for game items with:
- Item name, description, icon
- Item type enum (Weapon, Consumable, Quest)
- Rarity level
- Value in gold
"
    .GENCode()
    .ExecuteAsync();
```

### Example 3: Editor Script

```csharp
string editorTool = await @"
Create a Unity Editor script that:
- Adds a menu item 'Tools/Organize Scene'
- Finds all GameObjects with specific tags
- Organizes them into parent folders
"
    .GENCode()
    .ExecuteAsync();
```

### Example 4: Coroutine

```csharp
string coroutine = await @"
Create a Unity coroutine that:
- Fades a UI Image from alpha 0 to 1
- Takes duration as parameter
- Uses AnimationCurve for easing
"
    .GENCode()
    .ExecuteAsync();
```

## Best Practices

### ✅ Do

```csharp
// ✅ Be specific about requirements
string code = await @"
Create a C# class with:
- Public properties for health and maxHealth
- Method TakeDamage(int amount) with validation
- Event onHealthChanged
- XML documentation
"
    .GENCode()
    .ExecuteAsync();

// ✅ Specify coding standards
string code = await "C# singleton using lazy initialization"
    .GENCode()
    .ExecuteAsync();

// ✅ Include context
string code = await $@"
Add error handling to this method:

{existingMethod}
"
    .GENCode()
    .ExecuteAsync();
```

### ❌ Don't

```csharp
// ❌ Too vague
string code = await "make a game"
    .GENCode()
    .ExecuteAsync();

// ❌ Too complex for single request
string code = await "create entire MMO game system"
    .GENCode()
    .ExecuteAsync();

// ❌ Mixing multiple unrelated tasks
string code = await "player movement and inventory and UI and networking"
    .GENCode()
    .ExecuteAsync();
```

## Provider Support

| Provider | Support | Best Models |
|----------|---------|-------------|
| OpenAI | ✅ Full | GPT-4o, GPT-4 |
| Anthropic | ✅ Full | Claude 3.5 Sonnet |
| Google Gemini | ✅ Full | Gemini 1.5 Pro |
| OpenRouter | ✅ Full | Various |
| Groq | ✅ Full | Llama 3 |

## Error Handling

```csharp
try
{
    string code = await "Create player class"
        .GENCode()
        .ExecuteAsync();
    
    // Validate generated code
    if (string.IsNullOrEmpty(code))
    {
        Debug.LogWarning("Empty code generated");
        return;
    }
    
    // Use the generated code
    Debug.Log(code);
}
catch (AIApiException ex)
{
    Debug.LogError($"Code generation failed: {ex.Message}");
}
```

## Integration Example

```csharp
public class CodeGeneratorWindow : EditorWindow
{
    private string prompt = "";
    private string generatedCode = "";
    
    [MenuItem("Tools/AI Code Generator")]
    static void ShowWindow()
    {
        GetWindow<CodeGeneratorWindow>("Code Generator");
    }
    
    async void OnGUI()
    {
        EditorGUILayout.LabelField("Prompt:", EditorStyles.boldLabel);
        prompt = EditorGUILayout.TextArea(prompt, GUILayout.Height(100));
        
        if (GUILayout.Button("Generate Code"))
        {
            generatedCode = await prompt
                .GENCode()
                .SetModel(OpenAIModel.GPT4o)
                .ExecuteAsync();
        }
        
        if (!string.IsNullOrEmpty(generatedCode))
        {
            EditorGUILayout.LabelField("Generated:", EditorStyles.boldLabel);
            EditorGUILayout.TextArea(generatedCode, GUILayout.ExpandHeight(true));
            
            if (GUILayout.Button("Copy to Clipboard"))
            {
                GUIUtility.systemCopyBuffer = generatedCode;
            }
        }
    }
}
```

## Next Steps

- [Structured Output](structured-output.md) - Type-safe JSON generation
- [Chat Completions](chat-completions.md) - General text generation
- [Responses API](responses-api.md) - Advanced features
