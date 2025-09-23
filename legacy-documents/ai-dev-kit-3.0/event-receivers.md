---
icon: plug
---

# Event Receivers

These components let you connect UnityEvents to chatbot behavior — like receiving a message, handling errors, or reacting to streaming responses. Add them to any GameObject and assign events via the Inspector.&#x20;

***

### 🧩 How to Use Receivers

1. **Add the Receiver Component**\
   Select a GameObject in your scene and click **Add Component**.\
   Then choose one of the receiver types, such as `ChatTextEventReceiver` , `ErrorMessageReceiver`, etc.
2. **Assign UnityEvents**\
   In the Inspector, the receiver will expose several UnityEvent fields like **On Message Created** or **On Assistant Created**.
   * Click the ➕ button next to the event you want to use.
   * Drag a target GameObject that has the script containing your response method.
   * Select the method you want to call from the dropdown (e.g., `ShowMessage(string)` or `Log()`).
3. **Register the Receiver in Your AI Component**\
   Receivers do nothing on their own—they need to be **linked to your AI component** (like [`Chatbot`](../../unity-components/ai-agents/chatbot-chat-completion-api.md)).
   * Find the field labeled `Chat Event Receiver`, `Error Receiver`, etc. in your AI component.
   * Assign the corresponding Receiver component you added earlier.
4. **Test It**\
   Enter Play mode and trigger an AI interaction. If everything is wired correctly, the methods you assigned in UnityEvents will be called when those events occur.

