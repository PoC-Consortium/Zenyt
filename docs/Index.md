# Jantar Documentation

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
1. Download the latest [AppImage (Linux)](https://github.com/yourrepo/releases) or [Windows executable](https://github.com/yourrepo/releases)
2. Make it executable: `chmod +x jantar-*.AppImage`
3. Run: `./jantar-*.AppImage`
4. Type `help` to see available commands

### For Developers
```bash
# Clone the repository
git clone https://github.com/yourrepo/jantar.git
cd jantar

# Build
cargo build --release

# Run tests
cargo test

# Run
./target/release/jantar
```

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
- **Multi-Cryptocurrency Support** - Generate and validate addresses for ZENYT, BTC, ETH, and more

## 📖 Documentation Overview

| Section | Description | Primary Audience |
|---------|-------------|------------------|
| [User Guide](user/README.md) | How to use Jantar | End users |
| [Admin Guide](admin/README.md) | Deployment and operations | System administrators |
| [Developer Guide](development/README.md) | Technical details and contribution | Developers |

## 🔄 Version Information

- **Current Version**: 0.1.11
- **Protocol Version**: libp2p + iroh-blobs 0.35.0
- **Minimum Rust Version**: 1.70.0

## 📝 License

Jantar is released under the GNU General Public License v3.0. See [LICENSE](../LICENSE.md) for details.

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](development/contributing.md) for details.

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/yourrepo/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourrepo/discussions)
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
*Last updated: 2025-08-08*