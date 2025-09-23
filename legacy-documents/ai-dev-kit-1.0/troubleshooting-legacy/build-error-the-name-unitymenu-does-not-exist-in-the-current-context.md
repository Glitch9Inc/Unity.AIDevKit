# ❗Build Error: The name 'UnityMenu' does not exist in the current context

{% hint style="danger" %}
E**rror Details:**

You might encounter this error when you build your project.

Build Error: The name 'UnityMenu' does not exist in the current context\
Build Error: Build doesn’t exist in the namespace “Glitch9.Internal”
{% endhint %}

{% hint style="success" %}
Sol**ution:**

Find `ColorScheme.cs` in the Unity Editor and remove following lines.

**Remove Line 1**: using Glitch9.Internal;

**Remove Line 6**: \[CreateAssetMenu(fileName = UnityMenu.MaterialDesign.COLOR\_SCHEME\_NAME, menuName = UnityMenu.MaterialDesign.COLOR\_SCHEME\_CREATE, order = UnityMenu.MaterialDesign.COLOR\_SCHEME\_ORDER)]
{% endhint %}
