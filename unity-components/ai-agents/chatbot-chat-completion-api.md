---
description: >-
  Use this component to integrate an AI-powered chatbot into your scene using
  the OpenAI Chat Completion API.
icon: comment-lines
---

# Chatbot (Chat Completion API)

### What it does

This component enables:

* Text input & response using LLM
* Voice input using Speech-to-Text
* Voice output using Text-to-Speech
* Image generation if the user requests it
* Tool/function calling if AI requests an action

***

<figure><img src="../../.gitbook/assets/image (107).png" alt="" width="563"><figcaption></figcaption></figure>

### 1. Setting up the Chatbot

**Step 1:** Add the `Chatbot` component to any GameObject.

**Step 2:** Choose or create a Chat Session:

* Stores the message history and config
* Set it using `Selected Session`

**Step 3:** Select a model:

* OpenAI GPT-4o is default
* Optional: choose a separate model for summarization

***

### 2. Optional Features (Modules)

You can plug in optional features to extend the chatbot:

<table><thead><tr><th width="182.3333740234375">Feature</th><th width="188">Component</th><th>What it does</th></tr></thead><tbody><tr><td>Voice input</td><td><code>SpeechToText</code></td><td>User can speak instead of typing</td></tr><tr><td>Voice output</td><td><code>TextToSpeech</code></td><td>AI responses are played as audio</td></tr><tr><td>Image responses</td><td><code>ImageGenerator</code></td><td>User prompts like “draw a cat” return images</td></tr><tr><td>Tool execution</td><td><code>FunctionManager</code></td><td>AI can trigger Unity methods</td></tr></tbody></table>

> If you leave these blank, the chatbot still works with text input/output only.

***

### 3. Real-time options

| Setting     | Effect                                                |
| ----------- | ----------------------------------------------------- |
| `Stream`    | Show streamed tokens while AI responds                |
| `Auto Save` | Automatically updates chat session after each message |

***

### 4. Event Hooks

You can connect `UnityEvents` to respond when:

* A message is sent or received
* AI requests a function call
* A streamed token arrives
* An error occurs

***

### 5. Advanced Options (Optional)

You can manually override things like temperature, max tokens, top-p, etc.\
These are useful if you want finer control over the LLM behavior.
