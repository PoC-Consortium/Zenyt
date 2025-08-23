# Jantar

A decentralized peer-to-peer file sharing and communication platform
built with Rust, libp2p, and iroh. Technology demonstrator and
explorative proof-of-concept that may serve as a reference 
implementation wallet for ZENYT cryptocurrency in the future.

**Current Version:** 0.2.1

## Features

- **Dual P2P Networking**: libp2p for messaging + iroh-blobs for file
  distribution
- **Cryptographically Signed Updates**: Ed25519 verification with 
  master key
- **Cryptocurrency Support**: Generate/validate addresses for ZENYT, 
  BURST, NXT, ARDOR
- **P2P Chat**: Direct and broadcast messaging with peer discovery
- **File Sharing**: Content-addressed storage with automatic replication
- **WebCLI Interface**: Browser-based interface alongside terminal CLI
- **Prometheus Metrics**: System monitoring and observability
- **GPU Acceleration**: Optional OpenCL support for crypto operations
- **NAT Traversal**: AutoNAT v2, STUN, and relay support

## Quick Start

### Installation

Download the latest release for your platform from the releases page.

### Basic Usage

1. **Start with default config**:
```bash
./jantar
```

2. **Start with custom config**:
```bash
./jantar -c mynode.cfg
```

3. **Sample configuration**:
```yaml
name: "MyNode"
network:
  port: 4001
  bootstrap_nodes: []
storage:
  download_dir: "~/Downloads/jantar"
metrics:
  enabled: true
  port: 9090
```

## Commands

Jantar uses hierarchical commands similar to git/docker:

### File System
- `fs ls [pattern]` - List available files
- `fs cp <hash> <filename>` - Download file
- `fs add <path>` - Share file (master nodes only)

### Network
- `net peers` - List connected peers  
- `net msg <peer> <message>` - Send direct message
- `net send <message>` - Broadcast message

### Cryptocurrency
- `crypto gen [type]` - Generate address (ZENYT, BURST, NXT, ARDOR)
- `crypto val <address>` - Validate address format
- `crypto list` - Show supported cryptocurrencies

### Database
- `db status` - Show database information
- `db query <sql>` - Execute SQL query

### System
- `sys version` - Show version information
- `sys update` - Check for updates

### Quick Commands
- `help [command]` - Show help
- `quit` / `exit` - Exit application

## Configuration

See [docs/user/configuration.md](docs/user/configuration.md) for 
complete options.

### Key Features
- **Hierarchical YAML**: Nested configuration structure
- **Flexible Booleans**: Supports yes/no, true/false, on/off
- **Bootstrap Nodes**: List of seed peers for network joining
- **Security**: Master key validation for updates
- **Access Control Lists**: Command restrictions per interface (terminal/RPC/web)

## Documentation

- [User Guide](docs/user/README.md) - Installation and usage
- [Command Reference](docs/user/commands.md) - Complete command list
- [Configuration](docs/user/configuration.md) - Config file format
- [Access Control](docs/user/acl-configuration.md) - Command access restrictions

## Contributing

Jantar is developed as a proprietary reference implementation. Community contributions are welcome through:

1. **Feature requests**: Submit through community discussion channels
2. **Bug reports**: Report issues through established communication platforms  
3. **Protocol improvements**: Participate in CIP (Coin Improvement Proposal) process
4. **Documentation**: Help improve user guides and technical documentation

Independent implementations following published specifications are encouraged.

## License

Experimental research software provided as technology preview. Use at 
your own risk with no warranties or guarantees.
