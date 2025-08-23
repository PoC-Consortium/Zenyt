# RPC Commands Reference

**Complete reference for Jantar RPC interface with examples and JSON 
message formats.**

## Overview

The RPC interface allows programmatic control of Jantar daemon 
instances via JSON messages over Unix domain sockets (Linux/macOS) or 
named pipes (Windows). All regular Jantar commands plus special 
daemon-specific commands are available.

## Message Format

### RPC Request
```json
{
  "command": "net peers",
  "auth_token": "optional-secret-token",
  "id": "unique-request-id"
}
```

### RPC Response
```json
{
  "id": "unique-request-id",
  "success": true,
  "output": "Command output text",
  "message_type": "error|success|warning",
  "data": {
    "optional": "structured-data"
  }
}
```

## Command-Line Usage

```bash
# Basic command execution
jantar -rpc -e "command"

# With authentication
jantar -rpc --rpc-auth "token" -e "command"

# Custom socket path
jantar -rpc --rpc-socket "/path/to/socket" -e "command"
```

## Available Commands

### RPC-Specific Commands

#### `status`
Get daemon status and statistics.

**Command:** `status`

**Example:**
```bash
jantar -rpc -e "status"
```

**Response:**
```json
{
  "id": "req-123",
  "success": true,
  "output": "Daemon running\nNode: MyNode\nPeers connected: 3",
  "message_type": null,
  "data": null
}
```

#### `list-commands`
Get all available commands with descriptions.

**Command:** `list-commands`

**Response:**
```json
{
  "id": "req-456",
  "success": true,
  "output": "Available commands",
  "data": {
    "commands": [
      {"name": "fs", "description": "File system operations"},
      {"name": "net", "description": "Network operations"},
      {"name": "crypto", "description": "Cryptocurrency operations"}
    ]
  }
}
```

### File System Commands (`fs`)

#### `fs ls [pattern]`
List available files in P2P network.

**Examples:**
```bash
jantar -rpc -e "fs ls"
jantar -rpc -e "fs ls *.pdf"
```

**Sample Response:**
```
Hash: abc123def456... | Size: 1.2 MB | Name: document.pdf
Hash: 789xyz012345... | Size: 500 KB | Name: image.jpg
```

#### `fs cp <hash> <filename>`
Download file from P2P network.

**Examples:**
```bash
jantar -rpc -e "fs cp abc123def456 document.pdf"
jantar -rpc -e "fs cp 789xyz012345 ~/Downloads/image.jpg"
```

#### `fs add <path>` (Master Nodes Only)
Share file to P2P network.

**Examples:**
```bash
jantar -rpc -e "fs add /home/user/document.pdf"
jantar -rpc -e "fs add ~/Pictures/vacation.jpg"
```

### Network Commands (`net`)

#### `net peers`
List connected peers.

**Example:**
```bash
jantar -rpc -e "net peers"
```

**Sample Response:**
```
Connected peers: 3
12D3KooWABC123... (Alice) - RTT: 45ms
12D3KooWDEF456... (Bob) - RTT: 67ms  
12D3KooWGHI789... (Charlie) - RTT: 23ms
```

#### `net msg <peer> <message>`
Send direct message to specific peer.

**Example:**
```bash
jantar -rpc -e "net msg 12D3KooWABC123 Hello from RPC"
```

#### `net send <message>`
Broadcast message to all connected peers.

**Example:**
```bash
jantar -rpc -e "net send System maintenance in 5 minutes"
```

### Cryptocurrency Commands (`crypto`)

#### `crypto gen [type] [seed]`
Generate cryptocurrency address.

**Examples:**
```bash
jantar -rpc -e "crypto gen"                    # ZENYT address
jantar -rpc -e "crypto gen BURST"              # BURST address  
jantar -rpc -e "crypto gen NXT myseed"         # NXT with seed
```

**Sample Response:**
```
BURST Address: BURST-ABCD-EFGH-IJKL-MNOPQ | Private Key: 1a2b3c4d...
```

#### `crypto val <address>`
Validate cryptocurrency address format.

**Example:**
```bash
jantar -rpc -e "crypto val ZENYT-ABCD-EFGH-IJKL-MNOP-QRSTUV"
```

#### `crypto list`
Show supported cryptocurrencies.

**Example:**
```bash
jantar -rpc -e "crypto list"
```

