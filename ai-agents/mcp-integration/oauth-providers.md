# OAuth Providers

Configure OAuth authentication for MCP servers.

## Overview

OAuth Providers enable:

- Secure authentication for MCP servers
- Third-party service access
- Token management
- Multi-provider support
- Refresh token handling

## Basic Setup

### Register OAuth Provider

```csharp
var mcpController = agent.MCPController;

mcpController.RegisterOAuthProvider(new OAuthProviderConfig
{
    ProviderId = "github",
    ClientId = "your-client-id",
    ClientSecret = "your-client-secret",
    AuthorizationEndpoint = "https://github.com/login/oauth/authorize",
    TokenEndpoint = "https://github.com/login/oauth/access_token",
    Scopes = new[] { "repo", "user" }
});
```

## Provider Configuration

### GitHub OAuth

```csharp
mcpController.RegisterOAuthProvider(new OAuthProviderConfig
{
    ProviderId = "github",
    ClientId = GetEnvironmentVariable("GITHUB_CLIENT_ID"),
    ClientSecret = GetEnvironmentVariable("GITHUB_CLIENT_SECRET"),
    AuthorizationEndpoint = "https://github.com/login/oauth/authorize",
    TokenEndpoint = "https://github.com/login/oauth/access_token",
    Scopes = new[] { "repo", "read:user" },
    RedirectUri = "http://localhost:8080/callback"
});
```

### Google OAuth

```csharp
mcpController.RegisterOAuthProvider(new OAuthProviderConfig
{
    ProviderId = "google",
    ClientId = GetEnvironmentVariable("GOOGLE_CLIENT_ID"),
    ClientSecret = GetEnvironmentVariable("GOOGLE_CLIENT_SECRET"),
    AuthorizationEndpoint = "https://accounts.google.com/o/oauth2/v2/auth",
    TokenEndpoint = "https://oauth2.googleapis.com/token",
    Scopes = new[] { "https://www.googleapis.com/auth/drive.readonly" },
    RedirectUri = "http://localhost:8080/callback"
});
```

### Slack OAuth

```csharp
mcpController.RegisterOAuthProvider(new OAuthProviderConfig
{
    ProviderId = "slack",
    ClientId = GetEnvironmentVariable("SLACK_CLIENT_ID"),
    ClientSecret = GetEnvironmentVariable("SLACK_CLIENT_SECRET"),
    AuthorizationEndpoint = "https://slack.com/oauth/v2/authorize",
    TokenEndpoint = "https://slack.com/api/oauth.v2.access",
    Scopes = new[] { "channels:read", "chat:write" },
    RedirectUri = "http://localhost:8080/callback"
});
```

## OAuth Flow

### Authorization Flow

```csharp
public class OAuthFlowManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private MCPController mcpController;
    
    void Start()
    {
        mcpController = agent.MCPController;
        mcpController.onOAuthRequired.AddListener(HandleOAuthRequired);
    }
    
    async void HandleOAuthRequired(OAuthRequest request)
    {
        Debug.Log($"OAuth required for: {request.ProviderId}");
        
        // 1. Open browser for authorization
        string authUrl = request.AuthorizationUrl;
        Application.OpenURL(authUrl);
        
        Debug.Log($"Please authorize in browser: {authUrl}");
        
        // 2. Wait for callback with authorization code
        string authCode = await WaitForAuthorizationCode();
        
        // 3. Exchange code for access token
        await mcpController.CompleteOAuthAsync(request.ProviderId, authCode);
        
        Debug.Log("✓ OAuth completed successfully");
    }
    
    async UniTask<string> WaitForAuthorizationCode()
    {
        // Start local HTTP server to receive callback
        var listener = new HttpListener();
        listener.Prefixes.Add("http://localhost:8080/");
        listener.Start();
        
        Debug.Log("Waiting for OAuth callback...");
        
        var context = await listener.GetContextAsync();
        var code = context.Request.QueryString["code"];
        
        // Send success response
        var response = context.Response;
        string responseString = "<html><body>Authorization successful! You can close this window.</body></html>";
        byte[] buffer = System.Text.Encoding.UTF8.GetBytes(responseString);
        response.ContentLength64 = buffer.Length;
        await response.OutputStream.WriteAsync(buffer, 0, buffer.Length);
        response.Close();
        
        listener.Stop();
        
        return code;
    }
}
```

### Deep Link Authentication

