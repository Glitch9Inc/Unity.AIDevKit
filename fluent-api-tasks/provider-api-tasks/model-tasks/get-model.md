---
description: Retrieve a base (provider) model by ID
icon: search
---

# GetModel

returns [`IModelData`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IModelData.html)

Fetch a **base** (provider) model (e.g., OpenAI catalog) by its identifier.

## Basic Usage

```csharp
using Glitch9.AIDevKit;

// Provider base model
IModelData model = await Api.OpenAI
    .GetModel("gpt-4o")
    .ExecuteAsync();
```

## Notes

* Throws if `modelId` is null/empty.
* Targets **provider catalog** models (not your custom fine-tunes).
* Scope and visibility depend on your API credentials/org.