### Database Commands (`db`)

#### `db status`
Show database connection and statistics.

**Example:**
```bash
jantar -rpc -e "db status"
```

#### `db query <sql>`
Execute SQL query on local database.

**Example:**
```bash
jantar -rpc -e "db query 'SELECT COUNT(*) FROM peers'"
```

### System Commands (`sys`)

#### `sys version`
Show version information.

**Example:**
```bash
jantar -rpc -e "sys version"
```

**Sample Response:**
```
Jantar version 0.2.1 (x86_64-unknown-linux-gnu)
Built: 2025-08-23 08:08:30 UTC
```

#### `sys update`
Check for available updates.

**Example:**
```bash
jantar -rpc -e "sys update"
```

### Configuration Commands (`cfg`)

#### `cfg show`
Display current configuration.

**Example:**
```bash
jantar -rpc -e "cfg show"
```

#### `cfg reload`
Reload configuration from file.

**Example:**
```bash
jantar -rpc -e "cfg reload"
```

### General Commands

#### `help [command]`
Show help information.

**Examples:**
```bash
jantar -rpc -e "help"
jantar -rpc -e "help fs"
jantar -rpc -e "help crypto gen"
```

#### `quit` / `exit`
Gracefully shutdown daemon.

**Example:**
```bash
jantar -rpc -e "quit"
```

**Response:**
```json
{
  "id": "shutdown-123",
  "success": true,
  "output": "Shutting down daemon...\nGoodbye!",
  "message_type": null,
  "data": null
}
```

## Authentication Examples

### Setup with Authentication
```bash
# Start daemon with auth token
jantar -d --rpc-auth "secret123"

# All subsequent commands need token
jantar -rpc --rpc-auth "secret123" -e "net peers"
```

### Configuration File Auth
```yaml
# jantar.cfg
rpc:
  enabled: true
  auth_token: "my-secret-token"
  socket_path: "/var/run/jantar.sock"
```

## Error Responses

### Authentication Failed
```json
{
  "id": "req-error",
  "success": false,
  "output": "Authentication failed",
  "message_type": "error",
  "data": null
}
```

### Command Failed
```json
{
  "id": "req-fail",
  "success": false,
  "output": "Command failed: Unknown command 'invalid'",
  "message_type": "error", 
  "data": null
}
```

### Connection Error
```bash
jantar -rpc -e "peers"
# Error: Failed to connect to RPC socket at /tmp/jantar.sock. 
# Is the daemon running?
```

## Security Features

- **Socket Permissions**: Unix sockets created with 600 permissions (owner-only)
- **Optional Authentication**: Token-based authentication for production use
- **Input Sanitization**: All user input is escaped to prevent injection attacks
- **Safe Command Execution**: Commands executed through secure dispatcher

## Integration Examples

### Shell Script Monitoring
```bash
#!/bin/bash
STATUS=$(jantar -rpc -e "status" 2>/dev/null)
if [ $? -ne 0 ]; then
    echo "Daemon not responding!"
    systemctl restart jantar
else
    echo "Daemon OK: $STATUS"
fi
```

### Python Automation
```python
import subprocess
import json

def rpc_command(cmd, auth_token=None):
    args = ["jantar", "-rpc"]
    if auth_token:
        args.extend(["--rpc-auth", auth_token])
    args.extend(["-e", cmd])
    
    result = subprocess.run(args, capture_output=True, text=True)
    return result.returncode == 0, result.stdout

# Check daemon status
success, output = rpc_command("status")
if success:
    print(f"Daemon status: {output}")
```

### Monitoring with jq
```bash
# Extract structured data using jq
PEER_COUNT=$(jantar -rpc -e "net peers" | grep -c "12D3KooW")
echo "Connected peers: $PEER_COUNT"

# Check if specific peer is connected
if jantar -rpc -e "net peers" | grep -q "Alice"; then
    echo "Alice is online"
fi
```

## Socket Locations

| Platform | Default Socket Path |
|----------|-------------------|
| Linux | `$XDG_RUNTIME_DIR/jantar.sock` or `/tmp/jantar.sock` |
| macOS | `/tmp/jantar.sock` |
| Windows | `\\.\pipe\jantar-rpc` |

---
[← Back to RPC Interface Guide](rpc-interface.md) | 
[← Back to Admin Guide](README.md)