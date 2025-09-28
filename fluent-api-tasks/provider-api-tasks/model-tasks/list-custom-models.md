---
description: List custom / fine-tuned models with cursor pagination
icon: list-tree
---

# List Custom Models

returns [`IModelData[]`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.IModelData.html)

Enumerate **custom** (org/project) models with cursor pagination.

***

## Basic Usage

```csharp
using Glitch9.AIDevKit;

IModelData[] customModels = await Api.OpenAI
    .ListCustomModels()
    .ExecuteAsync();
```

***

## With `CursorQuery`

```csharp
using Glitch9.IO.Networking.RESTApi;

var q = new CursorQuery { Limit = 50, Order = SortOrder.Descending };
IModelData[] page1 = await Api.OpenAI.ListCustomModels(q).ExecuteAsync;

q.After = page1.Length > 0 ? page1[^1].Id : null;
IModelData[] page2 = await Api.OpenAI.ListCustomModels(q).ExecuteAsync();
```

***

## Notes

* Same cursor semantics as `ListModels` (`Limit`, `Order`, `After`, `Before`).
* Adapters ignore unsupported parameters per provider.
