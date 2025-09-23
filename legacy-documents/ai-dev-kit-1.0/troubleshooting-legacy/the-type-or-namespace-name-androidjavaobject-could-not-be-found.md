# ❗The type or namespace name 'AndroidJavaObject' could not be found

{% hint style="danger" %}
E**rror Details:**

Assets\Glitch9\CoreLib\System\Common\Utils\Vibrator.cs(27,24): error CS0246: The type or namespace name 'AndroidJavaObject' could not be found (are you missing a using directive or an assembly reference?)
{% endhint %}

{% hint style="success" %}
**Solution:**

Remove the `Vibrator.cs` file from the `Assets\Glitch9\CoreLib\System\Common\Utils\` directory. This file references unavailable Android-specific classes and is not essential for the asset's current functionality.
{% endhint %}
