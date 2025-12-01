# MCP (Model Context Protocol)

Connect to MCP servers to access external tools and resources.

## Overview

MCP (Model Context Protocol) enables agents to:

- Connect to MCP servers
- Access server-provided tools
- Use external resources
- Integrate with third-party services
- Extend agent capabilities dynamically

## Basic Setup

### Enable MCP

```csharp
// Access MCP controller
var mcp = agent.MCPController;

// Connect to server
await mcp.ConnectAsync("server-name");
```

## Connecting to Servers

### Connect to Local Server

```csharp
// Connect to local MCP server
await mcp.ConnectAsync(
    serverName: "filesystem",
    transport: "stdio",
    command: "node",
    args: new[] { "path/to/mcp-server.js" }
);
```

### Connect to Remote Server

```csharp
// Connect via HTTP
await mcp.ConnectAsync(
    serverName: "api-server",
    transport: "http",
    url: "https://api.example.com/mcp"
);

// Connect via SSE (Server-Sent Events)
await mcp.ConnectAsync(
    serverName: "sse-server",
    transport: "sse",
    url: "https://api.example.com/mcp/sse"
);
```

## Server Discovery

### List Available Servers

```csharp
var servers = await mcp.ListServersAsync();

foreach (var server in servers)
{
    Debug.Log($"Server: {server.Name}");
    Debug.Log($"  Description: {server.Description}");
    Debug.Log($"  Version: {server.Version}");
}
```

### Check Server Status

```csharp
var status = await mcp.GetServerStatusAsync("filesystem");

Debug.Log($"Status: {status.Connected}");
Debug.Log($"Tools: {status.ToolCount}");
Debug.Log($"Resources: {status.ResourceCount}");
```

## Using MCP Tools

### List Server Tools

```csharp
var tools = await mcp.ListToolsAsync("filesystem");

foreach (var tool in tools)
{
    Debug.Log($"Tool: {tool.Name}");
    Debug.Log($"  Description: {tool.Description}");
    Debug.Log($"  Parameters: {string.Join(", ", tool.Parameters)}");
}
```

### Call MCP Tool

```csharp
// MCP tools are automatically available to agent
await agent.SendAsync("List files in the Documents folder");

// Or call directly
var result = await mcp.CallToolAsync(
    serverName: "filesystem",
    toolName: "list_files",
    arguments: new { path = "Documents" }
);
```

## Built-in MCP Servers

### Filesystem Server

```csharp
public class FilesystemMCP : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void Start()
    {
        var mcp = agent.MCPController;
        
        // Connect to filesystem server
        await mcp.ConnectAsync(
            serverName: "filesystem",
            transport: "stdio",
            command: "npx",
            args: new[] { "-y", "@modelcontextprotocol/server-filesystem", "." }
        );
        
        Debug.Log("✓ Filesystem MCP connected");
    }
    
    public async void ListFiles(string directory)
    {
        await agent.SendAsync($"List all files in {directory}");
    }
    
    public async void ReadFile(string filePath)
    {
        await agent.SendAsync($"Read the contents of {filePath}");
    }
}
```

### GitHub MCP

```csharp
async void ConnectToGitHub()
{
    var mcp = agent.MCPController;
    
    await mcp.ConnectAsync(
        serverName: "github",
        transport: "stdio",
        command: "npx",
        args: new[] { "-y", "@modelcontextprotocol/server-github" },
        env: new Dictionary<string, string>
        {
            { "GITHUB_TOKEN", "your-token-here" }
        }
    );
    
    // Now agent can interact with GitHub
    await agent.SendAsync("List my GitHub repositories");
    await agent.SendAsync("Create a new issue in my-repo");
}
```

### Database MCP

