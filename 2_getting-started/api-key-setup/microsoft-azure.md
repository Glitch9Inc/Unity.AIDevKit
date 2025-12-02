---
icon: microsoft
---

# Microsoft Azure

## Getting Your API Key

1. Visit [Azure Portal](https://portal.azure.com)
2. Create **Azure OpenAI Service** resource
3. Go to **Keys and Endpoint** section
4. Copy **Key 1** or **Key 2**
5. Also copy your **Endpoint URL**

{% hint style="warning" %}
Azure OpenAI requires approval before access is granted.
{% endhint %}

## Adding to AI Dev Kit

**Tools > AI Dev Kit > Settings**

Find **Microsoft Azure** section and enter:

- **API Key**
- **Endpoint URL** (e.g., `https://your-resource.openai.azure.com`)
- **Deployment Name** (your model deployment name)

## Available Models

Azure OpenAI provides:

- GPT-4 / GPT-4 Turbo
- GPT-3.5 Turbo
- DALL-E 3
- Whisper
- Text-to-Speech

Models must be deployed in your Azure resource before use.

## Pricing

Check [Azure OpenAI Pricing](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/openai-service/) for current rates.
