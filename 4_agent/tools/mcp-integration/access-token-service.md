---
icon: key
---

# Access Token Service

Manage access tokens for MCP servers and external services.

## Overview

Access Token Service provides:

- Centralized token management
- Automatic token refresh
- Secure token storage
- Multi-service support
- Token expiration handling

## Basic Setup

### Initialize Token Service

```csharp
var tokenService = agent.MCPController.TokenService;

// Configure token service
tokenService.EnableAutoRefresh = true;
tokenService.RefreshThresholdSeconds = 300; // Refresh 5 minutes before expiry
```

## Token Management

### Store Access Token

```csharp
public class TokenManager : MonoBehaviour
{
    private AccessTokenService tokenService;
    
    void Start()
    {
        tokenService = agent.MCPController.TokenService;
    }
    
    public void StoreToken(string serviceId, AccessToken token)
    {
        tokenService.SetToken(serviceId, token);
        
        Debug.Log($"✓ Token stored for {serviceId}");
    }
    
    public AccessToken GetToken(string serviceId)
    {
        return tokenService.GetToken(serviceId);
    }
    
    public void RemoveToken(string serviceId)
    {
        tokenService.RemoveToken(serviceId);
        
        Debug.Log($"✓ Token removed for {serviceId}");
    }
}

[System.Serializable]
public class AccessToken
{
    public string Token;
    public string RefreshToken;
    public long ExpiresAt;
    public string[] Scopes;
    public string TokenType; // Bearer, etc.
}
```

### Check Token Validity

```csharp
public bool IsTokenValid(string serviceId)
{
    var token = tokenService.GetToken(serviceId);
    
    if (token == null)
        return false;
    
    long now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
    
    // Check if token has expired
    if (token.ExpiresAt <= now)
    {
        Debug.LogWarning($"Token expired for {serviceId}");
        return false;
    }
    
    // Check if token expires soon (within threshold)
    long threshold = tokenService.RefreshThresholdSeconds;
    if (token.ExpiresAt - now < threshold)
    {
        Debug.Log($"Token expiring soon for {serviceId}");
        RefreshToken(serviceId);
    }
    
    return true;
}
```

## Token Refresh

### Manual Refresh

```csharp
public async UniTask<bool> RefreshToken(string serviceId)
{
    var token = tokenService.GetToken(serviceId);
    
    if (token == null || string.IsNullOrEmpty(token.RefreshToken))
    {
        Debug.LogError($"No refresh token available for {serviceId}");
        return false;
    }
    
    try
    {
        Debug.Log($"🔄 Refreshing token for {serviceId}...");
        
        var newToken = await tokenService.RefreshTokenAsync(serviceId);
        
        Debug.Log($"✓ Token refreshed for {serviceId}");
        return true;
    }
    catch (Exception ex)
    {
        Debug.LogError($"Token refresh failed: {ex.Message}");
        return false;
    }
}
```

### Automatic Refresh

```csharp
public class AutoTokenRefresher : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int checkIntervalSeconds = 60;
    
    private AccessTokenService tokenService;
    private bool isRunning;
    
    async void Start()
    {
        tokenService = agent.MCPController.TokenService;
        tokenService.EnableAutoRefresh = true;
        
        await MonitorTokens();
    }
    
    async UniTask MonitorTokens()
    {
        isRunning = true;
        
        while (isRunning)
        {
            await CheckAndRefreshTokens();
            await UniTask.Delay(checkIntervalSeconds * 1000);
        }
    }
    
    async UniTask CheckAndRefreshTokens()
    {
        var services = tokenService.GetAllServices();
        
        foreach (var serviceId in services)
        {
            var token = tokenService.GetToken(serviceId);
            
            if (token == null)
                continue;
            
            long now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
            long threshold = tokenService.RefreshThresholdSeconds;
            
            // Check if refresh needed
            if (token.ExpiresAt - now < threshold)
            {
                Debug.Log($"Refreshing token for {serviceId}");
                await tokenService.RefreshTokenAsync(serviceId);
            }
        }
    }
    
    void OnDestroy()
    {
        isRunning = false;
    }
}
```

## Token Storage

### Secure Storage

```csharp
public class SecureTokenStorage : MonoBehaviour
{
    private const string ENCRYPTION_KEY = "your-encryption-key";
    
    public void SaveTokenSecurely(string serviceId, AccessToken token)
    {
        // Serialize token
        string json = JsonUtility.ToJson(token);
        
        // Encrypt
        string encrypted = EncryptToken(json);
        
        // Store
        PlayerPrefs.SetString($"token_{serviceId}", encrypted);
        PlayerPrefs.Save();
        
        Debug.Log($"✓ Token saved securely for {serviceId}");
    }
    
    public AccessToken LoadTokenSecurely(string serviceId)
    {
        string key = $"token_{serviceId}";
        
        if (!PlayerPrefs.HasKey(key))
            return null;
        
        // Load
        string encrypted = PlayerPrefs.GetString(key);
        
        // Decrypt
        string json = DecryptToken(encrypted);
        
        // Deserialize
        return JsonUtility.FromJson<AccessToken>(json);
    }
    
    public void DeleteToken(string serviceId)
    {
        PlayerPrefs.DeleteKey($"token_{serviceId}");
        PlayerPrefs.Save();
        
        Debug.Log($"✓ Token deleted for {serviceId}");
    }
    
    string EncryptToken(string plainText)
    {
        // Implement proper encryption (e.g., AES)
        // This is a simplified example
        byte[] bytes = System.Text.Encoding.UTF8.GetBytes(plainText);
        return System.Convert.ToBase64String(bytes);
    }
    
    string DecryptToken(string cipherText)
    {
        // Implement proper decryption
        byte[] bytes = System.Convert.FromBase64String(cipherText);
        return System.Text.Encoding.UTF8.GetString(bytes);
    }
}
```

