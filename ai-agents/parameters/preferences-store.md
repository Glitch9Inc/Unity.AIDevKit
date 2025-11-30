# Preferences Store

Persist user preferences for agent configuration.

## Overview

The Preferences Store allows you to:

- Save user's preferred settings
- Restore settings across sessions
- Sync preferences across devices (optional)
- Provide personalized experiences

## Basic Usage

### Save Preferences

```csharp
// Save individual preferences
PlayerPrefs.SetString("PreferredModel", agent.Model.Id);
PlayerPrefs.SetString("PreferredVoice", agent.Voice.Id);
PlayerPrefs.SetFloat("Temperature", agent.Temperature ?? 0.7f);
PlayerPrefs.SetInt("MaxTokens", agent.MaxTokens ?? 1000);
PlayerPrefs.Save();
```

### Load Preferences

```csharp
void Start()
{
    // Load saved preferences
    string model = PlayerPrefs.GetString("PreferredModel", "gpt-4o");
    string voice = PlayerPrefs.GetString("PreferredVoice", "alloy");
    float temperature = PlayerPrefs.GetFloat("Temperature", 0.7f);
    int maxTokens = PlayerPrefs.GetInt("MaxTokens", 1000);
    
    // Apply to agent
    agent.Model = new Model(model);
    agent.Voice = new Voice(voice);
    agent.Temperature = temperature;
    agent.MaxTokens = maxTokens;
}
```

## Preference Manager

### Complete Manager Class

```csharp
[System.Serializable]
public class AgentPreferences
{
    public string Model = "gpt-4o";
    public string Voice = "alloy";
    public float Temperature = 0.7f;
    public int MaxTokens = 1000;
    public float SpeechSpeed = 1.0f;
    public bool EnableStreaming = true;
    public bool EnableAudio = true;
    public ConversationStoreType ConversationStore = ConversationStoreType.LocalFile;
}

public class PreferencesManager : MonoBehaviour
{
    private const string PREFS_KEY = "AgentPreferences";
    
    public static void Save(AgentPreferences prefs)
    {
        string json = JsonUtility.ToJson(prefs);
        PlayerPrefs.SetString(PREFS_KEY, json);
        PlayerPrefs.Save();
        
        Debug.Log("✓ Preferences saved");
    }
    
    public static AgentPreferences Load()
    {
        if (PlayerPrefs.HasKey(PREFS_KEY))
        {
            string json = PlayerPrefs.GetString(PREFS_KEY);
            return JsonUtility.FromJson<AgentPreferences>(json);
        }
        
        return new AgentPreferences(); // Defaults
    }
    
    public static void ApplyToAgent(AgentBehaviour agent, AgentPreferences prefs)
    {
        agent.Model = new Model(prefs.Model);
        agent.Voice = new Voice(prefs.Voice);
        agent.Temperature = prefs.Temperature;
        agent.MaxTokens = prefs.MaxTokens;
        agent.Stream = prefs.EnableStreaming;
        agent.EnableOutputAudio = prefs.EnableAudio;
        agent.ConversationStore = prefs.ConversationStore;
        
        if (agent.Settings.OutputAudioParameters != null)
        {
            agent.Settings.OutputAudioParameters.Speed = prefs.SpeechSpeed;
        }
        
        Debug.Log("✓ Preferences applied to agent");
    }
    
    public static AgentPreferences GetFromAgent(AgentBehaviour agent)
    {
        return new AgentPreferences
        {
            Model = agent.Model.Id,
            Voice = agent.Voice.Id,
            Temperature = agent.Temperature ?? 0.7f,
            MaxTokens = agent.MaxTokens ?? 1000,
            SpeechSpeed = agent.Settings.OutputAudioParameters?.Speed ?? 1.0f,
            EnableStreaming = agent.Stream,
            EnableAudio = agent.EnableOutputAudio,
            ConversationStore = agent.ConversationStore
        };
    }
}
```

### Usage

```csharp
public class AgentSetup : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Load and apply preferences
        var prefs = PreferencesManager.Load();
        PreferencesManager.ApplyToAgent(agent, prefs);
    }
    
    void OnApplicationQuit()
    {
        // Save current settings
        var prefs = PreferencesManager.GetFromAgent(agent);
        PreferencesManager.Save(prefs);
    }
}
```

## Settings UI

### Preferences Panel

