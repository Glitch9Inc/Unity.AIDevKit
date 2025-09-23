---
description: Retrieve a custom / fine-tuned model by ID
icon: badge-check
---

# GetCustomModel

returns [`IModelData`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IModelData.html)

Fetch a **custom** (org/project-scoped) model by ID.

## Basic Usage

```csharp
using Glitch9.AIDevKit;

// Custom / fine-tuned model
IModelData custom = await Api.OpenAI
    .GetCustomModel("ft:gpt-4o:org:2025-09-01")
    .ExecuteAsync();
```

## Notes

* Throws if `modelId` is null/empty.
* Returns normalized `IModelData` regardless of provider.
