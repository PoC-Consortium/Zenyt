# Commands Reference

Complete reference for all Jantar commands. 

## Hierarchical Command Structure

Jantar uses a hierarchical command structure similar to
git/docker. Commands are organized into groups with subcommands.

## Prefix-based disambiguation

A command can be issued by the shortest unique prefix. So a command
like `quit` can be issued by simply typing `q` as long as there is no
other command starting with `q`. Assume there was a command `query`
(there isn't), then you would have to type `qui` to achieve a command
disambiguation and `que` for the (non existing) `query` command.

This mechanism is here merely for comfort purposes of users who
interactively type in CLI commands. We highly recommend to use
full-length commands in scripts and documentation as best practice.

The addition of new commands may render a previously working shortest
prefix ambiguous thus non-working.

## 📋 Command Groups

- `fs` - File system operations
- `net` - Network operations
- `crypto` - Cryptocurrency operations
- `db` - Database operations
- `cfg` - Configuration management
- `sys` - System operations

## General Commands

### help
Show help information
```
help                 # Show general help
help <command>       # Show help for specific command

# Examples:
help fs              # Show help for fs commands
help net             # Show help for network commands
```

### quit / exit
Exit the application
```
quit
exit
```
Also works: Ctrl+D

## File System Commands (`fs`)

### fs ls
List files in P2P storage
```
fs ls [pattern]

# Examples:
fs ls                # List all files
fs ls *.pdf          # List PDF files
fs ls jantar-*       # List Jantar updates
```

### fs cp
Copy file from P2P storage to filesystem
```
fs cp <hash> <destination>

# Examples:
fs cp abc123 output.pdf
fs cp abc123 ~/Downloads/file.txt
```

### fs rm
Remove file from local storage
```
fs rm <hash>

# Example:
fs rm abc123def456
```

### fs add
Add file to P2P network (master only)
```
fs add <file_path>

# Examples:
fs add document.pdf
fs add ~/uploads/image.png
```

### fs info
Show detailed file information
```
fs info <hash_or_filename>

# Examples:
fs info abc123
fs info document.pdf
```

## Network Commands (`net`)

### net peers
List connected peers
```
net peers
net                   # Default shows peers
```

### net msg
Send message to peer
```
net msg <peer> <message>

# Examples:
net msg Alice Hello!
net msg 12D3KooW... "How are you?"
```

### net send
Send file to peer
```
net send <peer> <file>

# Examples:
net send Bob document.pdf
net send Alice ~/Downloads/image.png
```

### net crawl
Crawl network topology (master only)
```
net crawl [+ghosts] [depth] [output_file]

# Examples:
net crawl                     # Default depth 5
net crawl +ghosts             # Include ghost nodes
net crawl 10 network.json     # Save to file
```

## Cryptocurrency Commands (`crypto`)

### crypto gen
Generate cryptocurrency address
```
crypto gen [<type>] [<seed>]
crypto                        # Default generates ZENYT

# Examples:
crypto gen                    # Generate ZENYT address
crypto gen BURST              # Generate BURST address
crypto gen NXT passphrase     # Generate NXT from seed
```

Supported types: ZENYT, BURST, NXT, ARDOR

### crypto val
Validate cryptocurrency address
```
crypto val <address>

# Examples:
crypto val ZENYT-WG7RZ-YMZ73-XP5MZ-NZMCW-A5NPA-AR783Z
crypto val BURST-4WCG-CNKL-N644-6DPND
```

### crypto list
List supported cryptocurrencies
```
crypto list
```

## Database Commands (`db`)

### db init
Initialize database
```
db init [path]

# Examples:
db init                    # Default location
db init /opt/zenyt.db      # Custom location
```

### db status
Show database status
```
db status
db                         # Default shows status
```

### db balance
Check balance of an address
```
db balance <address>

# Example:
db balance ZENYT-GENES-IS111-11111-11111-111111
```

### db transfer
Transfer ZENYT between addresses
```
db transfer <from> <to> <amount>

# Example:
db transfer ZENYT-ADDR1... ZENYT-ADDR2... 1000000
```
Amount is in Planck (1 ZENYT = 100,000,000 Planck)

### db top
Query top addresses by various criteria
```
db top [criterion] [limit] [coin]

# Arguments (positional, in order):
# criterion - What to sort by (default: balance)
# limit     - Number of results (default: 20, max: 100)  
# coin      - Filter by coin type (default: all)

# Available criteria:
# balance   - Top addresses by balance (richlist)
# age-in    - Addresses that haven't received funds in longest time
# age-out   - Addresses that sent funds most recently
# oldest    - Oldest addresses by creation date
# newest    - Newest addresses by creation date

# Examples:
db top                      # Top 20 by balance
db top balance 50           # Top 50 richest
db top age-in               # 20 addresses inactive longest (receiving)
db top age-out 30 ZENYT     # 30 ZENYT addresses that sent most recently
db top oldest 10            # 10 oldest addresses by creation
db top newest 5             # 5 newest addresses

# Note: 'db richlist' still works as legacy alias for 'db top balance'
```

### db stats
Show database statistics
```
db stats
```

### db seed
Seed database with test data (development)
```
db seed [<coin>] [<count>]

# Examples:
db seed                 # 20 ZENYT addresses
db seed ZENYT 10        # 10 ZENYT addresses
```

### db query
Query database by criteria
```
db query <criteria> [params]

# Examples:
db query 1000000 10000000           # Amount range
db query date-before 1640995200     # Before timestamp
db query consistency                # Check all coins
```

## Configuration Commands (`cfg`)

### cfg show
Display current configuration
```
cfg show
cfg                    # Default shows config
```

### cfg get
Get configuration value
```
cfg get <key>

# Example:
cfg get name
```

### cfg set
Set configuration value
```
cfg set <key> <value>

# Examples:
cfg set name "Alice"
cfg set auto_update true
```

### cfg save
Save configuration to file
```
cfg save
```

### cfg reload
Reload configuration from file
```
cfg reload
```

## System Commands (`sys`)

### sys status
Show system status
```
sys status
sys                    # Default shows status
```

### sys version
Show version information
```
sys version
```

### sys update
Check for and apply updates
```
sys update [<version>]

# Examples:
sys update             # Check for latest
sys update 0.1.10      # Update to specific version
```

## Command Shortcuts

Commands support prefix matching - type just enough letters to uniquely identify:

| Full Command | Shortcuts |
|-------------|-----------|
| `help` | `h`, `he`, `hel` |
| `quit` | `q`, `qu`, `qui` |
| `fs ls` | `fs l`, `fs li` |
| `net peers` | `net p`, `net pe` |
| `crypto gen` | `crypto g`, `cry g` |

## Batch Mode

Execute commands without entering interactive mode:

```bash
# Single command
./jantar -e "fs ls"
./jantar --execute "net peers"

# With custom config
./jantar -c alice.cfg -e "sys version"

# Useful for scripts
FILES=$(./jantar -e "fs ls" | grep -c "📄")
```

## Tips

1. **Use `help <command>`** - Get detailed help for any command group
2. **Command History** - Use ↑/↓ arrows to navigate previous commands
3. **Prefix Matching** - Type just enough letters to uniquely identify
4. **Batch Processing** - Use `-e` for scripting and automation
5. **Exit Options** - Use `quit`, `exit`, or Ctrl+D to leave

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)