```csharp
public class PreferencesPanel : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("UI")]
    [SerializeField] private TMP_Dropdown modelDropdown;
    [SerializeField] private TMP_Dropdown voiceDropdown;
    [SerializeField] private Slider temperatureSlider;
    [SerializeField] private TMP_InputField maxTokensInput;
    [SerializeField] private Toggle streamingToggle;
    [SerializeField] private Toggle audioToggle;
    [SerializeField] private Button saveButton;
    [SerializeField] private Button resetButton;
    
    void Start()
    {
        LoadCurrentSettings();
        
        saveButton.onClick.AddListener(SavePreferences);
        resetButton.onClick.AddListener(ResetToDefaults);
    }
    
    void LoadCurrentSettings()
    {
        var prefs = PreferencesManager.Load();
        
        // Populate dropdowns
        SetDropdownValue(modelDropdown, prefs.Model);
        SetDropdownValue(voiceDropdown, prefs.Voice);
        
        // Set values
        temperatureSlider.value = prefs.Temperature;
        maxTokensInput.text = prefs.MaxTokens.ToString();
        streamingToggle.isOn = prefs.EnableStreaming;
        audioToggle.isOn = prefs.EnableAudio;
    }
    
    void SavePreferences()
    {
        var prefs = new AgentPreferences
        {
            Model = GetDropdownValue(modelDropdown),
            Voice = GetDropdownValue(voiceDropdown),
            Temperature = temperatureSlider.value,
            MaxTokens = int.Parse(maxTokensInput.text),
            EnableStreaming = streamingToggle.isOn,
            EnableAudio = audioToggle.isOn
        };
        
        PreferencesManager.Save(prefs);
        PreferencesManager.ApplyToAgent(agent, prefs);
        
        ShowMessage("Preferences saved!");
    }
    
    void ResetToDefaults()
    {
        var defaults = new AgentPreferences();
        PreferencesManager.Save(defaults);
        PreferencesManager.ApplyToAgent(agent, defaults);
        LoadCurrentSettings();
        
        ShowMessage("Reset to defaults");
    }
    
    void SetDropdownValue(TMP_Dropdown dropdown, string value)
    {
        for (int i = 0; i < dropdown.options.Count; i++)
        {
            if (dropdown.options[i].text == value)
            {
                dropdown.value = i;
                break;
            }
        }
    }
    
    string GetDropdownValue(TMP_Dropdown dropdown)
    {
        return dropdown.options[dropdown.value].text;
    }
    
    void ShowMessage(string message)
    {
        Debug.Log(message);
        // Show toast/notification
    }
}
```

## Cloud Sync (Optional)

### Save to Cloud

```csharp
public class CloudPreferences : MonoBehaviour
{
    private const string CLOUD_ENDPOINT = "https://api.yourapp.com/preferences";
    
    public async UniTask SaveToCloud(string userId, AgentPreferences prefs)
    {
        string json = JsonUtility.ToJson(prefs);
        
        using (UnityWebRequest request = UnityWebRequest.Post(CLOUD_ENDPOINT, ""))
        {
            byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(json);
            request.uploadHandler = new UploadHandlerRaw(bodyRaw);
            request.downloadHandler = new DownloadHandlerBuffer();
            request.SetRequestHeader("Content-Type", "application/json");
            request.SetRequestHeader("User-Id", userId);
            
            await request.SendWebRequest();
            
            if (request.result == UnityWebRequest.Result.Success)
            {
                Debug.Log("✓ Preferences saved to cloud");
            }
            else
            {
                Debug.LogError($"Failed to save to cloud: {request.error}");
            }
        }
    }
    
    public async UniTask<AgentPreferences> LoadFromCloud(string userId)
    {
        using (UnityWebRequest request = UnityWebRequest.Get($"{CLOUD_ENDPOINT}?userId={userId}"))
        {
            await request.SendWebRequest();
            
            if (request.result == UnityWebRequest.Result.Success)
            {
                string json = request.downloadHandler.text;
                return JsonUtility.FromJson<AgentPreferences>(json);
            }
            else
            {
                Debug.LogWarning($"Failed to load from cloud: {request.error}");
                return new AgentPreferences();
            }
        }
    }
}
```

### Sync Manager

```csharp
public class PreferencesSyncManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private bool enableCloudSync = false;
    
    private CloudPreferences cloudPrefs;
    private string userId;
    
    void Start()
    {
        cloudPrefs = new CloudPreferences();
        userId = GetUserId();
        
        LoadPreferences();
    }
    
    async void LoadPreferences()
    {
        AgentPreferences prefs;
        
        if (enableCloudSync)
        {
            // Try cloud first
            prefs = await cloudPrefs.LoadFromCloud(userId);
            
            if (prefs == null)
            {
                // Fallback to local
                prefs = PreferencesManager.Load();
            }
        }
        else
        {
            // Local only
            prefs = PreferencesManager.Load();
        }
        
        PreferencesManager.ApplyToAgent(agent, prefs);
    }
    
    public async void SavePreferences()
    {
        var prefs = PreferencesManager.GetFromAgent(agent);
        
        // Always save locally
        PreferencesManager.Save(prefs);
        
        // Optionally sync to cloud
        if (enableCloudSync)
        {
            await cloudPrefs.SaveToCloud(userId, prefs);
        }
    }
    
    string GetUserId()
    {
        string userId = PlayerPrefs.GetString("UserId");
        if (string.IsNullOrEmpty(userId))
        {
            userId = System.Guid.NewGuid().ToString();
            PlayerPrefs.SetString("UserId", userId);
        }
        return userId;
    }
}
```

## Profile System

### Multiple Profiles

