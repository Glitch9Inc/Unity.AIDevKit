# Local Shell Tool Calls

Execute shell commands locally from agent tool calls.

## Overview

Local Shell allows agents to:

- Run command-line tools
- Execute scripts
- Access system utilities
- Manage files and processes

⚠️ **Security Warning**: Shell access should be carefully controlled and validated.

## Basic Usage

### Enable Shell Tool

```csharp
// Add shell tool
agent.AddTool(ToolType.LocalShell);

// Agent can now execute shell commands
```

### Execute Command

```csharp
agent.AddTool("run_command", (string command) =>
{
    var process = new System.Diagnostics.Process
    {
        StartInfo = new System.Diagnostics.ProcessStartInfo
        {
            FileName = "cmd.exe",
            Arguments = $"/c {command}",
            RedirectStandardOutput = true,
            UseShellExecute = false,
            CreateNoWindow = true
        }
    };
    
    process.Start();
    string output = process.StandardOutput.ReadToEnd();
    process.WaitForExit();
    
    return output;
});
```

## Safe Command Execution

### Command Whitelist

```csharp
public class SafeShellExecutor : IToolExecutor
{
    public string[] SupportedTools => new[] { "run_safe_command" };
    
    private readonly HashSet<string> allowedCommands = new()
    {
        "dir",
        "echo",
        "type",
        "findstr"
    };
    
    public async UniTask<string> ExecuteAsync(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<CommandArgs>(toolCall.Function.Arguments);
        
        // Extract command name
        string commandName = args.command.Split(' ')[0];
        
        // Check whitelist
        if (!allowedCommands.Contains(commandName))
        {
            return $"Error: Command '{commandName}' is not allowed";
        }
        
        // Execute
        return await ExecuteCommand(args.command);
    }
    
    async UniTask<string> ExecuteCommand(string command)
    {
        var process = new System.Diagnostics.Process
        {
            StartInfo = new System.Diagnostics.ProcessStartInfo
            {
                FileName = "cmd.exe",
                Arguments = $"/c {command}",
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                UseShellExecute = false,
                CreateNoWindow = true
            }
        };
        
        process.Start();
        
        string output = await process.StandardOutput.ReadToEndAsync();
        string error = await process.StandardError.ReadToEndAsync();
        
        await process.WaitForExitAsync();
        
        if (process.ExitCode != 0)
        {
            return $"Error: {error}";
        }
        
        return output;
    }
    
    [System.Serializable]
    class CommandArgs
    {
        public string command;
    }
}
```

## File Operations

### File Management

```csharp
public class FileShellTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("list_files", ListFiles);
        agent.AddTool("read_file", ReadFile);
        agent.AddTool("find_files", FindFiles);
    }
    
    string ListFiles(string directory)
    {
        try
        {
            var files = Directory.GetFiles(directory);
            var dirs = Directory.GetDirectories(directory);
            
            return $"Directories: {dirs.Length}\n" +
                   string.Join("\n", dirs.Select(Path.GetFileName)) +
                   $"\n\nFiles: {files.Length}\n" +
                   string.Join("\n", files.Select(Path.GetFileName));
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string ReadFile(string filePath)
    {
        try
        {
            // Security: Only allow certain file types
            string extension = Path.GetExtension(filePath).ToLower();
            if (extension != ".txt" && extension != ".log" && extension != ".json")
            {
                return "Error: Only .txt, .log, and .json files allowed";
            }
            
            string content = File.ReadAllText(filePath);
            
            // Limit output size
            if (content.Length > 5000)
            {
                content = content.Substring(0, 5000) + "\n... (truncated)";
            }
            
            return content;
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string FindFiles(string pattern, string directory = ".")
    {
        try
        {
            var files = Directory.GetFiles(directory, pattern, SearchOption.AllDirectories);
            
            if (files.Length == 0)
            {
                return $"No files found matching '{pattern}'";
            }
            
            // Limit results
            var results = files.Take(20);
            string output = $"Found {files.Length} files (showing first 20):\n";
            output += string.Join("\n", results.Select(Path.GetFileName));
            
            return output;
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
}
```

## System Information

### System Queries

```csharp
agent.AddTool("get_system_info", () =>
{
    return $@"Operating System: {SystemInfo.operatingSystem}
Processor: {SystemInfo.processorType}
Processor Count: {SystemInfo.processorCount}
Memory: {SystemInfo.systemMemorySize} MB
Graphics: {SystemInfo.graphicsDeviceName}
Graphics Memory: {SystemInfo.graphicsMemorySize} MB";
});

agent.AddTool("get_disk_info", () =>
{
    var drives = DriveInfo.GetDrives();
    var info = new StringBuilder();
    
    foreach (var drive in drives)
    {
        if (drive.IsReady)
        {
            info.AppendLine($"{drive.Name}:");
            info.AppendLine($"  Total: {drive.TotalSize / 1024 / 1024 / 1024} GB");
            info.AppendLine($"  Free: {drive.AvailableFreeSpace / 1024 / 1024 / 1024} GB");
        }
    }
    
    return info.ToString();
});
```

