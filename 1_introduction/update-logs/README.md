---
icon: calendar-lines-pen
---

# Update Logs

### 5.0.6 (2026-02-25)

**Fixed:**

* Inspector bugs resolved

---

### 5.0.5 (2026-02-25)

**Fixed:**

* Snippet generator fixed
* Initial snippets fixed

---

### 5.0.1 (2026-02-09)

**Changed:**

* Renamed `BinaryData` to `BinaryPayload`

**Removed:**

* Removed snippet references from OpenAI VoiceService

---

### 5.0.0 (2026-02-09)

#### Overview

After extensive real-world usage, structural limitations emerged in the previous architecture?�especially around streaming, event handling, and output consistency.

**v5.0.0 rebuilds the core abstractions from the ground up.**  
This is an architectural update, not a feature-driven one.

---

#### Key Changes

**Unified Streaming Architecture**

The streaming flow is now explicitly layered:

```
Transport
 ??StreamConverter
 ??StreamEvent<TPayload>
 ??Aggregation
 ??GenerativeEvent<TPayload, TFinal>
 ??Generated<T>
```

Applied consistently across:
* Text
* Audio
* Image
* Binary & multimodal streams

---

**StreamEvent & StreamHeader**

Streaming data is encapsulated in a single envelope:

`StreamEvent<TPayload>`

* Protocol metadata isolated in `StreamHeader`
* Payload contains domain data only
* Deterministic event structure

---

**Stream Converters**

Parsing-based streaming logic has been replaced with converter-based transformation.

`IStreamConverter` explicitly models:
* Protocol ??domain conversion
* Provider normalization
* Multi-stage stream pipelines

---

**GenerativeEvent Unification**

Streaming and non-streaming now share:

`GenerativeEvent<TPayload, TFinal>`

* Typed deltas during streaming
* Aggregation produces final output
* Usage metadata handled consistently

---

**Standardized Output**

All generative APIs now return:

`Generated<T>`

* Unified handling of single/multiple outputs
* Consistent metadata access
* Reduced API surface fragmentation

---

#### Breaking Changes

This version introduces breaking changes, particularly to:

* Streaming APIs
* Custom stream handlers/parsers
* Low-level provider integrations

**Existing code will require updates.**  
The major version bump intentionally signals this impact.

---

## Previous Versions

For version 4.x release notes, see [AI Dev Kit v4](ai-dev-kit-v4.md).

For version 3.x release notes, see [AI Dev Kit v3](ai-dev-kit-v3.md).

For version 2.x release notes, see [AI Dev Kit v2](ai-dev-kit-v2.md).

For version 1.x release notes, see [AI Dev Kit v1](ai-dev-kit-v1.md).