```csharp
public class DeepLinkOAuth : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private string pendingProvider;
    
    void Start()
    {
        // Register deep link handler
        Application.deepLinkActivated += OnDeepLinkActivated;
        
        agent.MCPController.onOAuthRequired.AddListener(HandleOAuthRequired);
    }
    
    void HandleOAuthRequired(OAuthRequest request)
    {
        pendingProvider = request.ProviderId;
        
        // Open browser with deep link callback
        string authUrl = request.AuthorizationUrl;
        authUrl += $"&redirect_uri=mygame://oauth/callback";
        
        Application.OpenURL(authUrl);
    }
    
    async void OnDeepLinkActivated(string url)
    {
        Debug.Log($"Deep link activated: {url}");
        
        // Parse authorization code
        var uri = new Uri(url);
        var query = System.Web.HttpUtility.ParseQueryString(uri.Query);
        string code = query["code"];
        
        if (!string.IsNullOrEmpty(code) && !string.IsNullOrEmpty(pendingProvider))
        {
            await agent.MCPController.CompleteOAuthAsync(pendingProvider, code);
            Debug.Log("✓ OAuth completed via deep link");
            
            pendingProvider = null;
        }
    }
}
```

## Token Management

### Store Access Tokens

```csharp
public class OAuthTokenStore : MonoBehaviour
{
    private const string TOKEN_KEY_PREFIX = "oauth_token_";
    
    public void SaveToken(string providerId, OAuthToken token)
    {
        string json = JsonUtility.ToJson(token);
        PlayerPrefs.SetString(TOKEN_KEY_PREFIX + providerId, json);
        PlayerPrefs.Save();
        
        Debug.Log($"✓ Token saved for {providerId}");
    }
    
    public OAuthToken LoadToken(string providerId)
    {
        string key = TOKEN_KEY_PREFIX + providerId;
        
        if (!PlayerPrefs.HasKey(key))
            return null;
        
        string json = PlayerPrefs.GetString(key);
        return JsonUtility.FromJson<OAuthToken>(json);
    }
    
    public void DeleteToken(string providerId)
    {
        PlayerPrefs.DeleteKey(TOKEN_KEY_PREFIX + providerId);
        PlayerPrefs.Save();
        
        Debug.Log($"✓ Token deleted for {providerId}");
    }
}

[System.Serializable]
public class OAuthToken
{
    public string AccessToken;
    public string RefreshToken;
    public long ExpiresAt;
    public string[] Scopes;
}
```

### Refresh Tokens

```csharp
public async UniTask<bool> RefreshAccessToken(string providerId)
{
    var token = LoadToken(providerId);
    
    if (token == null || string.IsNullOrEmpty(token.RefreshToken))
    {
        Debug.LogWarning("No refresh token available");
        return false;
    }
    
    try
    {
        var newToken = await mcpController.RefreshOAuthTokenAsync(
            providerId,
            token.RefreshToken
        );
        
        SaveToken(providerId, newToken);
        Debug.Log($"✓ Token refreshed for {providerId}");
        
        return true;
    }
    catch (Exception ex)
    {
        Debug.LogError($"Token refresh failed: {ex.Message}");
        return false;
    }
}
```

### Auto-Refresh

```csharp
public class TokenAutoRefresher : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int checkIntervalSeconds = 300; // 5 minutes
    
    private Dictionary<string, OAuthToken> tokens = new();
    
    async void Start()
    {
        await CheckAndRefreshTokens();
    }
    
    async UniTask CheckAndRefreshTokens()
    {
        while (true)
        {
            foreach (var kvp in tokens)
            {
                string providerId = kvp.Key;
                OAuthToken token = kvp.Value;
                
                // Check if token expires in next 5 minutes
                long now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
                if (token.ExpiresAt - now < 300)
                {
                    Debug.Log($"Token expiring soon for {providerId}, refreshing...");
                    await RefreshAccessToken(providerId);
                }
            }
            
            await UniTask.Delay(checkIntervalSeconds * 1000);
        }
    }
}
```

## Multi-Provider Support

### Provider Manager

