# \[IL2CPP] Cannot Convert 'Nullable\<Enum>' to 'int' During Android Build

### **Issue**

{% hint style="danger" %}
When building for Android with IL2CPP, the following error may occur:
{% endhint %}

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### **Possible Cause**

{% hint style="info" %}
* This happens when calling `.HasFlag()` on a `Nullable<Enum>`.
* This is a known issue in **Unity 2021**, where `Enum.HasFlag()` does not compile correctly under IL2CPP when used with nullable enums.\
  See Unity’s official issue tracker:\
  👉 [https://issuetracker.unity3d.com/issues/il2cpp-fails-code-conversion-for-enum-dot-hasflag-case](https://issuetracker.unity3d.com/issues/il2cpp-fails-code-conversion-for-enum-dot-hasflag-case)
{% endhint %}

### **How to Fix**

{% hint style="success" %}
* Upgrade your Unity version to **2022.1 or later**, where this bug has been fixed.
{% endhint %}

{% hint style="info" %}
If the issue persists even with the latest version, consider joining [Discord](https://discord.gg/hgajxPpJYf) — where I provide **preview builds**, **hotfixes**, and direct support.&#x20;

🔒 **Gain** **Access to the PRO preview builds channel.**\
To gain access, please send me your **proof of purchase** (e.g. invoice or order number).
{% endhint %}