```csharp
async void ConnectToDatabase()
{
    var mcp = agent.MCPController;
    
    await mcp.ConnectAsync(
        serverName: "postgres",
        transport: "stdio",
        command: "npx",
        args: new[] { "-y", "@modelcontextprotocol/server-postgres" },
        env: new Dictionary<string, string>
        {
            { "DATABASE_URL", "postgresql://localhost/gamedb" }
        }
    );
    
    // Query database through agent
    await agent.SendAsync("Show all players with score > 1000");
}
```

## OAuth Authentication

### Handle OAuth Flow

```csharp
public class MCPOAuthHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        var mcp = agent.MCPController;
        mcp.onOAuthRequired.AddListener(HandleOAuthRequest);
    }
    
    async void HandleOAuthRequest(OAuthRequest request)
    {
        Debug.Log($"OAuth required for: {request.ServerName}");
        Debug.Log($"Auth URL: {request.AuthUrl}");
        
        // Open browser for authorization
        Application.OpenURL(request.AuthUrl);
        
        // Wait for callback (implement your own callback handler)
        string authCode = await WaitForAuthCallback();
        
        // Complete OAuth
        await agent.MCPController.CompleteOAuthAsync(
            request.ServerName,
            authCode
        );
        
        Debug.Log("✓ OAuth completed");
    }
    
    async UniTask<string> WaitForAuthCallback()
    {
        // Implement callback listener
        // This could be a local HTTP server or deep link handler
        await UniTask.Delay(1000);
        return "auth-code-from-callback";
    }
}
```

## Resource Access

### Read Resources

```csharp
// MCP servers can provide resources (files, data, etc.)
var resources = await mcp.ListResourcesAsync("filesystem");

foreach (var resource in resources)
{
    Debug.Log($"Resource: {resource.Uri}");
    Debug.Log($"  Type: {resource.MimeType}");
    Debug.Log($"  Size: {resource.Size}");
}

// Read resource
var content = await mcp.ReadResourceAsync(
    serverName: "filesystem",
    resourceUri: "file:///path/to/file.txt"
);
```

## Approval System

### Request Approval

```csharp
public class MCPApprovalHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject approvalDialogPrefab;
    
    void Start()
    {
        var mcp = agent.MCPController;
        mcp.onApprovalRequired.AddListener(HandleApprovalRequest);
    }
    
    async void HandleApprovalRequest(ApprovalRequest request)
    {
        Debug.Log($"Approval needed: {request.ToolName}");
        Debug.Log($"Server: {request.ServerName}");
        Debug.Log($"Arguments: {request.Arguments}");
        
        // Show approval dialog
        var dialog = Instantiate(approvalDialogPrefab);
        var handler = dialog.GetComponent<ApprovalDialog>();
        
        bool approved = await handler.ShowAndWaitAsync(request);
        
        if (approved)
        {
            await mcp.ApproveToolCallAsync(request.Id);
            Debug.Log("✓ Tool call approved");
        }
        else
        {
            await mcp.DenyToolCallAsync(request.Id);
            Debug.Log("✗ Tool call denied");
        }
    }
}
```

## Custom MCP Server

### Create Simple Server

```javascript
// mcp-server.js
const { MCPServer } = require('@modelcontextprotocol/sdk');

const server = new MCPServer({
  name: 'game-server',
  version: '1.0.0'
});

// Register tool
server.registerTool({
  name: 'get_player_stats',
  description: 'Get player statistics',
  parameters: {
    playerId: { type: 'string', required: true }
  },
  handler: async (args) => {
    // Fetch player stats
    const stats = await getPlayerStats(args.playerId);
    return JSON.stringify(stats);
  }
});

server.listen();
```

### Connect to Custom Server

```csharp
async void ConnectToGameServer()
{
    var mcp = agent.MCPController;
    
    await mcp.ConnectAsync(
        serverName: "game-server",
        transport: "stdio",
        command: "node",
        args: new[] { "path/to/mcp-server.js" }
    );
    
    // Use custom tool
    await agent.SendAsync("Get stats for player123");
}
```

