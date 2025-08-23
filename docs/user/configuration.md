# Configuration Guide

Jantar uses YAML configuration files to customize node behavior. This guide covers all configuration options.

## 📁 Configuration File Location

Default: `jantar.cfg` in the current directory

Specify custom config:
```bash
./jantar -c myconfig.yaml
./jantar --config /path/to/config.yaml
```

## 🎯 Quick Start Config

Minimal configuration that works out of the box:

```yaml
# jantar.cfg
name: "Alice"
```

That's it! Everything else has sensible defaults.

## 📋 Complete Configuration Reference

### Basic Settings

```yaml
# Node name - how you appear to other peers
name: "Alice"

# Network port (0 = auto-assign)
port: 0

# Logging level: debug, info, warn, error
log_level: "info"
```

### Storage Settings

```yaml
# Where to store P2P data (iroh blobs)
iroh_storage_path: "iroh_storage/alice"

# Where to save received files
download_dir: "~/Downloads/jantar"

# Maximum storage size in bytes (10GB default)
max_storage_size: 10737418240

# Storage limit strategy when full
storage_limit_strategy: "remove_oldest"  # or "ignore_newest"

# File size restrictions (bytes)
min_file_size: 1024           # 1KB minimum
max_file_size: 1073741824     # 1GB maximum
```

### Network Settings

```yaml
# Bootstrap nodes for internet connectivity
bootstrap_nodes:
  - "/ip4/1.2.3.4/tcp/4001/p2p/12D3KooW..."
  - "/ip4/5.6.7.8/tcp/443/p2p/12D3KooW..."

# Enable mDNS for local network discovery
mdns_enabled: true

# Connection limits
max_connections: 50
connection_timeout: 30

# TURN relay for CGNAT/NAT traversal
turn_enabled: false
turn_server: "turn:relay.example.com:3478"
turn_username: "user"
turn_password: "pass"
```

### Update Settings

```yaml
# Enable automatic updates
auto_update: true

# Regex filter for accepting updates
update_accept: "^jantar-\\d+\\.\\d+\\.\\d+-(x86_64|aarch64)\\.(AppImage|exe)$"

# Regex filter for ignoring updates
update_ignore: ""

# Examples of update filters:
# Only AppImages (no Windows):
# update_accept: "^jantar-.*\\.AppImage$"

# Ignore beta versions:
# update_ignore: ".*(beta|rc).*"

# Only your architecture:
# update_accept: "^jantar-.*-x86_64\\..*$"  # Intel/AMD
# update_accept: "^jantar-.*-aarch64\\..*$" # ARM64
```

### Security Settings

```yaml
# Peer blacklist
peer_blacklist:
  - "12D3KooWBadPeer1234567890"
  - "12D3KooWMaliciousPeer98765"

# Rate limiting
rate_limit_enabled: true
max_messages_per_minute: 60
max_connections_per_minute: 5

# Signature verification (always enabled)
verify_signatures: true
```

### Master Node Settings

```yaml
# Master nodes can distribute files and updates
is_master: false

# Base64-encoded Ed25519 private key
# Only set this if you're a verified master
master_private_key: "MC4CAQAwBQYDK2VwBCIEI..."

# Masters typically don't auto-update
auto_update: false
```

### Database Settings (Experimental)

```yaml
# Enable blockchain database
database_enabled: false

# Database path (uses platform defaults if not set)
database_path: "/opt/jantar/zenyt.db"

# Auto-connect on startup
database_auto_connect: true
```

### Web Server Settings

```yaml
# Web server with granular feature control
web:
  enabled: true           # Enable web server
  port: 9090             # HTTP port for all web features
  bind_address: "127.0.0.1"  # Bind address (localhost for security)
  prometheus: true       # Enable /metrics endpoint
  dashboard: true        # Enable / interactive dashboard
  api: true             # Enable /api/* JSON endpoints
  cli: false            # Enable /cli/ web interface (disabled by default)
```

When web server is enabled, the following become available based on feature toggles:
- **Web Dashboard**: `http://localhost:9090/` - Interactive monitoring interface (if dashboard: true)
- **Prometheus Metrics**: `http://localhost:9090/metrics` - Prometheus format metrics (if prometheus: true)
- **System API**: `http://localhost:9090/api/system` - System resource data (if api: true)
- **Network API**: `http://localhost:9090/api/network` - P2P network statistics (if api: true)
- **Blockchain API**: `http://localhost:9090/api/blockchain` - ZENYT transaction data (if api: true)
- **Web CLI**: `http://localhost:9090/cli/` - Web-based command interface (if cli: true)


See [prometheus-metrics.md](prometheus-metrics.md) for detailed metrics documentation.

### Advanced Settings

```yaml
# Worker threads (0 = auto-detect CPU cores)
worker_threads: 0

# Enable experimental features
experimental_features:
  - "colored_coins"
  - "network_crawler"

# Custom relay server settings
relay_enabled: false
relay_url: "https://relay.example.com"

# DHT settings (future)
dht_enabled: false
dht_bootstrap_nodes: []
```

## 🎨 Configuration Examples

### Minimal Node
```yaml
name: "SimpleNode"
```

### Local Testing Node
```yaml
name: "TestNode"
port: 4002
iroh_storage_path: "test_storage"
log_level: "debug"
auto_update: false
```

