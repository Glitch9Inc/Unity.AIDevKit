# Computer Use Tool Calls

Allow agents to interact with the computer through UI automation.

## Overview

Computer Use enables agents to:

- Simulate mouse clicks
- Type keyboard input
- Capture screenshots
- Navigate UI elements
- Automate interactions

⚠️ **Warning**: Computer Use provides powerful system access. Use with caution and proper permissions.

## Basic Setup

### Enable Computer Use

```csharp
// Add computer use tool
agent.AddTool(ToolType.ComputerUse);

// Configure permissions
agent.Settings.ComputerUse = new ComputerUseSettings
{
    AllowMouseControl = true,
    AllowKeyboardControl = true,
    AllowScreenshots = true
};
```

## Mouse Control

### Mouse Operations

```csharp
public class MouseControl : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("mouse_click", PerformClick);
        agent.AddTool("mouse_move", MoveMouse);
        agent.AddTool("mouse_drag", DragMouse);
    }
    
    string PerformClick(int x, int y, string button = "left")
    {
        try
        {
            // Move to position
            SetCursorPos(x, y);
            
            // Click
            uint mouseButton = button.ToLower() switch
            {
                "left" => MOUSEEVENTF_LEFTDOWN | MOUSEEVENTF_LEFTUP,
                "right" => MOUSEEVENTF_RIGHTDOWN | MOUSEEVENTF_RIGHTUP,
                "middle" => MOUSEEVENTF_MIDDLEDOWN | MOUSEEVENTF_MIDDLEUP,
                _ => MOUSEEVENTF_LEFTDOWN | MOUSEEVENTF_LEFTUP
            };
            
            mouse_event(mouseButton, 0, 0, 0, 0);
            
            return $"Clicked {button} button at ({x}, {y})";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string MoveMouse(int x, int y)
    {
        SetCursorPos(x, y);
        return $"Moved mouse to ({x}, {y})";
    }
    
    string DragMouse(int startX, int startY, int endX, int endY)
    {
        // Move to start
        SetCursorPos(startX, startY);
        
        // Press button
        mouse_event(MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0);
        
        // Drag to end
        SetCursorPos(endX, endY);
        
        // Release button
        mouse_event(MOUSEEVENTF_LEFTUP, 0, 0, 0, 0);
        
        return $"Dragged from ({startX}, {startY}) to ({endX}, {endY})";
    }
    
    // Windows API
    [System.Runtime.InteropServices.DllImport("user32.dll")]
    static extern bool SetCursorPos(int x, int y);
    
    [System.Runtime.InteropServices.DllImport("user32.dll")]
    static extern void mouse_event(uint dwFlags, int dx, int dy, uint dwData, int dwExtraInfo);
    
    const uint MOUSEEVENTF_LEFTDOWN = 0x0002;
    const uint MOUSEEVENTF_LEFTUP = 0x0004;
    const uint MOUSEEVENTF_RIGHTDOWN = 0x0008;
    const uint MOUSEEVENTF_RIGHTUP = 0x0010;
    const uint MOUSEEVENTF_MIDDLEDOWN = 0x0020;
    const uint MOUSEEVENTF_MIDDLEUP = 0x0040;
}
```

## Keyboard Control

### Keyboard Input

```csharp
public class KeyboardControl : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("type_text", TypeText);
        agent.AddTool("press_key", PressKey);
        agent.AddTool("key_combination", KeyCombination);
    }
    
    string TypeText(string text)
    {
        try
        {
            foreach (char c in text)
            {
                SendKey(c);
                Thread.Sleep(10); // Small delay between keys
            }
            
            return $"Typed: {text}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string PressKey(string key)
    {
        try
        {
            KeyCode keyCode = ParseKeyCode(key);
            SimulateKeyPress(keyCode);
            
            return $"Pressed key: {key}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string KeyCombination(string modifier, string key)
    {
        try
        {
            KeyCode modifierKey = ParseKeyCode(modifier);
            KeyCode mainKey = ParseKeyCode(key);
            
            // Press modifier
            SimulateKeyDown(modifierKey);
            
            // Press main key
            SimulateKeyPress(mainKey);
            
            // Release modifier
            SimulateKeyUp(modifierKey);
            
            return $"Pressed {modifier}+{key}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    void SendKey(char c)
    {
        // Implementation depends on platform
        // Windows: SendKeys, user32.dll
        // Mac: CGEventPost
        // Linux: xdotool
    }
    
    KeyCode ParseKeyCode(string key)
    {
        return key.ToLower() switch
        {
            "enter" => KeyCode.Return,
            "space" => KeyCode.Space,
            "tab" => KeyCode.Tab,
            "esc" or "escape" => KeyCode.Escape,
            "ctrl" or "control" => KeyCode.LeftControl,
            "shift" => KeyCode.LeftShift,
            "alt" => KeyCode.LeftAlt,
            _ => (KeyCode)System.Enum.Parse(typeof(KeyCode), key, true)
        };
    }
    
    void SimulateKeyPress(KeyCode key) { /* Implementation */ }
    void SimulateKeyDown(KeyCode key) { /* Implementation */ }
    void SimulateKeyUp(KeyCode key) { /* Implementation */ }
}
```

