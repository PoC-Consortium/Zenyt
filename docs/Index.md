# Jantar Documentation

## ⚠️ WORK IN PROGRESS DISCLAIMER

**This project is under active development (v0.2.0-dev). Use at your own risk.**

- 🚧 **Experimental software** - Not suitable for production use
- 📝 **Documentation inconsistencies** expected during rapid development
- 🔬 **Breaking changes** may occur between versions  
- 🧪 **Testing in progress** - Some features may be unstable
- 💾 **No guarantees** of data persistence or backward compatibility

---

Welcome to the Jantar documentation. Jantar is a peer-to-peer (P2P) chat and file-sharing application with self-update capabilities, built with cutting-edge P2P technologies.

## 📚 Documentation Sections

### [User Guide](user/README.md)
For end users who want to use Jantar for P2P communication and file sharing.
- Installation and setup
- Basic usage and commands
- Configuration options
- Troubleshooting

### [Administrator Guide](admin/README.md)
For system administrators and node operators.
- Bootstrap node setup
- Network deployment
- System service configuration
- Monitoring and metrics
- Security considerations

### [Developer Guide](development/README.md)
For developers contributing to or building on Jantar.
- Architecture overview
- Building from source
- Testing guidelines
- CI/CD pipeline
- Contributing guidelines

## 🚀 Quick Start

### For Users
1. Download the latest AppImage (Linux) or Windows executable from the official releases page
2. Make it executable: `chmod +x jantar-*.AppImage`
3. Run: `./jantar-*.AppImage`
4. Type `help` to see available commands

### For Developers
Jantar is developed as a proprietary reference implementation. Developer access is limited to:
- Protocol specifications and documentation
- Community discussion and feedback
- Feature requests through established channels

### For Administrators
```bash
# Set up as systemd service (Linux)
sudo cp jantar.service /etc/systemd/system/
sudo systemctl enable jantar
sudo systemctl start jantar

# Monitor metrics (if enabled)
curl http://localhost:9090/metrics
```

## 🌟 Key Features

- **Decentralized P2P Network** - No central server required
- **End-to-End Encrypted Chat** - Secure messaging between peers
- **File Sharing & Distribution** - Content-addressed file storage using iroh-blobs
- **Self-Update System** - Automatic updates through P2P network
- **TURN Relay Support** - Connect through CGNAT/symmetric NAT (v0.1.11+)
- **Prometheus Metrics** - Complete observability (v0.1.11+)
- **GPU Acceleration** - OpenCL-based crypto operations with CPU fallback (v0.1.11+)
- **Multi-Cryptocurrency Support** - Generate and validate addresses for ZENYT, BTC, ETH, and more

### ZENYT Transaction System (v0.2.0+)
- **Revolutionary Transaction Types** - 8 sophisticated transaction types supporting complex operations
- **M:N Atomic Transfers** - Many-to-many transactions with up to 512 participants (256 senders + 256 recipients)
- **Genesis-Based Architecture** - All rewards distributed from 11 billion ZENYT genesis pool
- **Massive Genesis Distributions** - Up to 65,536 participants in genesis transactions (protocol-level operations)
- **Native Colored Coins** - 16 asset types with 4-bit color IDs packed efficiently
- **Same-Amount Distributions** - Space-efficient uniform payment patterns
- **Dust Rescue Operations** - Drain-smallest-first strategy preserves value
- **Smart Contract Framework** - Contract deployment and execution support
- **Tethered Asset Management** - Real-world asset linkage with metadata storage

## 📖 Documentation Overview

| Section | Description | Primary Audience |
|---------|-------------|------------------|
| [User Guide](user/README.md) | How to use Jantar | End users |
| [Admin Guide](admin/README.md) | Deployment and operations | System administrators |
| [Developer Guide](development/README.md) | Technical details and contribution | Developers |

## 🔄 Version Information

- **Current Version**: 0.2.1
- **Protocol Version**: libp2p + iroh-blobs 0.35.0
- **Transaction System**: Full ZENYT implementation with 8 transaction types
- **Minimum Rust Version**: 1.88.0

## 📝 License

Jantar is released under a proprietary license by PoC Consortium. The license restricts usage to the ZENYT blockchain network only. See [LICENSE](../LICENSE.md) for details.

## 🤝 Contributing

Jantar is developed as a proprietary reference implementation. Community contributions are welcome through:
- Feature requests and feedback
- Bug reports and issue reporting  
- Protocol improvement proposals (CIPs)
- Documentation improvements

## 📞 Support

- **Issues**: Report through community channels
- **Discord**: https://discord.gg/Mvqaa5k
- **Documentation**: You're reading it!

## 🗺️ Documentation Map

```
docs/
├── Index.md                    # This file - main entry point
├── user/                       # User documentation
│   ├── README.md              # User guide overview
│   ├── installation.md        # Installation instructions
│   ├── getting-started.md    # Quick start guide
│   ├── commands.md            # Command reference
│   ├── configuration.md      # Configuration guide
│   └── troubleshooting.md    # Common issues and solutions
├── admin/                      # Administrator documentation
│   ├── README.md              # Admin guide overview
│   ├── bootstrap-nodes.md    # Setting up bootstrap nodes
│   ├── deployment.md         # Network deployment
│   ├── monitoring.md         # Metrics and monitoring
│   ├── security.md           # Security best practices
│   └── systemd.md            # System service setup
├── coin/                       # ZENYT cryptocurrency documentation
│   ├── Zenyt_supply.md       # Supply mechanics and colored coins
│   └── Carry_over.md         # Cross-chain distribution mechanism
├── crypto/                     # Cryptographic implementation details
│   ├── ADDRESS_GENERATION_DETAILS.md  # Multi-crypto address generation
│   └── ENTROPY_AND_RANDOMNESS.md      # Security considerations
└── development/               # Developer documentation
    ├── README.md              # Developer guide overview
    ├── architecture.md        # System architecture
    ├── building.md           # Build instructions
    ├── testing.md            # Testing guide
    ├── ci-cd.md              # CI/CD pipeline
    ├── contributing.md       # Contribution guidelines
    └── TODO.md               # Development roadmap
```

---
*Last updated: 2025-08-20*