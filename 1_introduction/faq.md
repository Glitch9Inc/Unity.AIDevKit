---
icon: square-question
---

# FAQ

{% hint style="info" %}
Please join [Discord](https://discord.gg/hgajxPpJYf) if you have a question that's not listed here.
{% endhint %}

## 💰 Pricing & Free Tiers

<details>

<summary>My OpenAI free trial has ended or is inactive</summary>

There are lots of other ways to continue using AI Dev Kit. For free, if you want.

* **Upgrade to a paid OpenAI API account** – This is the most straightforward and affordable option. Most users spend less than $1 per month when actively using AI Dev Kit.

- **Try free alternatives like Google Gemini or OpenRouter** – Both Google and OpenRouter offers access to several free models that work seamlessly with AI Dev Kit. It's a great option if you're looking to continue development without spending anything.
- **If you're looking for a local, cost-free option, AI Dev Kit also supports Ollama as an AI backend.** Ollama runs models directly on your device (or your own server), eliminating the need for API keys or internet access.

* **Create a new OpenAI account** – If you have access to an additional phone number, you may create a new OpenAI account to receive a new free trial API key.

</details>

<details>

<summary>I have ChatGPT Plus subscription. Why can't I use OpenAI's paid models?</summary>

**Subscribing to ChatGPT Plus doesn't give you API access — it's only for using ChatGPT on the web.**\
If you're planning to use ChatGPT inside Unity or any other app, you'll need to set up **API billing** by enabling the "Pay-as-you-go" option in your OpenAI account.

</details>

<details>

<summary>I'm a student. Do you give out vouchers?</summary>

I appreciate your interest in **AI Dev Kit**! However, I currently give out vouchers **only to YouTubers, streamers, or influencers** who can help promote the asset publicly.

{% hint style="info" %}
This is because each voucher is limited and meant for marketing purposes.\
If you have an audience and would like to showcase the asset in a video or post, feel free to reach out and share your content!
{% endhint %}

</details>

<details>

<summary>I'm broke. What are the free API options?</summary>

No worries — you can still use **AI Dev Kit** without spending a dime. Here are several zero-cost options that work seamlessly:

* **Run models locally with Ollama**\
  Ollama lets you run powerful open-source models like LLaMA 3 directly on your machine, with **no API key or internet required**. It’s fully supported as a local backend in AI Dev Kit.\
  [Start settings up Ollama here. ](../quick-start/self-hosting-with-ollama.md)

- **Use Google Gemini's free tier**\
  Google Cloud offers a monthly free quota for Gemini models.\
  Simply sign up for a Google Cloud account, generate an API key, and you’re good to go.\
  [Get Google's free API key here.](../platform-api/google-gemini/)
