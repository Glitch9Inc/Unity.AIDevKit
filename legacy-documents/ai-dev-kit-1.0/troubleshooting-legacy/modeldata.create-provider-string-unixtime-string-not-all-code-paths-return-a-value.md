# ❗'ModelData.Create(Provider, string, UnixTime?, string)': not all code paths return a value

{% hint style="danger" %}
**Error Details:**

You might encounter this error when you build your project.

Build Error: \[16:45:38] Assets\Glitch9\AIDevKit\Core\Runtime\ScriptableObjects\ModelData.cs(152,33): error CS0161: 'ModelData.Create(Provider, string, UnixTime?, string)': not all code paths return a value
{% endhint %}

{% hint style="success" %}
**Solution:**

Find `ModelData.cs` in the Unity Editor and add one line like in the following screenshot.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>