## Error Handling

### Connection Errors

```csharp
try
{
    await mcp.ConnectAsync("filesystem");
}
catch (MCPConnectionException ex)
{
    Debug.LogError($"Failed to connect: {ex.Message}");
    
    if (ex.Reason == "server_not_found")
    {
        Debug.Log("Server not installed. Install with: npm install -g @modelcontextprotocol/server-filesystem");
    }
    else if (ex.Reason == "timeout")
    {
        Debug.Log("Connection timed out. Server may not be responding.");
    }
}
```

### Tool Execution Errors

```csharp
agent.onToolCallError.AddListener((toolCall, error) =>
{
    if (toolCall.Type == ToolType.MCP)
    {
        Debug.LogError($"MCP tool failed: {error}");
        
        // Parse MCP error
        if (error.Contains("unauthorized"))
        {
            ShowMessage("Authorization required. Please authenticate.");
        }
        else if (error.Contains("not_found"))
        {
            ShowMessage("Resource not found.");
        }
    }
});
```

## Server Management

### Disconnect Server

```csharp
await mcp.DisconnectAsync("filesystem");
Debug.Log("✓ Disconnected from filesystem server");
```

### Reconnect

```csharp
public async UniTask ReconnectServer(string serverName)
{
    await mcp.DisconnectAsync(serverName);
    await UniTask.Delay(1000);
    await mcp.ConnectAsync(serverName);
}
```

### Monitor Connection

```csharp
void Start()
{
    var mcp = agent.MCPController;
    
    mcp.onServerConnected.AddListener(server =>
    {
        Debug.Log($"✓ Connected: {server.Name}");
    });
    
    mcp.onServerDisconnected.AddListener(server =>
    {
        Debug.LogWarning($"✗ Disconnected: {server.Name}");
        
        // Auto-reconnect
        ReconnectServer(server.Name);
    });
}
```

## Best Practices

### 1. Validate Server Availability

```csharp
async UniTask<bool> IsServerAvailable(string serverName)
{
    try
    {
        await mcp.ConnectAsync(serverName);
        await mcp.DisconnectAsync(serverName);
        return true;
    }
    catch
    {
        return false;
    }
}
```

### 2. Handle Missing Dependencies

```csharp
async void ConnectWithDependencyCheck()
{
    if (!IsNodeInstalled())
    {
        ShowMessage("Node.js is required for MCP servers");
        return;
    }
    
    try
    {
        await mcp.ConnectAsync("filesystem");
    }
    catch (MCPConnectionException ex)
    {
        if (ex.Reason == "server_not_found")
        {
            ShowInstallInstructions();
        }
    }
}

bool IsNodeInstalled()
{
    try
    {
        var process = Process.Start(new ProcessStartInfo
        {
            FileName = "node",
            Arguments = "--version",
            RedirectStandardOutput = true,
            UseShellExecute = false
        });
        
        process.WaitForExit();
        return process.ExitCode == 0;
    }
    catch
    {
        return false;
    }
}
```

### 3. Cache Server Connections

