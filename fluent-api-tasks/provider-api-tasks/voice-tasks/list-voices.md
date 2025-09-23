---
description: List provider (catalog) voices with optional pagination and sorting
icon: microphone-lines
---

# ListVoices

returns [`IVoiceData[]`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IVoiceData.html)

Enumerate **built-in voices** from a provider’s catalog (e.g., ElevenLabs, OpenAI).
Supports unified `CursorQuery` for pagination and ordering.

---

## Basic Usage

```csharp
using Glitch9.AIDevKit;

// Provider voices
IVoiceData[] voices = await Api.ElevenLabs
    .ListVoices()       // no query = provider defaults
    .ExecuteAsync();
```

---

## With Pagination & Sorting (`CursorQuery`)

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Networking.RESTApi;

// First page
var q = new CursorQuery {
    Limit = 50,                   // 1..100, default 20
    Order = SortOrder.Descending  // by created_at (if supported)
};

IVoiceData[] page1 = await Api.ElevenLabs
    .ListVoices(q)
    .ExecuteAsync();

// Next page (forward)
q.After = page1.Length > 0 ? page1[^1].Id : null;
IVoiceData[] page2 = await Api.ElevenLabs.ListVoices(q).ExecuteAsync();

// Previous page (backward)
q.Before = page1.Length > 0 ? page1[0].Id : null;
IVoiceData[] prev = await Api.ElevenLabs.ListVoices(q).ExecuteAsync();
```

> 💡 Persist `After/Before` cursors for stable navigation across sessions.

---

## Notes

* `CursorQuery` is **generic**: `Limit`, `Order`, `After`, `Before`.
* Some providers support richer filters (labels, language, gender, style). Use their dedicated SDK calls if available, or filter client-side:

```csharp
var enVoices = voices
    .Where(v => v.LanguageCode?.StartsWith("en", StringComparison.OrdinalIgnoreCase) == true)
    .ToArray();
```

---

## Best Practices

* Wrap in `try/catch`; surface partial results if paging fails.
* Use small `Limit` values for UI lists and lazy-load on scroll.
* Cache commonly used pages to avoid repeated API calls.
