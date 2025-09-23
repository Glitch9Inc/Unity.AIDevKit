---
description: List your custom voices (org/workspace) with optional pagination and sorting
icon: microphone-plus
---

# ListCustomVoices

returns [`IVoiceData[]`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IVoiceData.html)

Enumerate **custom voices** created in your org/workspace.
Supports unified `CursorQuery` for pagination and ordering.

---

## Basic Usage

```csharp
using Glitch9.AIDevKit;

// Custom voices
IVoiceData[] custom = await Api.ElevenLabs
    .ListCustomVoices()
    .ExecuteAsync();
```

---

## With Pagination & Sorting (`CursorQuery`)

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Networking.RESTApi;

var q = new CursorQuery { Limit = 20, Order = SortOrder.Descending };

IVoiceData[] page1 = await Api.ElevenLabs.ListCustomVoices(q).ExecuteAsync;

q.After = page1.Length > 0 ? page1[^1].Id : null;
IVoiceData[] page2 = await Api.ElevenLabs.ListCustomVoices(q).ExecuteAsync();
```

---

## Notes

* Same semantics as `ListVoices`.
* Adapters normalize provider results into a common `IVoiceData` schema.
* Not all providers distinguish “catalog” vs “custom”; where unsupported, both calls may return the same set.