- **Try OpenRouter’s free models**\
  **OpenRouter** aggregates access to various models — some of which are free to use without authentication. You can easily distinguish which models are free using [AI Dev Kit Model Library.](broken-reference)\
  [Get OpenRouter's free API key here.](../quick-start/api-key-setup/openrouter.md)

</details>

<details>

<summary>What are the limitations of free API tiers?</summary>

Free tiers vary significantly by provider and **change frequently**. Here's what to generally expect:

* **Google Gemini** – Generous free tier with daily request limits. Check [Google's pricing page](https://ai.google.dev/pricing) for current quotas.
* **OpenRouter** – Some models are completely free with no authentication required. Free models typically have lower rate limits and may have queuing during peak hours.
* **OpenAI** – Offers limited free trial credits for new accounts. Expires after a set period. Check [OpenAI's pricing](https://openai.com/api/pricing/) for details.
* **Ollama (Local)** – No API limits whatsoever since it runs on your own machine. Performance depends on your hardware.

**Always verify current limits directly with each provider** as they update their free tier policies regularly.

Free tiers work well for:
* Single-player experiences where you control AI usage
* Development and testing phases
* Low-traffic applications

For high-traffic multiplayer games, consider paid tiers to avoid rate limiting and ensure reliability.

</details>

<details>

<summary>Can I use production-grade AI completely for free?</summary>

**Realistically, no.**

Running production-level AI at scale always requires compute resources — whether through a paid API provider or by hosting and maintaining your own infrastructure.

Free tiers are designed for development, testing, and light usage. They are not intended for unlimited or high-traffic production deployments.

If you want full control without API costs, you can host open-source models yourself (e.g., via Ollama), but this requires sufficient hardware, optimization effort, and ongoing maintenance.

For most developers, a paid API tier is significantly more cost-effective than building and operating custom AI infrastructure.

AI Dev Kit exists to make that integration simple, unified, and production-ready — without requiring developers to build and maintain their own AI backend stack.

</details>

***

## 🎮 Production & Commercial Use

<details>

<summary>Is this safe to use in a commercial game?</summary>

Yes — as long as you follow the licensing terms of each AI provider.\
Most major providers (OpenAI, Google, ElevenLabs) allow their APIs to be used in commercial products, including games.

However, always check for:

* **Attribution requirements**

- **Content usage limits (e.g. for generated voices or images)**

* **Prohibited use cases (e.g. misinformation, impersonation)**

Refer to each provider’s Terms of Use to ensure full compliance.

</details>

<details>

<summary>Can I expose free provider APIs directly in my published game?</summary>

**Technically yes, but it's not recommended for most cases.**

If you embed your API key in a published game:
* **All players share your quota** – Your free tier limits will be consumed quickly as more players use your game
* **API costs scale with users** – Each player action that calls the API counts against your account
* **Security risk** – API keys in client-side code can be extracted and abused

**Better approaches:**
* **Use Ollama for local inference** – Players run models on their own machines, eliminating API costs and server dependencies
* **Build a backend proxy** – Route API calls through your own server where you can implement rate limiting, user authentication, and cost control
* **Use paid tiers with safeguards** – Set spending limits and implement usage caps per user

For single-player games with optional AI features, direct API usage may work. For multiplayer or always-online games, use a backend intermediary.

</details>

***

## 🔧 Technical & Integration

<details>

<summary>Why use AI Dev Kit with Ollama instead of building my own integration?</summary>

AI Dev Kit provides several advantages over direct Ollama integration:

* **Unified API across providers** – Write code once and switch between Ollama, OpenAI, Google Gemini, or any other provider without changing your game logic
* **Built-in Unity components** – Drop-in components like AIAgent, ImageGenerator, and SpeechGenerator work out of the box with automatic lifecycle management
* **Streaming support** – Real-time token streaming for chat responses, with built-in UI handlers
* **Model management** – Browse, select, and switch between Ollama models directly in Unity Editor preferences
* **Error handling & retries** – Production-ready error handling, timeout management, and automatic retry logic
* **Type-safe APIs** – Strongly-typed C# APIs with IntelliSense support and compile-time validation

Building your own integration means maintaining HTTP clients, handling JSON serialization, managing WebSocket connections for streaming, and writing boilerplate for every provider. AI Dev Kit handles all of this so you can focus on game logic.

</details>

***

## ⚡ Features & Capabilities

<details>

<summary>What does "lightweight" mean? Can AI Dev Kit handle complex tasks?</summary>

**"Lightweight" refers to the integration code and performance overhead, not the capabilities.**

AI Dev Kit is lightweight in terms of:
* **UniTask-based async operations** – Extremely efficient async/await without the overhead of C# Tasks or Unity Coroutines. Zero garbage allocation per call.
* **Fluent API design** – One-line access to all AI features with method chaining (e.g., `"prompt".GENResponse().ExecuteAsync()`)
* **Minimal boilerplate** – Generate images, speech, or chat responses without complex setup code
* **Fast integration** – Add API keys and start using AI features in minutes

**AI Dev Kit is NOT limited in power or complexity.** It supports:
* **Production-ready AI infrastructure** – Text generation, image creation, voice synthesis, and speech recognition at scale
* **High-quality asset generation** – Professional images, voices, sound effects, and music through provider APIs
* **Complex AI agents** – Tool calling, function execution, multi-step reasoning, and conversation memory with full streaming support

**Code generation is available as an experimental feature** in the Playground, but AI Dev Kit's core focus is providing a robust AI infrastructure layer for production games—not automated code scaffolding.

The "power" comes from the AI models you choose (GPT-4, Claude, Gemini, etc.). AI Dev Kit simply makes accessing that power effortless. Whether you're generating a simple sprite or architecting a full dialogue system, AI Dev Kit scales to your needs.

</details>

<details>

<summary>Can I use AI Dev Kit for code generation or UI scaffolding?</summary>

**AI Dev Kit is primarily a production AI infrastructure layer, not a code generation tool.**

The core features are:
* **Runtime AI integration** – Text, image, audio, and speech APIs for live gameplay
* **AI Agents** – Chatbots, voice agents, and assistants with tool calling
* **Asset generation** – Images, voices, sound effects, and music

**Code generation is available experimentally** through the Editor Playground feature, where you can:
* Test AI models for generating Unity scripts
* Experiment with code completion and refactoring
* Prototype simple components

However, this is a **side feature for experimentation**, not the main purpose. If you need robust code scaffolding or automated UI generation, consider dedicated tools like GitHub Copilot or Cursor AI.

AI Dev Kit excels at bringing AI capabilities **into your game at runtime**—NPC dialogue, procedural content, dynamic voices, etc.

</details>
