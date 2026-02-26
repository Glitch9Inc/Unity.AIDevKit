---
icon: plug
---

# MCP Integration

Integrate your agent with Model Context Protocol (MCP) servers for extended capabilities.

## Overview

The Model Context Protocol (MCP) is an open protocol that enables AI applications to connect with external data sources and tools. AI Dev Kit provides full MCP integration, allowing your agents to:

- **Access External Data** - Connect to databases, APIs, and file systems
- **Use MCP Tools** - Execute functions provided by MCP servers
- **Manage Resources** - Read and manipulate external resources
- **Handle Prompts** - Use pre-defined conversation starters

## Key Features

### Automatic Server Discovery

AI Dev Kit automatically discovers and connects to MCP servers configured in your system:

```csharp
// MCP servers are automatically loaded from configuration
Agent agent = new Agent(config);
await agent.InitializeAsync();

// All MCP tools are automatically registered
```

### OAuth Authentication

Support for OAuth 2.0 authentication with MCP servers:

- Google Drive
- GitHub
- Slack
- Custom OAuth providers

### Approval System

Built-in approval mechanism for sensitive operations:

- User confirmation for tool executions
- Configurable approval handlers
- Timeout management
- Automatic approval for trusted tools

### Token Management

Secure token storage and refresh:

- Automatic token refresh
- Encrypted token storage
- Per-server token management

## MCP Server Types

AI Dev Kit supports various MCP servers:

### Standard MCP Servers

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/files"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
      }
    }
  }
}
```

### Custom Servers

```csharp
// Register custom MCP server
var serverConfig = new McpServerConfig
{
    Name = "custom-server",
    Command = "node",
    Args = new[] { "path/to/server.js" },
    Env = new Dictionary<string, string>
    {
        { "API_KEY", "your-api-key" }
    }
};

agent.RegisterMcpServer(serverConfig);
```

## Usage Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class McpExample : MonoBehaviour
{
    private Agent agent;
    
    async void Start()
    {
        // Create agent with MCP support
        var config = new AgentConfiguration
        {
            Model = "gpt-4o",
            EnableMcp = true
        };
        
        agent = new Agent(config);
        await agent.InitializeAsync();
        
        // MCP tools are automatically available
        var response = await agent.SendMessageAsync(
            "Search for files containing 'config' in the project folder"
        );
        
        Debug.Log(response.Content);
    }
}
```

## Security Considerations

### Approval Required

Sensitive operations require user approval:

```csharp
// Configure approval handler
agent.McpApprovalHandler = async (toolCall) => {
    // Show confirmation dialog to user
    bool approved = await ShowApprovalDialog(toolCall);
    return approved;
};
```

### Token Security

Tokens are stored securely:

- Encrypted storage
- Per-user isolation
- Automatic cleanup on logout

### Sandboxing

MCP servers run in isolated environments:

- Limited file system access
- Network isolation
- Resource limits

## Topics

Explore MCP integration in detail:

- **[Access Token Service](access-token-service.md)** - Manage OAuth tokens
- **[Approval Handlers](approval-handlers.md)** - Configure approval workflows
- **[Approval Timeout](approval-timeout.md)** - Handle timeout scenarios
- **[OAuth Providers](oauth-providers.md)** - Configure OAuth providers

## Best Practices

### 1. Limit Server Access

```csharp
// Only allow specific directories
var fsServer = new McpServerConfig
{
    Name = "filesystem",
    Command = "npx",
    Args = new[] { 
        "-y", 
        "@modelcontextprotocol/server-filesystem",
        Application.dataPath // Only Unity project folder
    }
};
```

### 2. Implement Approval Logic

```csharp
agent.McpApprovalHandler = async (toolCall) => {
    // Auto-approve read-only operations
    if (toolCall.Name.StartsWith("read_"))
        return true;
    
    // Require approval for write operations
    return await ShowApprovalDialog(toolCall);
};
```

### 3. Handle Errors

```csharp
agent.OnMcpError += (error) => {
    Debug.LogError($"MCP Error: {error.Message}");
    // Implement fallback logic
};
```

### 4. Monitor Token Expiry

```csharp
agent.OnTokenExpired += async (serverName) => {
    // Prompt user to re-authenticate
    await RequestReauthentication(serverName);
};
```

## Configuration File

MCP servers are configured in `mcp_config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./allowed-folder"]
    },
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "--db-path", "./data.db"]
    },
    "github": {
      "command": "uvx",
      "args": ["mcp-server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
      }
    }
  }
}
```

## Common MCP Servers

### File System

Access local files and directories:

```bash
npx -y @modelcontextprotocol/server-filesystem /path/to/folder
```

### GitHub

Interact with GitHub repositories:

```bash
uvx mcp-server-github
```

### Google Drive

Access Google Drive files:

```bash
npx -y @modelcontextprotocol/server-gdrive
```

### Slack

Send and read Slack messages:

```bash
npx -y @modelcontextprotocol/server-slack
```

### PostgreSQL

Query PostgreSQL databases:

```bash
docker run -i mcp/postgres postgresql://user:pass@host/db
```

## Troubleshooting

### Server Not Starting

Check server logs:

```csharp
agent.OnMcpServerLog += (serverName, log) => {
    Debug.Log($"[{serverName}] {log}");
};
```

### Authentication Failed

Re-authenticate with the server:

```csharp
await agent.ReauthenticateMcpServer("server-name");
```

### Tool Not Found

Ensure server is properly initialized:

```csharp
// Wait for servers to initialize
await agent.WaitForMcpServersAsync();

// List available tools
var tools = agent.GetAvailableMcpTools();
foreach (var tool in tools)
{
    Debug.Log($"Tool: {tool.Name} - {tool.Description}");
}
```

## Learn More

- [MCP Documentation](https://modelcontextprotocol.io)
- [Available MCP Servers](https://github.com/modelcontextprotocol/servers)
- [Creating Custom MCP Servers](https://modelcontextprotocol.io/docs/tools/creating)

## Next Steps

Set up your first MCP integration:

1. Configure an MCP server in `mcp_config.json`
2. Set up [OAuth authentication](oauth-providers.md) if needed
3. Implement [approval handlers](approval-handlers.md) for security
4. Use [Access Token Service](access-token-service.md) to manage tokens
