# Getting Started

Welcome to Jantar! This guide will help you get up and running quickly.

## 🚀 Quick Start

### 1. Start Your First Node

```bash
# Run with default settings
./jantar

# You'll see:
[Alice]> 
```

### 2. Check Your Connections

```bash
[Alice]> peers

Connected:
  ✅ 12D3KooWCUsopTQM7Bu9apwTWgyR6Ftkmh7vxe1mgHUB4Wg3uWku
    📍 /ip4/192.168.1.100/tcp/4001
```

### 3. Send Your First Message

```bash
[Alice]> msg 12D3KooW... Hello from Alice!
```

### 4. List Available Files

```bash
[Alice]> list

📢 Announced files:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 welcome.txt
   Size: 42 bytes
   Type: text/plain
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 🎮 Command-Line Options

### Interactive Mode (Default)
```bash
# Start interactive CLI
./jantar

# With custom config
./jantar -c alice.cfg

# With custom name
./jantar -n "Alice Node"
```

### Batch Mode
Execute single commands without entering interactive mode:

```bash
# Check version
./jantar -e "version"

# List peers
./jantar -e "peers"

# Send message
./jantar -e "msg 12D3KooW... Hello"

# Generate crypto address
./jantar -e "generate btc"
```

### Combining Options
```bash
# Use custom config and name, execute command
./jantar -c alice.cfg -n "Alice" -e "peers"
```

## 🌐 Network Discovery

### Local Network (mDNS)
Nodes automatically discover each other on the same local network:
- No configuration required
- Works through most home routers
- Instant discovery (< 1 second)

### Internet Scale (Bootstrap Nodes)
To connect across the internet, configure bootstrap nodes:

```yaml
# In your config file
bootstrap_nodes:
  - "/ip4/1.2.3.4/tcp/4001/p2p/12D3KooW..."
  - "/ip4/5.6.7.8/tcp/443/p2p/12D3KooW..."
```

## 💬 Messaging

### Direct Messages
Send messages to specific peers:

```bash
# Using full peer ID
msg 12D3KooWCUsopTQM7Bu9apwTWgyR6Ftkmh7vxe1mgHUB4Wg3uWku Hello!

# Message with spaces
msg 12D3KooW... "Hello there, how are you?"
```

### Receiving Messages
Messages appear automatically in your console:

```
💬 [Bob]: Hello Alice!
```

## 📁 File Sharing

### Viewing Files
```bash
# List all available files
list

# List only network files
list -a

# List with pattern matching
list *.pdf
list jantar-*.AppImage
```

### Copying Files
Export files from P2P storage to your filesystem:

```bash
# Copy to current directory
copy abc123def456 ./myfile.pdf

# Copy to specific path
copy abc123def456 ~/Downloads/document.pdf
```

### Sending Files Directly
Send files to specific peers:

```bash
# Send a file
send 12D3KooW... ~/Documents/report.pdf

# File with spaces in name
send 12D3KooW... "~/My Documents/important file.pdf"
```

## 🔄 Self-Updates

### Check for Updates
```bash
[Alice]> update
🔍 Checking for updates...
✅ You are running the latest version
```

### When Updates Are Available
```bash
[Alice]> update
🔍 Checking for updates...
🎉 Update available: 0.1.10 -> 0.1.11
💡 Run 'update' again to download and install
```

### Auto-Update Configuration
Enable automatic updates in your config:

```yaml
# jantar.cfg
auto_update: true
```

## 🔐 Cryptocurrency Features

### Generate Addresses
```bash
# Random ZENYT address
generate

# Random Bitcoin address
generate btc

# From passphrase
generate zenyt "my secret phrase"
generate burst "another passphrase"

# From private key (hex)
generate eth 0x1234567890abcdef...
```

### Validate Addresses
```bash
# Automatically detects cryptocurrency type
validate 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
validate ZENYT-WG7RZ-YMZ73-XP5MZ-NZMCW-A5NPA-AR783Z
validate 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

## 📊 Database Features (Experimental)

### Initialize Database
```bash
# Use default location
db-init

# Use custom location
db-init /opt/zenyt.db
```

### Check Balances
```bash
# Check address balance
db balance ZENYT-GENES-IS111-11111-11111-111111

# Show top addresses (richlist)
db top
db top balance 50
db top age-in 20        # Addresses inactive longest
db top age-out 10 ZENYT # Recently active ZENYT senders
```

### Transfer Funds
```bash
# Transfer between addresses
db transfer ZENYT-FROM1 ZENYT-TO222 1000000
```

## ⌨️ Keyboard Shortcuts

- **↑/↓** - Navigate command history
- **Tab** - Command completion (coming soon)
- **Ctrl+C** - Cancel current input
- **Ctrl+D** - Exit application

## 🎯 Tips for New Users

### 1. Start Simple
- First, just run `jantar` and explore
- Use `help` to see all commands
- Try `peers` to see who's online

### 2. Set Your Name
Edit `jantar.cfg`:
```yaml
name: "YourNameHere"
```

### 3. Join the Network
- Local peers are discovered automatically
- For internet connectivity, ask for bootstrap node addresses

### 4. Experiment Safely
- Use test configs for experimentation
- Keep backups of important configs
- Monitor logs if issues occur

## 🔍 Common Patterns

### Morning Routine
```bash
# Start node
./jantar

# Check connections
peers

# Check for updates
update

# List new files
list
```

### Batch Monitoring
```bash
#!/bin/bash
# monitoring.sh
./jantar -e "peers" | grep -c "Connected"
./jantar -e "version"
./jantar -e "list" | grep -c "📄"
```

### Multiple Nodes Testing
```bash
# Terminal 1
./jantar -c alice.cfg

# Terminal 2
./jantar -c bob.cfg

# They'll discover each other automatically!
```

## ❓ Getting Help

### In-Application Help
```bash
[Alice]> help
```

### Check Version and Platform
```bash
[Alice]> version
📌 Jantar version 0.1.11 (linux-x86_64)
   Built: 2025-08-08
```

### Enable Debug Logging
Edit config:
```yaml
log_level: "debug"
```

Then check `logs/` directory for detailed information.

## 📚 Next Steps

- [Commands Reference](commands.md) - Detailed command documentation
- [Configuration](configuration.md) - Customize your node
- [File Sharing](file-sharing.md) - Advanced file distribution
- [Troubleshooting](troubleshooting.md) - Common issues and solutions

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)