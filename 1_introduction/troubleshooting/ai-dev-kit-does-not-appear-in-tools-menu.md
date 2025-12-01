---
description: >-
  If AI Dev Kit is not visible in Unity after install, please read this
  instructions
---

# AI Dev Kit does not appear in Tools menu

### **Issue**

{% hint style="danger" %}
After importing **AI Dev Kit** into Unity, the package appears to install correctly, but the `AI Dev Kit` menu does **not appear under `Tools`**, and the package seems non-functional.
{% endhint %}

### **Possible Cause**

{% hint style="info" %}
In rare cases, Unity fails to automatically apply the required **Scripting Define Symbol** (`UNITY_NEWTONSOFT_JSON, CYSHARP_UNITASK`) during installation. This prevents the editor scripts from activating, resulting in the AI Dev Kit menu not appearing.
{% endhint %}

### **How to Fix**

{% hint style="success" %}
* Open **File > Build Settings >** **Project Settings > Player > Other Settings**
* Locate the **Scripting Define Symbols** field for your current build target
*   Manually add these two symbols:

    ```
    UNITY_NEWTONSOFT_JSON
    CYSHARP_UNITASK
    ```
* Press Enter or click elsewhere to confirm the change
* Unity should now recompile the scripts, and the `AI Dev Kit` menu should appear under `Tools`
{% endhint %}

### **If It Still Doesn’t Work**

{% hint style="success" %}
* Sometimes, Unity also fails to automatically install required dependencies during the package import. Make sure you have following folders in **Packages** in your project window:
  * `Newtonsoft Json`&#x20;
  * `UniTask`&#x20;
* If you are missing any of the above, manual install the dependencies.
  * **Newtonsoft.Json:** [Manual Installation Guide](https://glitch9.gitbook.io/docs/support/how-to-setup-newtonsoft.json)
  * **Cysharp.UniTask:** [Manual Installation Guide](https://glitch9.gitbook.io/docs/support/how-to-setup-unitask)
* Each guide provides step-by-step instructions for adding the packages to your Unity project and verifying that everything is set up correctly. After completing the installations, restart Unity to ensure the changes take effect.
{% endhint %}

{% hint style="info" %}
If the issue persists even with the latest version, consider joining [Discord](https://discord.gg/hgajxPpJYf) — where I provide **preview builds**, **hotfixes**, and direct support.&#x20;

🔒 **Gain** **Access to the PRO preview builds channel.**\
To gain access, please send me your **proof of purchase** (e.g. invoice or order number).
{% endhint %}