```csharp
public class MCPConnectionCache : MonoBehaviour
{
    private HashSet<string> connectedServers = new();
    
    public async UniTask<bool> EnsureConnected(string serverName)
    {
        if (connectedServers.Contains(serverName))
        {
            return true;
        }
        
        try
        {
            await mcp.ConnectAsync(serverName);
            connectedServers.Add(serverName);
            return true;
        }
        catch
        {
            return false;
        }
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class MCPManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private MCPController mcp;
    private HashSet<string> connectedServers = new();
    
    async void Start()
    {
        await SetupMCP();
    }
    
    async UniTask SetupMCP()
    {
        mcp = agent.MCPController;
        
        // Listen for events
        mcp.onServerConnected.AddListener(OnServerConnected);
        mcp.onServerDisconnected.AddListener(OnServerDisconnected);
        mcp.onOAuthRequired.AddListener(HandleOAuth);
        mcp.onApprovalRequired.AddListener(HandleApproval);
        
        // Connect to servers
        await ConnectFilesystem();
        await ConnectGitHub();
        
        Debug.Log("✓ MCP ready");
    }
    
    async UniTask ConnectFilesystem()
    {
        try
        {
            await mcp.ConnectAsync(
                serverName: "filesystem",
                transport: "stdio",
                command: "npx",
                args: new[] { "-y", "@modelcontextprotocol/server-filesystem", "." }
            );
            
            Debug.Log("✓ Filesystem connected");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Filesystem connection failed: {ex.Message}");
        }
    }
    
    async UniTask ConnectGitHub()
    {
        string token = PlayerPrefs.GetString("GitHubToken", "");
        
        if (string.IsNullOrEmpty(token))
        {
            Debug.LogWarning("GitHub token not set");
            return;
        }
        
        try
        {
            await mcp.ConnectAsync(
                serverName: "github",
                transport: "stdio",
                command: "npx",
                args: new[] { "-y", "@modelcontextprotocol/server-github" },
                env: new Dictionary<string, string>
                {
                    { "GITHUB_TOKEN", token }
                }
            );
            
            Debug.Log("✓ GitHub connected");
        }
        catch (Exception ex)
        {
            Debug.LogError($"GitHub connection failed: {ex.Message}");
        }
    }
    
    void OnServerConnected(MCPServer server)
    {
        Debug.Log($"✓ Connected: {server.Name}");
        connectedServers.Add(server.Name);
    }
    
    void OnServerDisconnected(MCPServer server)
    {
        Debug.LogWarning($"✗ Disconnected: {server.Name}");
        connectedServers.Remove(server.Name);
        
        // Auto-reconnect after delay
        ReconnectAfterDelay(server.Name);
    }
    
    async void ReconnectAfterDelay(string serverName)
    {
        await UniTask.Delay(5000);
        
        try
        {
            await mcp.ConnectAsync(serverName);
            Debug.Log($"✓ Reconnected: {serverName}");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Reconnect failed: {ex.Message}");
        }
    }
    
    async void HandleOAuth(OAuthRequest request)
    {
        Debug.Log($"OAuth required: {request.ServerName}");
        Application.OpenURL(request.AuthUrl);
        
        // Wait for user to complete OAuth
        string authCode = await WaitForOAuthCallback();
        
        if (!string.IsNullOrEmpty(authCode))
        {
            await mcp.CompleteOAuthAsync(request.ServerName, authCode);
            Debug.Log("✓ OAuth completed");
        }
    }
    
    async void HandleApproval(ApprovalRequest request)
    {
        Debug.Log($"Approval needed: {request.ToolName}");
        
        // For demo, auto-approve safe operations
        bool approved = IsSafeOperation(request);
        
        if (approved)
        {
            await mcp.ApproveToolCallAsync(request.Id);
        }
        else
        {
            await mcp.DenyToolCallAsync(request.Id);
        }
    }
    
    bool IsSafeOperation(ApprovalRequest request)
    {
        // Define safe operations
        string[] safeOps = { "read_file", "list_files", "get_*" };
        
        return safeOps.Any(op =>
            request.ToolName.StartsWith(op.Replace("*", "")));
    }
    
    async UniTask<string> WaitForOAuthCallback()
    {
        // Implement OAuth callback handling
        await UniTask.Delay(1000);
        return ""; // Return auth code from callback
    }
    
    void OnApplicationQuit()
    {
        // Disconnect all servers
        foreach (var server in connectedServers)
        {
            mcp.DisconnectAsync(server).Forget();
        }
    }
}
```

## Next Steps

- [OAuth Providers](../../mcp-integration/oauth-providers.md)
- [Approval Handlers](../../mcp-integration/approval-handlers.md)
- [Image Generation](image-generation.md)