## Process Management

### Process Control

```csharp
public class ProcessTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool("list_processes", ListProcesses);
        agent.AddTool("kill_process", KillProcess);
    }
    
    string ListProcesses()
    {
        var processes = Process.GetProcesses()
            .OrderByDescending(p => p.WorkingSet64)
            .Take(10);
        
        var output = new StringBuilder();
        output.AppendLine("Top 10 processes by memory:");
        
        foreach (var process in processes)
        {
            try
            {
                long memoryMB = process.WorkingSet64 / 1024 / 1024;
                output.AppendLine($"{process.ProcessName}: {memoryMB} MB");
            }
            catch
            {
                // Some processes may not be accessible
            }
        }
        
        return output.ToString();
    }
    
    string KillProcess(string processName)
    {
        try
        {
            var processes = Process.GetProcessesByName(processName);
            
            if (processes.Length == 0)
            {
                return $"No process found with name '{processName}'";
            }
            
            foreach (var process in processes)
            {
                process.Kill();
            }
            
            return $"Killed {processes.Length} process(es) named '{processName}'";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
}
```

## Script Execution

### Run Scripts

```csharp
public class ScriptExecutor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string scriptsDirectory = "Scripts";
    
    void Start()
    {
        agent.AddTool("run_script", RunScript);
    }
    
    async UniTask<string> RunScript(string scriptName)
    {
        string scriptPath = Path.Combine(scriptsDirectory, scriptName);
        
        // Security: Validate script path
        if (!File.Exists(scriptPath))
        {
            return $"Script not found: {scriptName}";
        }
        
        string extension = Path.GetExtension(scriptPath).ToLower();
        
        string executor = extension switch
        {
            ".bat" or ".cmd" => "cmd.exe",
            ".ps1" => "powershell.exe",
            ".py" => "python",
            _ => null
        };
        
        if (executor == null)
        {
            return $"Unsupported script type: {extension}";
        }
        
        // Execute
        var process = new Process
        {
            StartInfo = new ProcessStartInfo
            {
                FileName = executor,
                Arguments = extension == ".ps1" 
                    ? $"-ExecutionPolicy Bypass -File \"{scriptPath}\""
                    : $"/c \"{scriptPath}\"",
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                UseShellExecute = false,
                CreateNoWindow = true
            }
        };
        
        process.Start();
        
        string output = await process.StandardOutput.ReadToEndAsync();
        string error = await process.StandardError.ReadToEndAsync();
        
        await process.WaitForExitAsync();
        
        if (process.ExitCode != 0)
        {
            return $"Script failed:\n{error}";
        }
        
        return output;
    }
}
```

## Environment Variables

### Access Environment

```csharp
agent.AddTool("get_env", (string variableName) =>
{
    string value = Environment.GetEnvironmentVariable(variableName);
    
    if (string.IsNullOrEmpty(value))
    {
        return $"Environment variable '{variableName}' not found";
    }
    
    return $"{variableName} = {value}";
});

agent.AddTool("list_env", () =>
{
    var variables = Environment.GetEnvironmentVariables();
    var output = new StringBuilder();
    
    foreach (DictionaryEntry entry in variables)
    {
        output.AppendLine($"{entry.Key} = {entry.Value}");
    }
    
    return output.ToString();
});
```

## Timeout & Cancellation

### Handle Long Commands

```csharp
public async UniTask<string> ExecuteWithTimeout(
    string command,
    int timeoutSeconds = 30)
{
    var cts = new CancellationTokenSource();
    cts.CancelAfter(TimeSpan.FromSeconds(timeoutSeconds));
    
    var process = new Process
    {
        StartInfo = new ProcessStartInfo
        {
            FileName = "cmd.exe",
            Arguments = $"/c {command}",
            RedirectStandardOutput = true,
            UseShellExecute = false,
            CreateNoWindow = true
        }
    };
    
    try
    {
        process.Start();
        
        string output = await process.StandardOutput.ReadToEndAsync()
            .AsUniTask()
            .AttachExternalCancellation(cts.Token);
        
        await process.WaitForExitAsync(cts.Token);
        
        return output;
    }
    catch (OperationCanceledException)
    {
        process.Kill();
        return "Error: Command timed out";
    }
}
```

## Security Best Practices

### Input Sanitization

