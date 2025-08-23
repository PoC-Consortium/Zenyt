# Access Control List (ACL) Configuration

This document explains how to configure command access control for different 
interfaces in Jantar.

## Overview

Jantar supports Access Control Lists (ACLs) to restrict which commands can be 
executed through different interfaces:

- **terminal**: Direct command line interface (interactive mode)
- **rpc**: RPC daemon interface (used by RPC clients and WebCLI)
- **web**: Web interface (for future web-specific restrictions)

## Configuration Format

Add an `acl` section to your `jantar.cfg` file:

```yaml
acl:
  terminal:
    allow: ["*"]           # Allow all commands
    forbid: []             # Forbid no commands
  rpc:
    allow: ["*"]           # Allow all commands  
    forbid: []             # Forbid no commands
  web:
    allow: ["*"]           # Allow all commands
    forbid: ["exit", "quit"]  # Forbid exit/quit commands
```

## Default Configuration

By default, Jantar uses these ACL settings:

- **terminal**: Allow all commands (no restrictions)
- **rpc**: Allow all commands (no restrictions)  
- **web**: Allow all commands **except** `exit` and `quit`

This prevents web users from accidentally terminating the daemon.

## Configuration Rules

### Allow List Processing

The `allow` list determines which commands are permitted:

- **Empty list or `["*"]`**: Allow all available commands
- **Specific commands**: Only allow listed commands

Examples:
```yaml
allow: ["*"]                    # Allow everything
allow: []                       # Allow everything (same as "*")
allow: ["fs", "net", "help"]   # Only allow fs, net, and help commands
```

### Forbid List Processing  

The `forbid` list takes **precedence** over the allow list:

- Commands in the forbid list are **always blocked**
- This happens even if they're in the allow list

Examples:
```yaml
allow: ["*"]
forbid: ["exit", "quit"]        # Allow all except exit/quit
```

### Processing Order

1. **Allow filtering**: Check if command is in allow list (or allow all)
2. **Forbid filtering**: Remove commands that are explicitly forbidden
3. **Final result**: Commands that pass both filters are executable

## Interface-Specific Examples

### Secure Web Interface

Prevent web users from terminating the daemon or accessing sensitive commands:

```yaml
acl:
  web:
    allow: ["fs", "net", "help", "crypto", "sys"]
    forbid: ["exit", "quit", "cfg"]
```

### Restricted RPC Access

Allow only read-only operations via RPC:

```yaml
acl:
  rpc:
    allow: ["fs ls", "net peers", "sys version", "help"]
    forbid: ["fs add", "cfg"]
```

### Terminal-Only Administration

Reserve administrative commands for direct terminal access:

```yaml
acl:
  terminal:
    allow: ["*"]
    forbid: []
  rpc:
    allow: ["*"] 
    forbid: ["cfg", "sys update"]
  web:
    allow: ["fs", "net", "help", "crypto"]
    forbid: ["exit", "quit", "cfg", "sys"]
```

## Available Commands

The following commands can be controlled via ACL:

### File System (`fs`)
- `fs ls` - List files
- `fs cp` - Download files
- `fs add` - Upload files (master nodes only)

### Network (`net`)
- `net peers` - List connected peers
- `net msg` - Send direct messages
- `net send` - Broadcast messages

### Cryptocurrency (`crypto`)
- `crypto gen` - Generate addresses
- `crypto val` - Validate addresses
- `crypto list` - List supported currencies

### Database (`db`)
- `db status` - Database information
- `db query` - Execute SQL queries

### Configuration (`cfg`)
- `cfg show` - Display configuration
- `cfg reload` - Reload configuration

### System (`sys`)
- `sys version` - Show version
- `sys update` - Check for updates

### General Commands
- `help` - Show help information
- `exit` - Exit application
- `quit` - Exit application (alias for exit)

## Security Considerations

### Web Interface Security

The default web ACL configuration forbids `exit` and `quit` because:

- Web interfaces may be accessible to multiple users
- Accidental daemon termination disrupts service for all users
- Administrative control should remain with terminal/RPC access

### Command Granularity

Currently, ACL operates at the **command group level** (`fs`, `net`, etc.), 
not subcommand level (`fs ls`, `fs add`).

For finer control:
- Use separate user accounts with different configurations
- Implement custom wrapper scripts
- Use system-level access controls (sudo, etc.)

### Performance

ACL checking is optimized for runtime performance:

- Command lists are processed into HashSets during startup
- Command validation uses O(1) hash lookups
- No performance impact on command execution

## Troubleshooting

### Command Denied Messages

When a command is blocked by ACL, you'll see:
```
Command 'exit' is not allowed for web interface
```

### Debugging ACL Issues

1. **Check configuration syntax**: Ensure YAML is valid
2. **Verify command names**: Use exact command names (`fs`, not `filesystem`)
3. **Test interfaces**: Commands may work in terminal but not RPC/web
4. **Check allow/forbid precedence**: Forbid always overrides allow

### Common Issues

**Issue**: Commands work in terminal but not web interface
**Solution**: Check web ACL configuration, add commands to allow list

**Issue**: All commands blocked after configuration change  
**Solution**: Ensure allow list contains `["*"]` or specific commands

**Issue**: Exit commands still work in web despite being forbidden
**Solution**: Verify configuration is loaded correctly, restart daemon

## Configuration Validation

Jantar validates ACL configuration on startup:

- **Valid command names**: Only existing commands are processed
- **Format validation**: YAML syntax must be correct
- **Default fallback**: Invalid configuration falls back to defaults

Invalid configurations log warnings but don't prevent startup.

## Migration from Legacy Configurations

If upgrading from older Jantar versions:

1. **Add ACL section**: New installations include ACL automatically
2. **Default behavior**: Without ACL section, all commands allowed everywhere  
3. **Gradual migration**: Start with default configuration, then customize

## Examples

### Development Configuration
```yaml
acl:
  terminal:
    allow: ["*"]
    forbid: []
  rpc: 
    allow: ["*"]
    forbid: []
  web:
    allow: ["*"] 
    forbid: ["exit", "quit"]
```

### Production Configuration
```yaml
acl:
  terminal:
    allow: ["*"]
    forbid: []
  rpc:
    allow: ["fs", "net", "crypto", "sys", "help"]
    forbid: ["cfg", "db"]
  web:
    allow: ["help", "sys version"]
    forbid: ["exit", "quit", "fs", "net", "crypto", "cfg", "db"]
```

### Read-Only Configuration  
```yaml
acl:
  terminal:
    allow: ["*"]
    forbid: []
  rpc:
    allow: ["fs ls", "net peers", "sys version", "help"]
    forbid: ["fs add", "net send", "net msg"]
  web:
    allow: ["help"]
    forbid: ["*"]
```

For more configuration options, see [Configuration Guide](configuration.md).