```csharp
public class OAuthProviderManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, OAuthProviderConfig> providers = new();
    
    public void RegisterProviders()
    {
        // GitHub
        RegisterProvider(new OAuthProviderConfig
        {
            ProviderId = "github",
            ClientId = GetConfig("GITHUB_CLIENT_ID"),
            ClientSecret = GetConfig("GITHUB_CLIENT_SECRET"),
            AuthorizationEndpoint = "https://github.com/login/oauth/authorize",
            TokenEndpoint = "https://github.com/login/oauth/access_token",
            Scopes = new[] { "repo", "user" }
        });
        
        // Google
        RegisterProvider(new OAuthProviderConfig
        {
            ProviderId = "google",
            ClientId = GetConfig("GOOGLE_CLIENT_ID"),
            ClientSecret = GetConfig("GOOGLE_CLIENT_SECRET"),
            AuthorizationEndpoint = "https://accounts.google.com/o/oauth2/v2/auth",
            TokenEndpoint = "https://oauth2.googleapis.com/token",
            Scopes = new[] { "https://www.googleapis.com/auth/drive.readonly" }
        });
        
        // Slack
        RegisterProvider(new OAuthProviderConfig
        {
            ProviderId = "slack",
            ClientId = GetConfig("SLACK_CLIENT_ID"),
            ClientSecret = GetConfig("SLACK_CLIENT_SECRET"),
            AuthorizationEndpoint = "https://slack.com/oauth/v2/authorize",
            TokenEndpoint = "https://slack.com/api/oauth.v2.access",
            Scopes = new[] { "channels:read", "chat:write" }
        });
    }
    
    void RegisterProvider(OAuthProviderConfig config)
    {
        agent.MCPController.RegisterOAuthProvider(config);
        providers[config.ProviderId] = config;
        
        Debug.Log($"✓ Registered OAuth provider: {config.ProviderId}");
    }
    
    public bool IsProviderRegistered(string providerId)
    {
        return providers.ContainsKey(providerId);
    }
    
    string GetConfig(string key)
    {
        return PlayerPrefs.GetString(key, "");
    }
}
```

## Security

### Secure Token Storage

```csharp
public class SecureTokenStore : MonoBehaviour
{
    public void SaveTokenSecurely(string providerId, OAuthToken token)
    {
        // Encrypt token before storing
        string json = JsonUtility.ToJson(token);
        string encrypted = EncryptString(json);
        
        PlayerPrefs.SetString($"oauth_token_{providerId}", encrypted);
        PlayerPrefs.Save();
    }
    
    public OAuthToken LoadTokenSecurely(string providerId)
    {
        string key = $"oauth_token_{providerId}";
        
        if (!PlayerPrefs.HasKey(key))
            return null;
        
        string encrypted = PlayerPrefs.GetString(key);
        string json = DecryptString(encrypted);
        
        return JsonUtility.FromJson<OAuthToken>(json);
    }
    
    string EncryptString(string plainText)
    {
        // Implement encryption (e.g., AES)
        // For production, use proper encryption library
        return System.Convert.ToBase64String(
            System.Text.Encoding.UTF8.GetBytes(plainText)
        );
    }
    
    string DecryptString(string cipherText)
    {
        // Implement decryption
        byte[] bytes = System.Convert.FromBase64String(cipherText);
        return System.Text.Encoding.UTF8.GetString(bytes);
    }
}
```

### Validate Scopes

```csharp
public bool ValidateScopes(string providerId, string[] requiredScopes)
{
    var token = LoadToken(providerId);
    
    if (token == null || token.Scopes == null)
        return false;
    
    foreach (var required in requiredScopes)
    {
        if (!token.Scopes.Contains(required))
        {
            Debug.LogWarning($"Missing scope: {required}");
            return false;
        }
    }
    
    return true;
}
```

## Error Handling

### Handle OAuth Errors