### Cloud Storage

```csharp
public class CloudTokenStorage : MonoBehaviour
{
    [SerializeField] private string cloudEndpoint = "https://api.example.com/tokens";
    
    public async UniTask SaveToCloud(string userId, string serviceId, AccessToken token)
    {
        var data = new
        {
            userId,
            serviceId,
            token = JsonUtility.ToJson(token)
        };
        
        string json = JsonUtility.ToJson(data);
        
        using var request = UnityWebRequest.Put(cloudEndpoint, json);
        request.SetRequestHeader("Content-Type", "application/json");
        
        await request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log($"✓ Token saved to cloud for {serviceId}");
        }
        else
        {
            Debug.LogError($"Failed to save token: {request.error}");
        }
    }
    
    public async UniTask<AccessToken> LoadFromCloud(string userId, string serviceId)
    {
        string url = $"{cloudEndpoint}?userId={userId}&serviceId={serviceId}";
        
        using var request = UnityWebRequest.Get(url);
        await request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            string json = request.downloadHandler.text;
            return JsonUtility.FromJson<AccessToken>(json);
        }
        
        Debug.LogError($"Failed to load token: {request.error}");
        return null;
    }
}
```

## Token Scopes

### Validate Scopes

```csharp
public bool ValidateTokenScopes(string serviceId, string[] requiredScopes)
{
    var token = tokenService.GetToken(serviceId);
    
    if (token == null || token.Scopes == null)
    {
        Debug.LogWarning($"No token or scopes for {serviceId}");
        return false;
    }
    
    foreach (var required in requiredScopes)
    {
        if (!token.Scopes.Contains(required))
        {
            Debug.LogWarning($"Missing required scope: {required}");
            return false;
        }
    }
    
    return true;
}
```

### Request Additional Scopes

```csharp
public async UniTask RequestAdditionalScopes(string serviceId, string[] newScopes)
{
    var token = tokenService.GetToken(serviceId);
    
    if (token == null)
    {
        Debug.LogError($"No token found for {serviceId}");
        return;
    }
    
    // Combine existing and new scopes
    var allScopes = token.Scopes.Union(newScopes).ToArray();
    
    // Request new token with additional scopes
    Debug.Log($"Requesting additional scopes for {serviceId}");
    
    // Trigger OAuth flow with new scopes
    await agent.MCPController.RequestOAuthAsync(serviceId, allScopes);
}
```

## Multi-Service Management

### Service Registry

```csharp
public class ServiceTokenRegistry : MonoBehaviour
{
    private Dictionary<string, ServiceConfig> services = new();
    
    public void RegisterService(string serviceId, ServiceConfig config)
    {
        services[serviceId] = config;
        
        Debug.Log($"✓ Service registered: {serviceId}");
    }
    
    public ServiceConfig GetServiceConfig(string serviceId)
    {
        return services.TryGetValue(serviceId, out var config) 
            ? config 
            : null;
    }
    
    public List<string> GetAllServices()
    {
        return services.Keys.ToList();
    }
}

[System.Serializable]
public class ServiceConfig
{
    public string ServiceId;
    public string Name;
    public string AuthEndpoint;
    public string TokenEndpoint;
    public string[] RequiredScopes;
    public int TokenLifetimeSeconds;
}
```

### Batch Token Operations

```csharp
public async UniTask RefreshAllTokens()
{
    var services = tokenService.GetAllServices();
    
    Debug.Log($"Refreshing tokens for {services.Count} services");
    
    foreach (var serviceId in services)
    {
        try
        {
            await tokenService.RefreshTokenAsync(serviceId);
            Debug.Log($"✓ Refreshed: {serviceId}");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Failed to refresh {serviceId}: {ex.Message}");
        }
    }
}

public void ClearAllTokens()
{
    var services = tokenService.GetAllServices();
    
    foreach (var serviceId in services)
    {
        tokenService.RemoveToken(serviceId);
    }
    
    Debug.Log($"✓ Cleared {services.Count} tokens");
}
```

## Token Events

### Listen for Token Events

