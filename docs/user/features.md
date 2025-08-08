# Jantar Features

## ✅ Implemented Features

### Core Networking (v0.1.0+)
- **P2P Network** - Decentralized peer-to-peer communication using libp2p
- **mDNS Discovery** - Automatic peer discovery on local networks
- **Bootstrap Nodes** - Network entry points for internet-scale connectivity
- **Gossipsub Messaging** - Efficient message propagation across the network
- **NAT Traversal** - Basic hole punching for NAT connectivity

### Messaging & Chat (v0.1.0+)
- **Direct Messaging** - Send messages to specific peers
- **Broadcast Messages** - Network-wide announcements
- **Message History** - Per-session message logging
- **Peer Names** - Human-readable peer identification
- **Emoji Support** - Full Unicode support in messages

### File Distribution (v0.1.5+)
- **Content-Addressed Storage** - Files identified by cryptographic hash
- **P2P File Sharing** - Distribute files across the network
- **Cryptographic Signatures** - All files signed by master nodes
- **Automatic Distribution** - Files propagate through the network
- **Manifest System** - Metadata for distributed files

### Self-Update System (v0.1.3+)
- **Automatic Updates** - Receive updates through P2P network
- **Version Detection** - Smart version comparison
- **Platform Detection** - Updates match your OS/architecture
- **Manual Updates** - Update to specific versions
- **Update Filtering** - Regex-based accept/ignore rules

### File Management (v0.1.8+)
- **List Command** - View available files with filtering
- **Copy Command** - Export blobs to filesystem
- **Remove Command** - Delete blobs from local storage
- **Pattern Matching** - Glob patterns for file listing
- **Storage Limits** - Configurable storage quotas

### Cryptocurrency Support (v0.1.9+)
- **Multi-Crypto Generation** - Generate addresses for multiple cryptocurrencies
- **Address Validation** - Verify cryptocurrency addresses
- **Supported Currencies**:
  - ZENYT (native, Ed25519 with Reed-Solomon)
  - Bitcoin (P2PKH addresses)
  - Ethereum (EIP-55 checksummed)
  - Litecoin, Dogecoin
  - BURST, NXT, ARDOR (Curve25519)

### Database Integration (v0.1.9+, Experimental)
- **ZENYT Blockchain** - Local blockchain database
- **Balance Tracking** - Check account balances
- **Transaction System** - Transfer between addresses
- **Colored Coins** - 16 different coin types
- **Rich List** - Top addresses by balance

### Network Analysis (v0.1.10+)
- **Network Crawler** - Map network topology (master nodes only)
- **Peer Exchange** - Differential peer list sharing
- **Connection Tracking** - Monitor peer connections
- **Geographic Mapping** - IP-based location detection

### TURN Relay Support (v0.1.11+)
- **CGNAT Support** - Connect through carrier-grade NAT
- **Symmetric NAT** - Relay for restrictive NATs
- **Automatic Detection** - Identify when relay is needed
- **STUN/TURN Server** - Standard protocol support
- **Configurable Relay** - Enable/disable as needed

### Observability (v0.1.11+)
- **Prometheus Metrics** - Complete metrics collection
- **Connection Metrics** - Track success/failure rates
- **Performance Metrics** - Latency, throughput monitoring
- **Network Health** - CGNAT detection, relay usage
- **HTTP Endpoint** - `/metrics` for Prometheus scraping

### Security Features (v0.1.11+)
- **Rate Limiting** - Connection and message flood protection
- **Peer Blacklisting** - Block malicious peers
- **File Size Limits** - Min/max file size restrictions
- **Storage Quotas** - Prevent disk exhaustion
- **Signature Verification** - Cryptographic file verification

### CLI Features
- **Command History** - Per-node history files
- **Tab Completion** - Command prefix matching
- **Colored Output** - Visual feedback with colors
- **Batch Mode** - Execute commands without interactive mode
- **Debug Logging** - Detailed logs for troubleshooting

### Configuration
- **YAML Configuration** - Human-readable config files
- **Environment Variables** - Override config via env
- **Hot Reload** - Some settings apply without restart
- **Multiple Configs** - Run multiple nodes with different configs
- **Sensible Defaults** - Works out of the box

## 🔄 Version History

### v0.1.11 (Current)
- TURN relay server integration
- Prometheus metrics
- Rate limiting
- Improved peer deduplication

### v0.1.10
- Network crawler for topology mapping
- Enhanced peer exchange protocol
- Connection status tracking

### v0.1.9
- Multi-cryptocurrency support
- ZENYT database integration
- Colored coins system

### v0.1.8
- File management commands (list, copy, remove)
- Pattern matching for file operations
- Storage management

### v0.1.7
- Bootstrap node improvements
- Better NAT traversal
- Enhanced logging

### v0.1.6
- Improved update system
- Better error handling
- Performance optimizations

### v0.1.5
- iroh-blobs integration
- Content-addressed storage
- Manifest system

### v0.1.3
- Self-update system
- Version management
- Platform detection

### v0.1.0
- Initial release
- Basic P2P networking
- Chat functionality

## 🚀 Upcoming Features

See [Development Roadmap](../development/TODO.md) for planned features.

## 💡 Feature Requests

Have an idea for a new feature? Please:
1. Check the [TODO list](../development/TODO.md) first
2. Search [existing issues](https://github.com/yourrepo/issues)
3. Open a [feature request](https://github.com/yourrepo/issues/new)

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)