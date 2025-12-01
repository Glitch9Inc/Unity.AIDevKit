---
icon: shield-halved
---

# Moderation

AI Dev Kit provides content moderation capabilities to detect potentially harmful content using `.GENModeration()`.

## What is Content Moderation?

Content moderation helps you:

- Detect inappropriate or harmful content
- Filter user-generated content
- Comply with content policies
- Protect users from harmful material

## Basic Usage

```csharp
var result = await "User input text"
    .GENModeration(safetySettings)
    .ExecuteAsync();

if (result.IsFlagged)
{
    Debug.LogWarning($"Content flagged: {result.Categories}");
}
```

## Categories

Moderation typically checks for:

- **Hate**: Hateful or discriminatory content
- **Harassment**: Bullying or harassing content
- **Self-harm**: Self-harm or suicide-related content
- **Sexual**: Sexual or explicit content
- **Violence**: Violent or graphic content

## Input Types

### String Input

```csharp
var result = await "Check this text"
    .GENModeration(safetySettings)
    .ExecuteAsync();
```

### ModerationPrompt

```csharp
var prompt = new ModerationPrompt("Text with context");
var result = await prompt
    .GENModeration(safetySettings)
    .ExecuteAsync();
```

### IModeratable

```csharp
var moderatable = GetModeratable();
var result = await moderatable
    .GENModeration(safetySettings)
    .ExecuteAsync();
```

## Safety Settings

```csharp
var settings = new[]
{
    new SafetySetting
    {
        Category = HarmCategory.Hate,
        Threshold = HarmThreshold.BlockMediumAndAbove
    },
    new SafetySetting
    {
        Category = HarmCategory.Violence,
        Threshold = HarmThreshold.BlockHighOnly
    }
};

var result = await text
    .GENModeration(settings)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Chat Filter

```csharp
public class ChatFilter : MonoBehaviour
{
    private SafetySetting[] safetySettings;
    
    void Start()
    {
        safetySettings = new[]
        {
            new SafetySetting
            {
                Category = HarmCategory.Hate,
                Threshold = HarmThreshold.BlockMediumAndAbove
            }
        };
    }
    
    public async UniTask<bool> IsMessageSafe(string message)
    {
        var result = await message
            .GENModeration(safetySettings)
            .ExecuteAsync();
        
        return !result.IsFlagged;
    }
    
    public async UniTask<string> FilterMessage(string message)
    {
        bool isSafe = await IsMessageSafe(message);
        
        if (!isSafe)
        {
            return "[Message removed by moderator]";
        }
        
        return message;
    }
}
```

### Example 2: User Content Validator

```csharp
public class ContentValidator : MonoBehaviour
{
    public async UniTask<ValidationResult> ValidateContent(string content)
    {
        var settings = GetStrictSettings();
        
        var result = await content
            .GENModeration(settings)
            .ExecuteAsync();
        
        return new ValidationResult
        {
            IsValid = !result.IsFlagged,
            Reason = result.IsFlagged ? GetReason(result) : null
        };
    }
    
    private SafetySetting[] GetStrictSettings()
    {
        return new[]
        {
            new SafetySetting
            {
                Category = HarmCategory.Hate,
                Threshold = HarmThreshold.BlockLowAndAbove
            },
            new SafetySetting
            {
                Category = HarmCategory.Violence,
                Threshold = HarmThreshold.BlockLowAndAbove
            }
        };
    }
}
```

### Example 3: Real-time Chat Moderator

```csharp
public class RealtimeChatModerator : MonoBehaviour
{
    private Queue<string> messageQueue = new();
    
    public void OnUserMessage(string message)
    {
        ModerateAndSendAsync(message).Forget();
    }
    
    async UniTaskVoid ModerateAndSendAsync(string message)
    {
        var result = await message
            .GENModeration(GetDefaultSettings())
            .ExecuteAsync();
        
        if (result.IsFlagged)
        {
            NotifyUser("Your message was blocked due to content policy.");
            return;
        }
        
        SendMessageToChat(message);
    }
}
```

## Provider Support

### OpenAI

```csharp
var result = await text
    .GENModeration(settings)
    .ExecuteAsync();
```

### Google Gemini

```csharp
var settings = new[]
{
    new SafetySetting
    {
        Category = HarmCategory.Hate,
        Threshold = HarmThreshold.BlockMediumAndAbove
    }
};

var result = await text
    .GENModeration(settings)
    .ExecuteAsync();
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Check user-generated content
string userInput = GetUserInput();
bool isSafe = await IsContentSafe(userInput);

// ✅ Use appropriate thresholds
var settings = GetThresholdsForContext();

// ✅ Provide user feedback
if (!isSafe)
{
    ShowWarning("Content violates community guidelines");
}

// ✅ Log moderation events
LogModerationEvent(result);
```

### ❌ Bad Practices

```csharp
// ❌ Don't skip moderation for privileged users
// Everyone's content should be checked

// ❌ Don't moderate in Update()
void Update()
{
    await text.GENModeration(settings).ExecuteAsync();  // NO!
}

// ❌ Don't ignore results
await text.GENModeration(settings).ExecuteAsync();  // Check result!
```

## Next Steps

- [Content Moderation](content-moderation.md) - Detailed moderation guide
- [Safety Settings](safety-settings.md) - Configure safety thresholds
