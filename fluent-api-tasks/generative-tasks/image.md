---
description: Generate or edit an image using a diffusion model
icon: message-image
---

# Image

returns [`GeneratedImage`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedImage.html)

Generates an entirely new image from a text prompt using models like **DALL·E**, **Gemini** or **Imagen**.\
This is the primary entry point for creative image generation.

**Generating an Image**

```csharp
Texture2D result = await "A cat surfing a wave"
    .GENImage()
    .SetModel(ImageModel.DallE3)
    .ExecuteAsync();
```

**DALL·E-specific Configurations**

```csharp
Texture2D result = await "A cyberpunk city at night"
    .GENImage()
    .SetModel(ImageModel.DallE3)
    .SetSize(ImageSize._1024x1024)
    .SetQuality(ImageQuality.HD)
    .SetStyle(ImageStyle.Vivid)
    .ExecuteAsync();
```

**Google-specific Configurations (Gemini, Imagen3)**

```csharp
Texture2D result = await "An astronaut riding a horse"
    .GENImage()
    .SetModel(GoogleModel.Gemini2_0_Flash_Exp_Image_Generation)
    .SetAspectRatio(AspectRatio.Square)
    .SetPersonGeneration(PersonGeneration.Unspecified)
    .ExecuteAsync();
```

{% hint style="warning" %}
Results are saved to a temporary location unless you call `.SetOutputPath()`.
{% endhint %}

***

## GENInpaint

returns [`GeneratedImage`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedImage.html)

Edits an existing image using a prompt. Ideal for adding, removing, or changing parts of an image.

#### Basic Usage

```csharp
// The given image is used as a base, and the model performs localized edits.

Texture2D result = await myTexture
    .GENInpaint("Add a crown on the cat’s head")
    .SetModel(ImageModel.DallE2)
    .ExecuteAsync();
    
// You can provide a mask using InpaintPrompt if you need precise region control.
```

{% hint style="warning" %}
DALL·E 3 **does not** supports inpaint.
{% endhint %}

***

## GENVariation

returns [`GeneratedImage`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedImage.html)

Creates new stylistic variants of an existing image. This is useful for generating concept variations or alternatives.

{% hint style="warning" %}
Only **DALL·E 2** currently supports image variation.
{% endhint %}

#### Basic Usage

```csharp
Texture2D result = await myTexture
    .GENImageVariation()
    .SetModel(OpenAIModel.DallE2)
    .ExecuteAsync();
```
