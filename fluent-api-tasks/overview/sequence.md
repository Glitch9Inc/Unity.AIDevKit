---
description: Executing Multiple Tasks in Sequence
icon: link
---

# Sequence

Sometimes you may want to run several generative tasks one after the other. For example, generate a piece of text, then use that text in a subsequent task, or just run a batch of independent tasks in order. **GENSequence** is a utility class to help with this.

{% hint style="info" %}
Each task in the sequence will start only after the previous one finishes. This is handy to ensure you don't overload your system with simultaneous requests or when one task's result is needed for the next (with manual passing of data in between).
{% endhint %}

#### **Generating text then speech**

Suppose we want to first generate a line of dialogue and then synthesize it into speech audio

```csharp
var sequence = new GENSequence();
string lineResult = null;  // to capture the text result of the first task

sequence.AppendText("Write a one-sentence line for a pirate character.".GENResponse())             
        .AppendTextToAudio(text => text.GENSpeech());

// Now execute the sequence
await sequence.ExecuteAsync();
```