```csharp
public class ProfileManager : MonoBehaviour
{
    [System.Serializable]
    public class Profile
    {
        public string Name;
        public AgentPreferences Preferences;
    }
    
    private List<Profile> profiles = new List<Profile>();
    private string currentProfileName;
    
    public void CreateProfile(string profileName)
    {
        var profile = new Profile
        {
            Name = profileName,
            Preferences = new AgentPreferences()
        };
        
        profiles.Add(profile);
        SaveProfiles();
    }
    
    public void SwitchProfile(string profileName)
    {
        var profile = profiles.Find(p => p.Name == profileName);
        if (profile != null)
        {
            currentProfileName = profileName;
            PreferencesManager.ApplyToAgent(agent, profile.Preferences);
            PlayerPrefs.SetString("CurrentProfile", profileName);
        }
    }
    
    public void SaveCurrentProfile()
    {
        var profile = profiles.Find(p => p.Name == currentProfileName);
        if (profile != null)
        {
            profile.Preferences = PreferencesManager.GetFromAgent(agent);
            SaveProfiles();
        }
    }
    
    void SaveProfiles()
    {
        string json = JsonUtility.ToJson(new ProfileList { profiles = profiles });
        PlayerPrefs.SetString("Profiles", json);
    }
    
    void LoadProfiles()
    {
        if (PlayerPrefs.HasKey("Profiles"))
        {
            string json = PlayerPrefs.GetString("Profiles");
            profiles = JsonUtility.FromJson<ProfileList>(json).profiles;
        }
    }
    
    [System.Serializable]
    class ProfileList
    {
        public List<Profile> profiles;
    }
}
```

## Best Practices

### 1. Auto-Save on Change

```csharp
public class AutoSavePreferences : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private float saveDelay = 1f;
    
    private float lastChangeTime;
    private bool hasUnsavedChanges;
    
    void Update()
    {
        if (hasUnsavedChanges && Time.time - lastChangeTime >= saveDelay)
        {
            SavePreferences();
            hasUnsavedChanges = false;
        }
    }
    
    public void OnSettingChanged()
    {
        lastChangeTime = Time.time;
        hasUnsavedChanges = true;
    }
    
    void SavePreferences()
    {
        var prefs = PreferencesManager.GetFromAgent(agent);
        PreferencesManager.Save(prefs);
        Debug.Log("Auto-saved preferences");
    }
}
```

### 2. Validate Preferences

```csharp
public bool ValidatePreferences(AgentPreferences prefs)
{
    // Validate model
    string[] validModels = { "gpt-4o", "gpt-4-turbo", "gpt-3.5-turbo" };
    if (!validModels.Contains(prefs.Model))
    {
        Debug.LogWarning($"Invalid model: {prefs.Model}, using default");
        prefs.Model = "gpt-4o";
    }
    
    // Validate temperature
    if (prefs.Temperature < 0f || prefs.Temperature > 2f)
    {
        prefs.Temperature = Mathf.Clamp(prefs.Temperature, 0f, 2f);
    }
    
    // Validate max tokens
    if (prefs.MaxTokens < 1 || prefs.MaxTokens > 4096)
    {
        prefs.MaxTokens = Mathf.Clamp(prefs.MaxTokens, 1, 4096);
    }
    
    return true;
}
```

### 3. Provide Reset Option

```csharp
public void ResetAllPreferences()
{
    if (ShowConfirmDialog("Reset all preferences to default?"))
    {
        PlayerPrefs.DeleteAll();
        var defaults = new AgentPreferences();
        PreferencesManager.Save(defaults);
        PreferencesManager.ApplyToAgent(agent, defaults);
        
        Debug.Log("✓ All preferences reset");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class CompletePreferencesSystem : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        LoadPreferences();
    }
    
    void LoadPreferences()
    {
        var prefs = PreferencesManager.Load();
        
        // Validate before applying
        ValidatePreferences(prefs);
        
        // Apply to agent
        PreferencesManager.ApplyToAgent(agent, prefs);
        
        Debug.Log("✓ Preferences loaded");
        Debug.Log($"  Model: {prefs.Model}");
        Debug.Log($"  Voice: {prefs.Voice}");
        Debug.Log($"  Temperature: {prefs.Temperature}");
    }
    
    public void SaveCurrentSettings()
    {
        var prefs = PreferencesManager.GetFromAgent(agent);
        PreferencesManager.Save(prefs);
        
        Debug.Log("✓ Preferences saved");
    }
    
    void OnApplicationQuit()
    {
        // Auto-save on quit
        SaveCurrentSettings();
    }
    
    void ValidatePreferences(AgentPreferences prefs)
    {
        prefs.Temperature = Mathf.Clamp(prefs.Temperature, 0f, 2f);
        prefs.MaxTokens = Mathf.Clamp(prefs.MaxTokens, 1, 4096);
        prefs.SpeechSpeed = Mathf.Clamp(prefs.SpeechSpeed, 0.25f, 4f);
    }
}
```

## Next Steps

- [Models](models.md)
- [Voice](voice.md)
- [Temperature & Max Tokens](temperature-tokens.md)
- [Agent Settings](../configuration/agent-settings.md)