## Screenshot Capture

### Screen Capture

```csharp
public class ScreenCapture : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string screenshotPath = "Screenshots";
    
    void Start()
    {
        agent.AddTool("take_screenshot", TakeScreenshot);
        agent.AddTool("capture_region", CaptureRegion);
    }
    
    string TakeScreenshot()
    {
        try
        {
            string fileName = $"screenshot_{DateTime.Now:yyyyMMdd_HHmmss}.png";
            string fullPath = Path.Combine(screenshotPath, fileName);
            
            // Ensure directory exists
            Directory.CreateDirectory(screenshotPath);
            
            // Capture
            UnityEngine.ScreenCapture.CaptureScreenshot(fullPath);
            
            return $"Screenshot saved: {fullPath}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string CaptureRegion(int x, int y, int width, int height)
    {
        try
        {
            // Create texture for region
            Texture2D screenshot = new Texture2D(width, height, TextureFormat.RGB24, false);
            
            // Read pixels from screen
            screenshot.ReadPixels(new Rect(x, y, width, height), 0, 0);
            screenshot.Apply();
            
            // Save
            byte[] bytes = screenshot.EncodeToPNG();
            string fileName = $"region_{DateTime.Now:yyyyMMdd_HHmmss}.png";
            string fullPath = Path.Combine(screenshotPath, fileName);
            
            File.WriteAllBytes(fullPath, bytes);
            
            Destroy(screenshot);
            
            return $"Region captured: {fullPath}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
}
```

## UI Element Detection

### Find UI Elements

```csharp
public class UIDetection : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("find_button", FindButton);
        agent.AddTool("find_element", FindElement);
        agent.AddTool("get_element_info", GetElementInfo);
    }
    
    string FindButton(string buttonText)
    {
        var buttons = FindObjectsOfType<Button>();
        
        foreach (var button in buttons)
        {
            var text = button.GetComponentInChildren<TMP_Text>();
            if (text != null && text.text.Contains(buttonText))
            {
                var pos = RectTransformUtility.WorldToScreenPoint(
                    Camera.main,
                    button.transform.position
                );
                
                return $"Found button at ({pos.x}, {pos.y})";
            }
        }
        
        return $"Button with text '{buttonText}' not found";
    }
    
    string FindElement(string elementName)
    {
        var element = GameObject.Find(elementName);
        
        if (element == null)
        {
            return $"Element '{elementName}' not found";
        }
        
        var rectTransform = element.GetComponent<RectTransform>();
        if (rectTransform != null)
        {
            var pos = RectTransformUtility.WorldToScreenPoint(
                Camera.main,
                rectTransform.position
            );
            
            return JsonUtility.ToJson(new
            {
                name = element.name,
                position = pos,
                size = rectTransform.sizeDelta,
                active = element.activeSelf
            });
        }
        
        return $"Element found but no RectTransform";
    }
    
    string GetElementInfo(string elementName)
    {
        var element = GameObject.Find(elementName);
        
        if (element == null)
        {
            return $"Element '{elementName}' not found";
        }
        
        var info = new
        {
            name = element.name,
            active = element.activeSelf,
            layer = LayerMask.LayerToName(element.layer),
            tag = element.tag,
            components = element.GetComponents<Component>()
                .Select(c => c.GetType().Name)
                .ToArray()
        };
        
        return JsonUtility.ToJson(info);
    }
}
```

## Automation Workflows

### Multi-Step Actions

```csharp
public class AutomationWorkflow : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("fill_form", FillForm);
        agent.AddTool("login_sequence", LoginSequence);
    }
    
    async UniTask<string> FillForm(string formData)
    {
        try
        {
            var data = JsonUtility.FromJson<FormData>(formData);
            
            // Click first field
            await ClickElement("NameField");
            await UniTask.Delay(100);
            
            // Type name
            TypeText(data.name);
            await UniTask.Delay(100);
            
            // Tab to next field
            PressKey("Tab");
            await UniTask.Delay(100);
            
            // Type email
            TypeText(data.email);
            await UniTask.Delay(100);
            
            // Click submit
            await ClickElement("SubmitButton");
            
            return "Form filled successfully";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    async UniTask<string> LoginSequence(string username, string password)
    {
        try
        {
            // Click username field
            await ClickElement("UsernameField");
            await UniTask.Delay(100);
            
            // Type username
            TypeText(username);
            await UniTask.Delay(100);
            
            // Tab to password
            PressKey("Tab");
            await UniTask.Delay(100);
            
            // Type password
            TypeText(password);
            await UniTask.Delay(100);
            
            // Press Enter
            PressKey("Enter");
            
            return "Login sequence completed";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    async UniTask ClickElement(string elementName)
    {
        var element = GameObject.Find(elementName);
        if (element != null)
        {
            var button = element.GetComponent<Button>();
            button?.onClick.Invoke();
        }
        await UniTask.Yield();
    }
    
    void TypeText(string text) { /* Implementation */ }
    void PressKey(string key) { /* Implementation */ }
    
    [System.Serializable]
    class FormData
    {
        public string name;
        public string email;
    }
}
```