```csharp
agent.MCPController.onOAuthError.AddListener((providerId, error) =>
{
    Debug.LogError($"OAuth error for {providerId}: {error}");
    
    if (error.Contains("access_denied"))
    {
        ShowMessage("Authorization was denied. Please try again.");
    }
    else if (error.Contains("invalid_grant"))
    {
        ShowMessage("Authorization code expired. Please re-authenticate.");
        DeleteToken(providerId);
    }
    else if (error.Contains("invalid_client"))
    {
        ShowMessage("Invalid client credentials. Check configuration.");
    }
    else
    {
        ShowMessage("OAuth authentication failed.");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using System.Net;

public class OAuthManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private MCPController mcpController;
    private Dictionary<string, OAuthToken> tokens = new();
    
    async void Start()
    {
        await SetupOAuth();
    }
    
    async UniTask SetupOAuth()
    {
        mcpController = agent.MCPController;
        
        // Register providers
        RegisterGitHubProvider();
        RegisterGoogleProvider();
        
        // Listen for OAuth events
        mcpController.onOAuthRequired.AddListener(HandleOAuthRequired);
        mcpController.onOAuthCompleted.AddListener(HandleOAuthCompleted);
        mcpController.onOAuthError.AddListener(HandleOAuthError);
        
        // Load saved tokens
        LoadSavedTokens();
        
        Debug.Log("✓ OAuth manager ready");
    }
    
    void RegisterGitHubProvider()
    {
        mcpController.RegisterOAuthProvider(new OAuthProviderConfig
        {
            ProviderId = "github",
            ClientId = GetConfig("GITHUB_CLIENT_ID"),
            ClientSecret = GetConfig("GITHUB_CLIENT_SECRET"),
            AuthorizationEndpoint = "https://github.com/login/oauth/authorize",
            TokenEndpoint = "https://github.com/login/oauth/access_token",
            Scopes = new[] { "repo", "read:user" },
            RedirectUri = "http://localhost:8080/callback"
        });
        
        Debug.Log("✓ GitHub OAuth provider registered");
    }
    
    void RegisterGoogleProvider()
    {
        mcpController.RegisterOAuthProvider(new OAuthProviderConfig
        {
            ProviderId = "google",
            ClientId = GetConfig("GOOGLE_CLIENT_ID"),
            ClientSecret = GetConfig("GOOGLE_CLIENT_SECRET"),
            AuthorizationEndpoint = "https://accounts.google.com/o/oauth2/v2/auth",
            TokenEndpoint = "https://oauth2.googleapis.com/token",
            Scopes = new[] { "https://www.googleapis.com/auth/drive.readonly" },
            RedirectUri = "http://localhost:8080/callback"
        });
        
        Debug.Log("✓ Google OAuth provider registered");
    }
    
    async void HandleOAuthRequired(OAuthRequest request)
    {
        Debug.Log($"🔐 OAuth required: {request.ProviderId}");
        
        // Open browser
        Application.OpenURL(request.AuthorizationUrl);
        Debug.Log($"Opening: {request.AuthorizationUrl}");
        
        // Wait for callback
        string authCode = await WaitForAuthorizationCallback();
        
        if (!string.IsNullOrEmpty(authCode))
        {
            await mcpController.CompleteOAuthAsync(request.ProviderId, authCode);
        }
    }
    
    async UniTask<string> WaitForAuthorizationCallback()
    {
        var listener = new HttpListener();
        listener.Prefixes.Add("http://localhost:8080/");
        listener.Start();
        
        Debug.Log("⏳ Waiting for OAuth callback...");
        
        try
        {
            var context = await listener.GetContextAsync();
            var code = context.Request.QueryString["code"];
            
            // Send success page
            var response = context.Response;
            string html = "<html><body><h1>✓ Authorization Successful</h1><p>You can close this window.</p></body></html>";
            byte[] buffer = System.Text.Encoding.UTF8.GetBytes(html);
            response.ContentLength64 = buffer.Length;
            await response.OutputStream.WriteAsync(buffer, 0, buffer.Length);
            response.Close();
            
            return code;
        }
        finally
        {
            listener.Stop();
        }
    }
    
    void HandleOAuthCompleted(string providerId, OAuthToken token)
    {
        Debug.Log($"✓ OAuth completed: {providerId}");
        
        // Save token
        tokens[providerId] = token;
        SaveToken(providerId, token);
    }
    
    void HandleOAuthError(string providerId, string error)
    {
        Debug.LogError($"OAuth error ({providerId}): {error}");
    }
    
    void SaveToken(string providerId, OAuthToken token)
    {
        string json = JsonUtility.ToJson(token);
        PlayerPrefs.SetString($"oauth_token_{providerId}", json);
        PlayerPrefs.Save();
    }
    
    void LoadSavedTokens()
    {
        string[] providers = { "github", "google" };
        
        foreach (var providerId in providers)
        {
            string key = $"oauth_token_{providerId}";
            if (PlayerPrefs.HasKey(key))
            {
                string json = PlayerPrefs.GetString(key);
                var token = JsonUtility.FromJson<OAuthToken>(json);
                tokens[providerId] = token;
                
                Debug.Log($"✓ Loaded token for {providerId}");
            }
        }
    }
    
    string GetConfig(string key)
    {
        return PlayerPrefs.GetString(key, "");
    }
}
```

## Next Steps

- [Approval Handlers](approval-handlers.md)
- [Approval Timeout](approval-timeout.md)
- [Access Token Service](access-token-service.md)