### GPU-Accelerated Node
```yaml
name: "GPUNode"
gpu:
  enabled: true              # Enable GPU acceleration
  backend: "auto"            # auto|opencl|cpu_only
  device_index: 0            # Which GPU to use (0 = first)
  max_memory_mb: 1024        # Max GPU memory to use (MB)
  batch_size: 1000           # Operations per batch
  fallback_on_error: true    # Fall back to CPU on errors
  operations:
    hashing: true            # GPU for hashing
    address_generation: true # GPU for address generation
    signature_verification: true # GPU for signatures
```

### Production Node (Secure)
```yaml
name: "ProductionNode"
network:
  port: 4001
  bootstrap_nodes:
    - "/ip4/1.2.3.4/tcp/4001/p2p/12D3KooW..."
storage:
  max_size: 107374182400  # 100GB
update:
  auto: true
  accept: "^jantar-\\d+\\.\\d+\\.\\d+-x86_64\\.AppImage$"
web:
  enabled: true
  prometheus: true    # Metrics for monitoring
  dashboard: true     # Dashboard for ops team
  api: true          # API for external tools
  cli: false         # DISABLED for security
log_level: "warn"
```

### Development Node (All Features)
```yaml
name: "DevNode"
network:
  port: 0
  enable_mdns: true
web:
  enabled: true
  prometheus: true
  dashboard: true
  api: true
  cli: true          # ENABLED for development
update:
  auto: false        # Manual updates in development
log_level: "debug"
```

### Master Node
```yaml
name: "Master"
is_master: true
master_private_key: "MC4CAQAwBQYDK2VwBCIEI..."
auto_update: false
iroh_storage_path: "master_storage"
log_level: "info"
```

### CGNAT/Mobile Network Node
```yaml
name: "MobileNode"
turn_enabled: true
turn_server: "turn:relay.example.com:3478"
turn_username: "user"
turn_password: "pass"
bootstrap_nodes:
  - "/ip4/relay.example.com/tcp/443/p2p/12D3KooW..."
```

### Database-Enabled Node
```yaml
name: "BlockchainNode"
database_enabled: true
database_path: "/var/lib/jantar/blockchain.db"
database_auto_connect: true
```

## 🔧 Runtime Configuration

### Using config Command

View current configuration:
```
[Alice]> config show
```

Get specific value:
```
[Alice]> config get name
[Alice]> config get auto_update
```

Set configuration value:
```
[Alice]> config set name "NewName"
[Alice]> config set auto_update true
[Alice]> config set log_level debug
```

Save changes:
```
[Alice]> config save
```

Reload from file:
```
[Alice]> config reload
```

### Environment Variables

Override config with environment variables:

```bash
# Override node name
JANTAR_NAME="EnvNode" ./jantar

# Override port
JANTAR_PORT=4002 ./jantar

# Override log level
JANTAR_LOG_LEVEL=debug ./jantar

# Multiple overrides
JANTAR_NAME="Test" JANTAR_PORT=4003 ./jantar
```

### Command-Line Overrides

Command-line arguments take precedence over config and env:

```bash
# Override name
./jantar -n "CLINode"

# Override config file and name
./jantar -c custom.cfg -n "Override"
```

Priority order (highest to lowest):
1. Command-line arguments
2. Environment variables
3. Configuration file
4. Default values

## 📝 Configuration Best Practices

### 1. Use Descriptive Names
```yaml
# Good
name: "Alice-Laptop"
name: "Server-EU-1"

# Bad
name: "Node1"
name: "Test"
```

### 2. Separate Configs for Testing
```yaml
# production.cfg
name: "Production"
auto_update: true
log_level: "warn"

# development.cfg
name: "Dev"
auto_update: false
log_level: "debug"
```

### 3. Secure Master Keys
```yaml
# Never commit master keys to version control!
# Use environment variable instead:
master_private_key: "${MASTER_KEY}"
```

### 4. Set Appropriate Limits
```yaml
# For limited disk space
max_storage_size: 1073741824  # 1GB

# For high-traffic nodes
max_connections: 100
rate_limit_enabled: true
```

### 5. Document Custom Settings
```yaml
# Bootstrap nodes for company network
# Contact: admin@example.com
bootstrap_nodes:
  - "/ip4/10.0.1.5/tcp/4001/p2p/12D3KooW..."  # Main server
  - "/ip4/10.0.2.5/tcp/4001/p2p/12D3KooW..."  # Backup
```

## 🔍 Validating Configuration

Check if config is valid:
```bash
# Jantar validates on startup
./jantar -c myconfig.yaml

# Check specific settings
./jantar -c myconfig.yaml -e "config show"
```

Common validation errors:
- Invalid YAML syntax
- Invalid port number (must be 0-65535)
- Invalid log level (must be debug/info/warn/error)
- Invalid regex patterns in update filters
- Invalid bootstrap node addresses

## 🆘 Troubleshooting Config Issues

### Config Not Loading
- Check file exists: `ls jantar.cfg`
- Check YAML syntax: Use online YAML validator
- Check permissions: `chmod 644 jantar.cfg`

### Settings Not Applied
- Some settings require restart
- Check priority (CLI > env > config)
- Use `config show` to verify

### Bootstrap Nodes Not Working
- Ensure correct multiaddr format
- Include peer ID in address
- Check network connectivity

### Update Filters Not Working
- Test regex patterns online
- Escape special characters properly
- Check update_accept before update_ignore

## 📚 See Also

- [Getting Started](getting-started.md) - Basic usage
- [Commands Reference](commands.md) - All commands
- [Admin Guide](../admin/README.md) - Advanced deployment

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)