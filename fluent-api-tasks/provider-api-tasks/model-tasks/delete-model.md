---
description: Delete a custom model by ID (where supported)
icon: trash-2
---

# DeleteModel

returns `bool`

Delete a **custom** model by ID. Not all providers support model deletion.

## Basic Usage

```csharp
using Glitch9.AIDevKit;

bool ok = await Api.OpenAI
    .DeleteModel("ft:gpt-4o-mini:org:2025-08-12")
    .ExecuteAsync();

if (!ok) DevLog.Warning("Delete rejected or not supported.");
```

## Notes

* **Irreversible** — gate behind a confirmation UI.
* Provider limitations may apply (owner/admin permissions, base models cannot be deleted).
