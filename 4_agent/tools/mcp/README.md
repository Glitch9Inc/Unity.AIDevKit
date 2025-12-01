# MCP Integration

Model Context Protocol (MCP) integration for advanced tool capabilities.

## What is MCP?

MCP (Model Context Protocol) is an open protocol that enables:

- **Secure tool access** via OAuth
- **User approval workflows** for sensitive operations
- **Standardized tool interfaces** across platforms
- **External service integration** (Slack, GitHub, etc.)

## Configuration

### Enable MCP

```csharp
// In AgentSettings
settings.EnableMcpTools = true;

// MCP approval timeout
behaviour.McpApprovalTimeoutSeconds = 30;
```

### MCP Controller

```csharp
// Check if agent has MCP tools
bool hasMcp = agent.HasMcpTool;

// Access MCP controller
var mcpController = agent.mcpController;
```

## OAuth Token Providers

### Implement Token Provider

```csharp
public class CustomOAuthProvider : IMcpOAuthTokenProvider
{
    public async UniTask<string> GetAccessTokenAsync(
        string service,
        string[] scopes,
        CancellationToken ct = default)
    {
        // Implement OAuth flow
        // 1. Check if token exists
        // 2. If not, redirect to OAuth page
        // 3. Get authorization code
        // 4. Exchange for access token
        // 5. Return token
        
        return accessToken;
    }
}
```

### Register Provider

```csharp
var tokenService = new McpAccessTokenService();
tokenService.RegisterProvider("slack", new SlackOAuthProvider());
tokenService.RegisterProvider("github", new GitHubOAuthProvider());

// Pass to agent
var agent = new Agent(
    settings: settings,
    behaviour: behaviour,
    tokenService: tokenService
);
```

## Approval Handlers

### Built-in Approval Dialog

```csharp
// Agent prompts user automatically
behaviour.McpApprovalBehaviour = McpApprovalBehaviour.Prompt;
```

### Custom Approval Handler

```csharp
agent.onMcpToolApprovalRequested += async (toolCall, timeoutSeconds) =>
{
    // Show custom UI
    bool approved = await ShowApprovalDialogAsync(
        toolName: toolCall.Function.Name,
        arguments: toolCall.Function.Arguments,
        timeout: timeoutSeconds
    );
    
    return approved;
};
```

## Approval Timeout

```csharp
// Set timeout in AgentBehaviour
behaviour.McpApprovalTimeoutSeconds = 30;

// Or at runtime
agent.McpApprovalTimeoutSeconds = 60;
```

## Access Token Service

### Token Caching

```csharp
public class TokenCache
{
    private Dictionary<string, CachedToken> tokens = new();
    
    public async UniTask<string> GetOrRefreshTokenAsync(
        string service,
        string[] scopes)
    {
        if (tokens.TryGetValue(service, out var cached))
        {
            if (!cached.IsExpired)
            {
                return cached.Token;
            }
        }
        
        // Refresh token
        var newToken = await RefreshTokenAsync(service, scopes);
        tokens[service] = new CachedToken(newToken);
        return newToken;
    }
}
```

### Token Refresh

```csharp
public async UniTask<string> RefreshTokenAsync(
    string service,
    string refreshToken)
{
    // Use refresh token to get new access token
    var response = await HttpClient.PostAsync(
        $"https://{service}/oauth/token",
        new FormUrlEncodedContent(new Dictionary<string, string>
        {
            ["grant_type"] = "refresh_token",
            ["refresh_token"] = refreshToken
        })
    );
    
    var result = await response.Content.ReadAsStringAsync();
    var token = JsonConvert.DeserializeObject<TokenResponse>(result);
    
    return token.AccessToken;
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Glitch9.AIDevKit.Agents.McpTools;

public class McpManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Enable MCP
        agent.HasMcpTool = true;
        agent.McpApprovalTimeoutSeconds = 30;
        
        // Setup approval handler
        agent.onMcpToolApprovalRequested += OnApprovalRequested;
        
        // Setup token providers
        SetupTokenProviders();
    }
    
    void SetupTokenProviders()
    {
        var tokenService = new McpAccessTokenService();
        
        // Register OAuth providers
        tokenService.RegisterProvider("slack", new SlackOAuthProvider());
        tokenService.RegisterProvider("github", new GitHubOAuthProvider());
        
        // Use with agent (if creating manually)
        // var agent = new Agent(settings, behaviour, tokenService: tokenService);
    }
    
    async UniTask<bool> OnApprovalRequested(
        ToolCall toolCall,
        int timeoutSeconds)
    {
        Debug.Log($"Approval requested for: {toolCall.Function.Name}");
        
        // Show approval dialog
        bool approved = await ShowApprovalDialog(
            toolName: toolCall.Function.Name,
            description: GetToolDescription(toolCall),
            timeoutSeconds: timeoutSeconds
        );
        
        return approved;
    }
    
    async UniTask<bool> ShowApprovalDialog(
        string toolName,
        string description,
        int timeoutSeconds)
    {
        // Show custom UI dialog
        var dialog = Instantiate(approvalDialogPrefab);
        dialog.SetToolInfo(toolName, description);
        dialog.SetTimeout(timeoutSeconds);
        
        // Wait for user decision
        bool approved = await dialog.WaitForDecisionAsync();
        
        Destroy(dialog.gameObject);
        return approved;
    }
    
    string GetToolDescription(ToolCall toolCall)
    {
        // Generate human-readable description
        return $"Execute {toolCall.Function.Name} with arguments: " +
               JsonConvert.SerializeObject(toolCall.Function.Arguments);
    }
}

// OAuth Provider Example
public class SlackOAuthProvider : IMcpOAuthTokenProvider
{
    public async UniTask<string> GetAccessTokenAsync(
        string service,
        string[] scopes,
        CancellationToken ct = default)
    {
        // 1. Check for cached token
        if (HasValidToken(service))
        {
            return GetCachedToken(service);
        }
        
        // 2. Start OAuth flow
        string authUrl = BuildAuthUrl(service, scopes);
        Application.OpenURL(authUrl);
        
        // 3. Wait for callback
        string authCode = await WaitForAuthCodeAsync(ct);
        
        // 4. Exchange for token
        string accessToken = await ExchangeCodeForTokenAsync(authCode);
        
        // 5. Cache and return
        CacheToken(service, accessToken);
        return accessToken;
    }
}
```

## Best Practices

### 1. Secure Token Storage

```csharp
// Use secure storage (not PlayerPrefs)
public class SecureTokenStorage
{
    public void SaveToken(string service, string token)
    {
        // Use platform-specific secure storage
        #if UNITY_IOS
            // Use Keychain
        #elif UNITY_ANDROID
            // Use KeyStore
        #else
            // Use encrypted file
        #endif
    }
}
```

### 2. Handle Token Expiration

```csharp
if (tokenResponse.ExpiresIn.HasValue)
{
    var expiryTime = DateTime.UtcNow.AddSeconds(tokenResponse.ExpiresIn.Value);
    CacheTokenWithExpiry(service, token, expiryTime);
}
```

### 3. User-Friendly Approvals

```csharp
// Clear description
string description = $"Allow agent to {action} in {service}?";

// Show required permissions
ShowPermissionsList(requiredScopes);

// Provide option to trust always
bool trustAlways = await ShowTrustAlwaysOption();
```

## Next Steps

- [OAuth Token Providers](oauth-providers.md)
- [Approval Handlers](approval-handlers.md)
- [Approval Timeout](approval-timeout.md)
- [Access Token Service](access-token-service.md)
- [MCP Tool](../tools/hosted-tools/mcp.md)