```csharp
void Start()
{
    var tokenService = agent.MCPController.TokenService;
    
    tokenService.onTokenRefreshed.AddListener((serviceId, token) =>
    {
        Debug.Log($"✓ Token refreshed: {serviceId}");
        SaveTokenSecurely(serviceId, token);
    });
    
    tokenService.onTokenExpired.AddListener(serviceId =>
    {
        Debug.LogWarning($"⚠️ Token expired: {serviceId}");
        ShowTokenExpiredNotification(serviceId);
    });
    
    tokenService.onTokenRefreshFailed.AddListener((serviceId, error) =>
    {
        Debug.LogError($"Token refresh failed ({serviceId}): {error}");
        ShowTokenErrorDialog(serviceId, error);
    });
}
```

## Error Handling

### Handle Token Errors

```csharp
public async UniTask<bool> EnsureValidToken(string serviceId)
{
    var token = tokenService.GetToken(serviceId);
    
    if (token == null)
    {
        Debug.Log($"No token for {serviceId}, requesting authentication");
        await RequestAuthentication(serviceId);
        return false;
    }
    
    if (!IsTokenValid(serviceId))
    {
        Debug.Log($"Token invalid for {serviceId}, attempting refresh");
        
        bool refreshed = await RefreshToken(serviceId);
        
        if (!refreshed)
        {
            Debug.LogError($"Failed to refresh token for {serviceId}");
            await RequestAuthentication(serviceId);
            return false;
        }
    }
    
    return true;
}

async UniTask RequestAuthentication(string serviceId)
{
    // Trigger OAuth flow
    await agent.MCPController.RequestOAuthAsync(serviceId);
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class TokenServiceManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Settings")]
    [SerializeField] private bool enableAutoRefresh = true;
    [SerializeField] private int refreshThresholdSeconds = 300;
    [SerializeField] private int checkIntervalSeconds = 60;
    
    private AccessTokenService tokenService;
    private SecureTokenStorage secureStorage;
    private bool isMonitoring;
    
    async void Start()
    {
        await SetupTokenService();
    }
    
    async UniTask SetupTokenService()
    {
        tokenService = agent.MCPController.TokenService;
        secureStorage = GetComponent<SecureTokenStorage>();
        
        // Configure service
        tokenService.EnableAutoRefresh = enableAutoRefresh;
        tokenService.RefreshThresholdSeconds = refreshThresholdSeconds;
        
        // Listen for events
        tokenService.onTokenRefreshed.AddListener(OnTokenRefreshed);
        tokenService.onTokenExpired.AddListener(OnTokenExpired);
        tokenService.onTokenRefreshFailed.AddListener(OnTokenRefreshFailed);
        
        // Load saved tokens
        LoadSavedTokens();
        
        // Start monitoring
        if (enableAutoRefresh)
        {
            await MonitorTokens();
        }
        
        Debug.Log("✓ Token service ready");
    }
    
    void LoadSavedTokens()
    {
        string[] services = { "github", "google", "slack" };
        
        foreach (var serviceId in services)
        {
            var token = secureStorage.LoadTokenSecurely(serviceId);
            
            if (token != null)
            {
                tokenService.SetToken(serviceId, token);
                Debug.Log($"✓ Loaded token for {serviceId}");
            }
        }
    }
    
    async UniTask MonitorTokens()
    {
        isMonitoring = true;
        
        while (isMonitoring)
        {
            await CheckAndRefreshTokens();
            await UniTask.Delay(checkIntervalSeconds * 1000);
        }
    }
    
    async UniTask CheckAndRefreshTokens()
    {
        var services = tokenService.GetAllServices();
        
        foreach (var serviceId in services)
        {
            var token = tokenService.GetToken(serviceId);
            
            if (token == null)
                continue;
            
            long now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
            long timeUntilExpiry = token.ExpiresAt - now;
            
            // Refresh if expiring soon
            if (timeUntilExpiry < refreshThresholdSeconds)
            {
                Debug.Log($"🔄 Auto-refreshing token for {serviceId}");
                
                try
                {
                    await tokenService.RefreshTokenAsync(serviceId);
                }
                catch (Exception ex)
                {
                    Debug.LogError($"Auto-refresh failed for {serviceId}: {ex.Message}");
                }
            }
        }
    }
    
    void OnTokenRefreshed(string serviceId, AccessToken token)
    {
        Debug.Log($"✓ Token refreshed: {serviceId}");
        
        // Save updated token
        secureStorage.SaveTokenSecurely(serviceId, token);
    }
    
    void OnTokenExpired(string serviceId)
    {
        Debug.LogWarning($"⚠️ Token expired: {serviceId}");
        
        // Remove expired token
        secureStorage.DeleteToken(serviceId);
    }
    
    void OnTokenRefreshFailed(string serviceId, string error)
    {
        Debug.LogError($"Token refresh failed ({serviceId}): {error}");
        
        // Notify user
        ShowMessage($"Authentication failed for {serviceId}. Please sign in again.");
    }
    
    void ShowMessage(string message)
    {
        Debug.Log(message);
        // Show in UI
    }
    
    void OnDestroy()
    {
        isMonitoring = false;
    }
}
```

## Next Steps

- [OAuth Providers](oauth-providers.md)
- [Approval Handlers](approval-handlers.md)
- [MCP Tools](../tools/hosted-tools/mcp.md)
