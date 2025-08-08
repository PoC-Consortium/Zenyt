# Commands Reference

Complete reference for all Jantar commands. All commands support prefix-based matching (e.g., `h` for `help`, `pe` for `peers`).

## 📋 Command Categories

- [General Commands](#general-commands)
- [Network Commands](#network-commands)
- [File Commands](#file-commands)
- [Crypto Commands](#crypto-commands)
- [Database Commands](#database-commands)
- [Configuration Commands](#configuration-commands)
- [Master-Only Commands](#master-only-commands)

## General Commands

### help
Show all available commands
```
help
```

### version
Display current version and build information
```
version
```
Output: `📌 Jantar version 0.1.11 (linux-x86_64)`

### quit / exit
Exit the application gracefully
```
quit
exit
q
```

## Network Commands

### peers
List all connected and connecting peers
```
peers
```
Shows:
- ✅ Connected peers with addresses
- ⏳ Connecting peers with attempt count

### msg
Send a direct message to a peer
```
msg <peer_id> <message>

# Examples:
msg 12D3KooW... Hello!
msg 12D3KooW... How are you today?
```

### send
Send a file directly to a peer
```
send <peer_id> <file_path>

# Examples:
send 12D3KooW... document.pdf
send 12D3KooW... ~/Downloads/image.png
send 12D3KooW... "/path/with spaces/file.txt"
```

## File Commands

### list
List files available in the network
```
list [options] [pattern]

Options:
  -a    Show only announced files (network)
  -l    Show only local blobs

# Examples:
list                      # All files
list -a                   # Network files only
list *.pdf               # Files matching pattern
list -a jantar-*.AppImage # Updates only
```

### copy
Export a blob from storage to filesystem
```
copy <hash> <destination_path>

# Examples:
copy abc123... output.pdf
copy abc123... ~/Downloads/file.txt
copy abc123... "/path/with spaces/file.pdf"
```

### remove
Remove a blob from local storage
```
remove <hash>

# Example:
remove abc123def456
```

## Crypto Commands

### generate
Generate cryptocurrency addresses
```
generate [<crypto>] [<private_key_or_passphrase>]

Supported cryptocurrencies:
  - ZENYT (native)
  - BTC/Bitcoin
  - LTC/Litecoin
  - DOGE/Dogecoin
  - ETH/Ethereum
  - BURST
  - NXT
  - ARDOR

# Examples:
generate                          # Random ZENYT
generate btc                      # Random Bitcoin
generate eth "my passphrase"      # ETH from passphrase
generate btc 0x1234567890abcdef   # BTC from hex key
generate btc L1uyy5qTuGrVX...    # BTC from WIF
```

### validate
Validate cryptocurrency addresses
```
validate <address>

# Examples:
validate ZENYT-WG7RZ-YMZ73-XP5MZ-NZMCW-A5NPA-AR783Z
validate 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
validate 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
validate BURST-4WCG-CNKL-N644-6DPND
```

## Database Commands

### db-init
Initialize or connect to ZENYT database
```
db-init [path]

# Examples:
db-init                    # Default location
db-init /opt/zenyt.db      # Custom location
db-init ~/blockchain.db    # User directory
```

### db-status
Show current database connection status
```
db-status
```

### balance
Check balance of a ZENYT address
```
balance <address>

# Example:
balance ZENYT-GENES-IS111-11111-11111-111111
```

### transfer
Transfer ZENYT between addresses
```
transfer <from_address> <to_address> <amount>

# Example:
transfer ZENYT-ADDR1... ZENYT-ADDR2... 1000000
```
Amount is in Planck (1 ZENYT = 100,000,000 Planck)

### richlist
Show top addresses by balance
```
richlist [coin] [limit]

# Examples:
richlist                  # Top 20 all coins
richlist 50              # Top 50 all coins
richlist ZENYT           # Top 20 ZENYT only
richlist ZCC07 30        # Top 30 of colored coin 07
```

### db-stats
Show database statistics
```
db-stats
```
Shows total addresses, transactions, and supply info

### db-seed
Seed database with test addresses
```
db-seed [coin] [count]

# Examples:
db-seed                   # 20 ZENYT addresses
db-seed 10               # 10 ZENYT addresses
db-seed ZTSTN            # 20 test network addresses
db-seed ZCC05 15         # 15 colored coin addresses
```
⚠️ Creates real keypairs - logs saved to `seed_log_*.txt`

### db-query
Query database by various criteria
```
db-query <criteria> [params]

# Examples:
db-query 1000000 10000000           # Amount range
db-query date-before 1640995200     # Before timestamp
db-query date-after 1640995200      # After timestamp
db-query consistency                # Check all coins
db-query consistency ZENYT          # Check specific coin
```

## Configuration Commands

### config
Manage configuration settings
```
config <subcommand> [args]

Subcommands:
  show              # Display current config
  get <key>         # Get specific value
  set <key> <value> # Set configuration value
  save              # Save changes to file
  reload            # Reload from file

# Examples:
config show
config get name
config set name "New Name"
config set auto_update true
config save
```

## Update Commands

### update
Check for and apply updates
```
update [version]

# Examples:
update                    # Check for latest
update 0.1.10            # Update to specific version
```

## Master-Only Commands

These commands require a valid master private key in configuration.

### add-file
Add a file to P2P distribution network
```
add-file <file_path>

# Examples:
add-file document.pdf
add-file ~/uploads/image.png
add-file "jantar-0.1.11-x86_64.AppImage"
```

### push-update
Push update package from directory
```
push-update <directory>

# Example:
push-update ./release/
```

### crawl
Crawl and map network topology
```
crawl [options] [depth] [output_file]

Options:
  +ghosts     Include non-responsive nodes
  json        Output in JSON format
  graphml     Output in GraphML format

# Examples:
crawl                     # Default: depth 5, JSON
crawl +ghosts            # Include ghost nodes
crawl 10                 # Depth 10
crawl 5 network.json     # Save to file
crawl +ghosts 8 map.graphml  # Full options
```

## Command Shortcuts

Most commands can be abbreviated to their unique prefix:

| Full Command | Shortcuts |
|-------------|-----------|
| help | h, he, hel |
| quit | q, qu, qui |
| peers | p, pe, pee, peer |
| message | m, ms, msg |
| list | l, li, lis |
| version | v, ve, ver, vers, versi, versio |
| update | u, up, upd, upda, updat |
| generate | g, ge, gen, gene, gener, genera, generat |
| validate | val, vali, valid, valida, validat |

## Batch Mode

Execute commands without entering interactive mode:

```bash
# Single command
./jantar -e "peers"
./jantar --execute "version"

# With custom config
./jantar -c alice.cfg -e "list"

# Useful for scripts
PEERS=$(./jantar -e "peers" | grep -c "Connected")
VERSION=$(./jantar -e "version" | grep -oP '\d+\.\d+\.\d+')
```

## Command Pipeline Examples

### Monitor Network Health
```bash
#!/bin/bash
while true; do
  echo "Peers: $(./jantar -e 'peers' | grep -c '✅')"
  echo "Files: $(./jantar -e 'list' | grep -c '📄')"
  sleep 60
done
```

### Batch Address Generation
```bash
for i in {1..10}; do
  ./jantar -e "generate zenyt passphrase_$i"
done > addresses.txt
```

### Automated File Distribution
```bash
for file in *.pdf; do
  ./jantar -c master.cfg -e "add-file $file"
  sleep 5
done
```

## Error Messages

Common error messages and their meanings:

| Error | Meaning |
|-------|---------|
| "Unknown command" | Command not recognized, check spelling |
| "Ambiguous command" | Multiple commands match, be more specific |
| "Usage: ..." | Missing required parameters |
| "Only verified master nodes..." | Command requires master privileges |
| "Iroh node not available" | File storage system not initialized |
| "No peers found yet" | Not connected to any peers |
| "File not found" | Specified file doesn't exist |
| "Blob not found" | Hash doesn't match any stored content |

## Tips

1. **Use Tab Completion** (coming soon) - Press Tab to auto-complete commands
2. **Command History** - Use ↑/↓ arrows to navigate previous commands
3. **Prefix Matching** - Type just enough letters to uniquely identify a command
4. **Batch Processing** - Use `-e` for scripting and automation
5. **Check Help** - When in doubt, use `help` command

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)