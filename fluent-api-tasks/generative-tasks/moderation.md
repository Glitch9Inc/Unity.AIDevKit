---
description: Analyze text input for potentially harmful or inappropriate content using GENModeration.
icon: shield
---

# Moderation

returns [`ModerationResult`](<https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.ModerationResult>    .html)

**GENModeration** is used to analyze text input for potentially harmful or inappropriate content. It helps ensure that generated content adheres to safety and ethical guidelines by flagging content that may violate these standards.

**Basic Usage**

```csharp
List<SafetySetting> safetySettings = new List<SafetySetting>
{
    new SafetySetting
    {
        Category = HarmCategory.Hate,
        Threshold = HarmBlockThreshold.Medium
    },
    new SafetySetting
    {
        Category = HarmCategory.SelfHarm,
        Threshold = HarmBlockThreshold.Low
    }
};        

ModerationResult result = await "my text input to moderate"
    .GENModeration(safetySettings)
    .ExecuteAsync();
    
if (result.IsFlagged)
{
    string reason = result.Reason;
    // Handle flagged content
}
else
{
    // Proceed with safe content
}
```
