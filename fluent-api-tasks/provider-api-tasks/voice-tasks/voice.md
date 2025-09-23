---
description: List built-in and custom TTS voices across providers with unified pagination and sorting
icon: microphone-lines
---
 
# Voices

returns [`IVoiceData[]`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IVoiceData.html)

Enumerate **provider voices** (catalog voices) and **custom voices** (your org/workspace) in a provider-agnostic way.

This page covers:

* **ListVoices** — list provider voices
* **ListCustomVoices** — list your custom voices

Both tasks support `CursorQuery` for limit/order/after/before. Adapters ignore unsupported params per provider.

---

## Quick Start

```csharp
using Glitch9.AIDevKit;

// Provider voices (e.g., ElevenLabs, OpenAI TTS-capable providers)
IVoiceData[] voices = await Api.ElevenLabs
    .ListVoices()                 // no query = provider defaults
    .ExecuteAsync();

// Custom voices (org/workspace scoped)
IVoiceData[] custom = await Api.ElevenLabs
    .ListCustomVoices()
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

// Next page (forward) using the last item as cursor
q.After = page1.Length > 0 ? page1[^1].Id : null;

IVoiceData[] page2 = await Api.ElevenLabs
    .ListVoices(q)
    .ExecuteAsync();

// Previous page (backward) example
q.Before = page1.Length > 0 ? page1[0].Id : null;

IVoiceData[] prev = await Api.ElevenLabs
    .ListVoices(q)
    .ExecuteAsync();
```

> Tip: Persist `After/Before` cursors if you need stable paging across sessions.

---

## Filtering Notes

* `CursorQuery` is **generic**: `Limit`, `Order`, `After`, `Before`.
* Provider adapters may expose richer filters internally (labels, language, gender, style, etc.). If you need those, use the provider’s dedicated query type/methods (when available in your SDK). Otherwise, do client-side filtering on the returned `IVoiceData[]`.

Example client-side filter:

```csharp
var enVoices = voices.Where(v => v.LanguageCode?.StartsWith("en", StringComparison.OrdinalIgnoreCase) == true).ToArray();
```

---

## Error Handling & Best Practices

* Wrap calls in `try/catch`; surface partial results gracefully.
* Keep `Limit` small for UI lists; page on scroll.
* Cache frequently used pages locally to reduce API calls.
* Not all providers separate “catalog” vs “custom” — the adapter normalizes to these two calls.
