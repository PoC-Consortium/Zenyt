# RPC Interface for Daemon Mode

The RPC (Remote Procedure Call) interface allows administrators to interact with Jantar daemons running in the background, such as those managed by systemd.

## Overview

When running Jantar as a daemon (`-d` flag), it starts an RPC server that accepts commands via Unix domain socket (Linux/macOS) or named pipe (Windows). This enables you to:

- Query daemon status without stopping it
- Execute commands on running nodes
- Monitor bootstrap nodes remotely
- Manage nodes via scripts and automation

## Starting a Daemon with RPC

### Basic Daemon
```bash
# Start daemon with default RPC settings
jantar -d

# Daemon with custom config
jantar -c bootstrap.cfg -d

# Daemon with custom RPC socket
jantar -d --rpc-socket /var/run/jantar.sock

# Daemon with RPC authentication
jantar -d --rpc-auth "secret-token"
```

### Systemd Service
```ini
[Service]
ExecStart=/usr/local/bin/jantar -c /etc/jantar/bootstrap.cfg -d
```

## Sending Commands via RPC

Use the `-rpc` flag with `-e` to send commands to a running daemon:

```bash
# Check daemon status
jantar -rpc -e "status"

# List connected peers
jantar -rpc -e "net peers"

# Show version
jantar -rpc -e "sys version"

# List available files
jantar -rpc -e "fs ls"

# Send a message (master nodes)
jantar -rpc -e "net msg 12D3KooW... Hello from RPC"

# Add file (master nodes only)
jantar -rpc -e "fs add /path/to/file.txt"

# Graceful shutdown
jantar -rpc -e "quit"
# or
jantar -rpc -e "exit"
```

## Configuration

### Config File Settings

Add to your `jantar.cfg`:

```yaml
# Enable RPC server (default: true in daemon mode)
rpc_enabled: true

# Unix socket path (Linux/macOS)
rpc_socket_path: "/var/run/jantar.sock"

# Windows named pipe
# rpc_socket_path: "\\\\.\\pipe\\jantar-rpc"

# Optional authentication token
rpc_auth_token: "your-secret-token"
```

### Default Socket Locations

- **Linux**: `$XDG_RUNTIME_DIR/jantar.sock` or `/tmp/jantar.sock`
- **macOS**: `/tmp/jantar.sock`
- **Windows**: `\\.\pipe\jantar-rpc`

## Authentication

### Without Authentication (Default)
```bash
# Anyone with socket access can send commands
jantar -rpc -e "peers"
```

### With Authentication
```bash
# Start daemon with auth token
jantar -d --rpc-auth "secret123"

# Send authenticated command
jantar -rpc --rpc-auth "secret123" -e "peers"

# Wrong token = access denied
jantar -rpc --rpc-auth "wrong" -e "peers"
# Error: Authentication failed
```

## Security Considerations

### Unix Socket Permissions
The RPC socket is created with `600` permissions (owner only):
```bash
ls -l /tmp/jantar.sock
# srw------- 1 user user 0 Aug 8 12:00 /tmp/jantar.sock
```

### Best Practices

1. **Use Authentication** for production deployments:
   ```yaml
   rpc_auth_token: "$(openssl rand -hex 32)"
   ```

2. **Secure Socket Location** - Use directories with restricted access:
   ```yaml
   rpc_socket_path: "/var/run/jantar/rpc.sock"
   ```

3. **Limit Socket Access** - Only the service user should access:
   ```bash
   chmod 700 /var/run/jantar
   chown jantar:jantar /var/run/jantar
   ```

## Common Use Cases

### Monitoring Script
```bash
#!/bin/bash
# monitor.sh - Check daemon health

if ! jantar -rpc -e "status" > /dev/null 2>&1; then
    echo "Daemon not responding!"
    systemctl restart jantar
fi

PEERS=$(jantar -rpc -e "net peers" | grep -c "Connected")
echo "Connected peers: $PEERS"
```

### Automated File Distribution
```bash
#!/bin/bash
# distribute.sh - Add files from directory

for file in /var/jantar/uploads/*; do
    jantar -rpc --rpc-auth "$RPC_TOKEN" -e "fs add $file"
    sleep 5
done
```

