---
description: List available base models with cursor pagination
icon: list
---

# ListModels

returns [`IModelData[]`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.IModelData.html)

Enumerate **base** (provider) models. Supports cursor pagination via `CursorQuery`

---

## Basic Usage

```csharp
using Glitch9.AIDevKit;

IModelData[] models = await Api.OpenAI
    .ListModels()
    .ExecuteAsync();
```

---

## With `CursorQuery` (pagination & sort)

```csharp
using Glitch9.IO.Networking.RESTApi;

// First page
var q = new CursorQuery {
    Limit = 100,                 // 1..100 (default 20)
    Order = SortOrder.Descending // by created_at (if supported)
};

IModelData[] first = await Api.OpenAI.ListModels(q).ExecuteAsync();

// Next page (forward)
q.After = first.Length > 0 ? first[^1].Id : null;
IModelData[] second = await Api.OpenAI.ListModels(q).ExecuteAsync();

// Previous page (backward)
q.Before = first.Length > 0 ? first[0].Id : null;
IModelData[] prev = await Api.OpenAI.ListModels(q).ExecuteAsync();
```

---

## Tips

* Prefer **server-side pagination** (`Limit`, `After`, `Before`) over client filtering.
* `Order` supports `Ascending` or `Descending` (by `created_at` where applicable).