```csharp
public string SanitizeCommand(string command)
{
    // Remove dangerous characters
    var dangerous = new[] { "&", "|", ";", ">", "<", "`", "$", "(", ")" };
    
    foreach (var ch in dangerous)
    {
        if (command.Contains(ch))
        {
            throw new ArgumentException($"Command contains dangerous character: {ch}");
        }
    }
    
    return command;
}
```

### Permission Checks

```csharp
public class PermissionChecker
{
    private readonly HashSet<string> adminCommands = new()
    {
        "format",
        "del",
        "rmdir",
        "reg"
    };
    
    public bool RequiresAdmin(string command)
    {
        string cmd = command.Split(' ')[0].ToLower();
        return adminCommands.Contains(cmd);
    }
    
    public bool HasAdminRights()
    {
        using var identity = System.Security.Principal.WindowsIdentity.GetCurrent();
        var principal = new System.Security.Principal.WindowsPrincipal(identity);
        return principal.IsInRole(System.Security.Principal.WindowsBuiltInRole.Administrator);
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using System.Diagnostics;

public class SafeShellTools : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int commandTimeout = 30;
    
    private readonly HashSet<string> allowedCommands = new()
    {
        "dir", "echo", "type", "findstr", "where"
    };
    
    void Start()
    {
        agent.AddTool("shell_command", ExecuteShellCommand);
        agent.AddTool("list_directory", ListDirectory);
        agent.AddTool("find_file", FindFile);
    }
    
    async UniTask<string> ExecuteShellCommand(string command)
    {
        // Validate command
        if (!IsCommandAllowed(command))
        {
            return $"Error: Command not allowed";
        }
        
        // Sanitize input
        try
        {
            command = SanitizeCommand(command);
        }
        catch (ArgumentException ex)
        {
            return ex.Message;
        }
        
        // Execute with timeout
        return await ExecuteWithTimeout(command);
    }
    
    bool IsCommandAllowed(string command)
    {
        string cmdName = command.Split(' ')[0].ToLower();
        return allowedCommands.Contains(cmdName);
    }
    
    string SanitizeCommand(string command)
    {
        var dangerous = new[] { "&", "|", ";", ">", "<", "`", "$" };
        
        foreach (var ch in dangerous)
        {
            if (command.Contains(ch))
            {
                throw new ArgumentException($"Dangerous character: {ch}");
            }
        }
        
        return command;
    }
    
    async UniTask<string> ExecuteWithTimeout(string command)
    {
        var cts = new CancellationTokenSource();
        cts.CancelAfter(TimeSpan.FromSeconds(commandTimeout));
        
        var process = new Process
        {
            StartInfo = new ProcessStartInfo
            {
                FileName = "cmd.exe",
                Arguments = $"/c {command}",
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                UseShellExecute = false,
                CreateNoWindow = true
            }
        };
        
        try
        {
            process.Start();
            
            var outputTask = process.StandardOutput.ReadToEndAsync();
            var errorTask = process.StandardError.ReadToEndAsync();
            
            await process.WaitForExitAsync(cts.Token);
            
            string output = await outputTask;
            string error = await errorTask;
            
            if (process.ExitCode != 0)
            {
                return $"Error (code {process.ExitCode}):\n{error}";
            }
            
            return output;
        }
        catch (OperationCanceledException)
        {
            try { process.Kill(); } catch { }
            return "Error: Command timed out";
        }
    }
    
    string ListDirectory(string path = ".")
    {
        try
        {
            // Security: Validate path
            if (!Directory.Exists(path))
            {
                return $"Directory not found: {path}";
            }
            
            var files = Directory.GetFiles(path);
            var dirs = Directory.GetDirectories(path);
            
            var output = new StringBuilder();
            output.AppendLine($"Directory: {Path.GetFullPath(path)}");
            output.AppendLine($"\nDirectories ({dirs.Length}):");
            foreach (var dir in dirs)
            {
                output.AppendLine($"  [DIR] {Path.GetFileName(dir)}");
            }
            
            output.AppendLine($"\nFiles ({files.Length}):");
            foreach (var file in files)
            {
                var info = new FileInfo(file);
                output.AppendLine($"  {info.Length,10:N0} {info.Name}");
            }
            
            return output.ToString();
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
    
    string FindFile(string pattern, string directory = ".")
    {
        try
        {
            var files = Directory.GetFiles(
                directory,
                pattern,
                SearchOption.AllDirectories
            );
            
            if (files.Length == 0)
            {
                return $"No files found matching '{pattern}'";
            }
            
            var output = new StringBuilder();
            output.AppendLine($"Found {files.Length} files:");
            
            foreach (var file in files.Take(50))
            {
                output.AppendLine($"  {file}");
            }
            
            if (files.Length > 50)
            {
                output.AppendLine($"  ... and {files.Length - 50} more");
            }
            
            return output.ToString();
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
}
```

## Next Steps

- [Function Calls](function.md)
- [Computer Use](computer-use.md)
- [Hosted Tools](../hosted-tools/)