### Remote Management
```bash
# Check multiple nodes
for node in node1 node2 node3; do
    echo "Checking $node..."
    ssh $node "jantar -rpc -e 'net peers' | head -5"
done
```

## Troubleshooting

### "Cannot connect to daemon"
- Check if daemon is running: `ps aux | grep jantar`
- Verify socket exists: `ls -l /tmp/jantar.sock`
- Check socket permissions

### "Authentication failed"
- Verify auth token matches between daemon and client
- Check for typos or extra spaces
- Token is case-sensitive

### "Command not found"
- Some commands are master-only (fs add, net crawl)
- Check command spelling and hierarchical format
- Use `help` to see available commands

### Socket Permission Denied
```bash
# Fix socket permissions
sudo chown $USER /tmp/jantar.sock
chmod 600 /tmp/jantar.sock
```

## RPC Commands Reference

### Standard Commands
All regular Jantar hierarchical commands work via RPC:
- `help [command]` - Show commands
- `net peers` - List peers
- `fs ls` - List files
- `sys version` - Show version
- `sys update` - Check updates
- `crypto gen [type]` - Generate crypto address
- `db status` - Database information

### RPC-Specific Commands
- `status` - Get daemon status and stats
- `list-commands` - Get all commands with descriptions
- `quit` / `exit` - Graceful daemon shutdown

### Master-Only Commands
Require master node with valid key:
- `fs add <file>` - Add file to P2P network
- `sys push-update` - Push update package
- `net crawl` - Network topology analysis

**📖 For complete command reference with examples and JSON formats:**
**See [RPC Commands Reference](rpc-commands-reference.md)**

## Integration Examples

### Systemd Service with RPC
```ini
[Unit]
Description=Jantar P2P Node
After=network.target

[Service]
Type=simple
User=jantar
ExecStart=/usr/local/bin/jantar -c /etc/jantar/node.cfg -d
ExecStop=/usr/local/bin/jantar -rpc -e "quit"
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Docker Container
```dockerfile
FROM ubuntu:22.04
COPY jantar /usr/local/bin/
COPY config.yaml /etc/jantar/

# Start daemon
CMD ["jantar", "-c", "/etc/jantar/config.yaml", "-d"]

# Health check via RPC
HEALTHCHECK CMD jantar -rpc -e "status" || exit 1
```

### Kubernetes Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: jantar-node
spec:
  containers:
  - name: jantar
    image: jantar:latest
    command: ["jantar", "-d"]
    livenessProbe:
      exec:
        command:
        - jantar
        - -rpc
        - -e
        - status
      periodSeconds: 30
```

## Advanced Features

### Custom Socket Path
```bash
# Start with custom socket
jantar -d --rpc-socket /opt/jantar/daemon.sock

# Connect to custom socket
jantar -rpc --rpc-socket /opt/jantar/daemon.sock -e "peers"
```

### Multiple Daemons
Run multiple daemons with different sockets:
```bash
# Bootstrap node
jantar -c bootstrap.cfg -d --rpc-socket /tmp/bootstrap.sock

# Regular node
jantar -c node.cfg -d --rpc-socket /tmp/node.sock

# Query each
jantar -rpc --rpc-socket /tmp/bootstrap.sock -e "peers"
jantar -rpc --rpc-socket /tmp/node.sock -e "peers"
```

### Scripted Control
```python
#!/usr/bin/env python3
import subprocess
import json

def rpc_command(cmd, socket="/tmp/jantar.sock"):
    """Execute RPC command and return output"""
    result = subprocess.run(
        ["jantar", "-rpc", "--rpc-socket", socket, "-e", cmd],
        capture_output=True,
        text=True
    )
    return result.stdout

# Get peer count
output = rpc_command("peers")
peer_count = output.count("Connected:")
print(f"Connected peers: {peer_count}")
```

## Summary

The RPC interface provides powerful remote control capabilities for Jantar daemons:

- ✅ **No Binary Bloat** - Same binary for daemon and client
- ✅ **Simple Interface** - Familiar `-e` command syntax
- ✅ **Secure by Default** - Unix socket with restricted permissions
- ✅ **Optional Auth** - Token-based authentication when needed
- ✅ **Full Compatibility** - All commands work via RPC

Perfect for production deployments where interactive access isn't available!

---
[← Back to Admin Guide](README.md) | [← Back to Index](../Index.md)