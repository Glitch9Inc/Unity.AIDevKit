---
description: Learn the basics of API keys
icon: key
---

# API Key Setup

## What is an API key and why do I need it?

An **API key** is a unique string of characters that identifies your app to an external AI service like OpenAI, Gemini, or ElevenLabs. It works like a secure password that allows your app to access models for text generation, image creation, speech synthesis, and more.

Without an API key, the AI Dev Kit cannot communicate with the AI provider — it's required for all runtime requests.

***

## How do I get my API key?

See each provider section under **Provider Setup** for step-by-step instructions.

[openai.md](openai.md "mention")

[google-gemini.md](google-gemini.md "mention")

[elevenlabs.md](elevenlabs.md "mention")

[openrouter.md](openrouter.md "mention")

***

## Where do I paste my API key in Unity?

{% hint style="info" %}
Go to **Edit > Preferences > AI Dev Kit > \[Each Provider Tab]**\
then paste your API key into the \[API key] field.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

***

## Can I use the same API key for multiple projects?

Yes.

Most providers — including OpenAI — allow you to reuse a single API key across multiple projects.\
However, keep in mind that all usage will count toward the same billing account and rate limits.\
If you're working on separate commercial products, it's a good idea to generate and track separate keys.

***

## Is my API key safe in a Unity project?

Yes.

**AI Dev Kit** supports **API Key Encryption (AES)** through the **Preferences window** under each provider tab.

{% hint style="info" %}
Go to **Edit > Preferences > AI Dev Kit > \[Each Provider Tab]**\
then press the little key button on the right side of your API key.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

***

## What happens if I lose my API key?

If you lose your API key, you won’t be able to authenticate requests to the AI provider.\
Most providers (like OpenAI) allow you to regenerate a new key from your account dashboard.\
**You cannot recover the original key once it's lost**, so make sure to store it in a password manager or reissue it when needed.