## Safety & Permissions

### Permission System

```csharp
public class ComputerUsePermissions : MonoBehaviour
{
    [SerializeField] private bool allowMouseControl = false;
    [SerializeField] private bool allowKeyboardControl = false;
    [SerializeField] private bool allowScreenshots = true;
    [SerializeField] private bool requireConfirmation = true;
    
    public bool CanPerformAction(string actionType)
    {
        return actionType switch
        {
            "mouse" => allowMouseControl,
            "keyboard" => allowKeyboardControl,
            "screenshot" => allowScreenshots,
            _ => false
        };
    }
    
    public async UniTask<bool> RequestPermission(string action)
    {
        if (!requireConfirmation)
            return true;
        
        // Show permission dialog
        Debug.Log($"🔐 Permission requested: {action}");
        
        // Wait for user response
        await UniTask.Delay(100);
        
        return true; // For example
    }
}
```

## Best Practices

### 1. Always Validate Actions

```csharp
async UniTask<string> SafeExecute(string action, Func<UniTask<string>> execute)
{
    // Check permissions
    if (!permissions.CanPerformAction(action))
    {
        return "Error: Permission denied";
    }
    
    // Request confirmation if needed
    if (!await permissions.RequestPermission(action))
    {
        return "Error: User denied permission";
    }
    
    // Execute
    try
    {
        return await execute();
    }
    catch (Exception ex)
    {
        return $"Error: {ex.Message}";
    }
}
```

### 2. Add Delays

```csharp
// Allow UI to respond
await UniTask.Delay(100);

// Between keystrokes
await UniTask.Delay(50);
```

### 3. Handle Failures

```csharp
int retries = 3;
for (int i = 0; i < retries; i++)
{
    try
    {
        return await PerformAction();
    }
    catch (Exception ex)
    {
        if (i == retries - 1) throw;
        await UniTask.Delay(1000);
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ComputerUseManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private ComputerUsePermissions permissions;
    
    void Start()
    {
        RegisterTools();
    }
    
    void RegisterTools()
    {
        agent.AddTool("click", SafeClick);
        agent.AddTool("type", SafeType);
        agent.AddTool("screenshot", SafeScreenshot);
        agent.AddTool("find_and_click", FindAndClick);
    }
    
    async UniTask<string> SafeClick(int x, int y)
    {
        if (!permissions.CanPerformAction("mouse"))
        {
            return "Error: Mouse control not allowed";
        }
        
        if (!await permissions.RequestPermission("click"))
        {
            return "Error: Permission denied";
        }
        
        try
        {
            PerformClick(x, y);
            return $"Clicked at ({x}, {y})";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    async UniTask<string> SafeType(string text)
    {
        if (!permissions.CanPerformAction("keyboard"))
        {
            return "Error: Keyboard control not allowed";
        }
        
        if (!await permissions.RequestPermission("type"))
        {
            return "Error: Permission denied";
        }
        
        try
        {
            foreach (char c in text)
            {
                TypeCharacter(c);
                await UniTask.Delay(50);
            }
            
            return $"Typed: {text}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string SafeScreenshot()
    {
        if (!permissions.CanPerformAction("screenshot"))
        {
            return "Error: Screenshots not allowed";
        }
        
        try
        {
            string path = $"screenshot_{DateTime.Now:yyyyMMdd_HHmmss}.png";
            UnityEngine.ScreenCapture.CaptureScreenshot(path);
            return $"Screenshot: {path}";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    async UniTask<string> FindAndClick(string elementName)
    {
        if (!permissions.CanPerformAction("mouse"))
        {
            return "Error: Mouse control not allowed";
        }
        
        try
        {
            var element = GameObject.Find(elementName);
            if (element == null)
            {
                return $"Element '{elementName}' not found";
            }
            
            var button = element.GetComponent<Button>();
            if (button != null)
            {
                if (!await permissions.RequestPermission($"click {elementName}"))
                {
                    return "Error: Permission denied";
                }
                
                button.onClick.Invoke();
                return $"Clicked {elementName}";
            }
            
            return "Element is not a button";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    void PerformClick(int x, int y) { /* Implementation */ }
    void TypeCharacter(char c) { /* Implementation */ }
}
```

## Next Steps

- [Function Calls](function.md)
- [Local Shell](local-shell.md)
- [Hosted Tools](../hosted-tools/)
