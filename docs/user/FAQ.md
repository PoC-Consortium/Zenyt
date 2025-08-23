# Frequently Asked Questions

## General Questions

### When moon? 🌙

Not sure - not a priority.

### When will source code be available?

While not all code may be immediately open-sourced (such as the current wallet implementation), the development process itself remains transparent through community discussion, proposal mechanisms, and consistent communication. Ultimately, the relationship between anonymous developers and the community is built on trust earned through consistent delivery and principled action rather than complete code transparency.

Instead of source code availability, Zenyt focuses on providing comprehensive documentation and technical specifications that would enable independent reimplementation by qualified developers. This approach ensures the protocol itself remains open and well-documented while maintaining control over the reference implementation.

The development team maintains transparency through:
- **Coin Improvement Proposals (CIPs)**: Community participation in protocol decisions
- **Technical specifications**: Detailed protocol documentation
- **Public communication**: Open discussion of features and roadmap
- **Consistent delivery**: Trust built through reliable software updates

### Why is the source code not available?

The Zenyt development team maintains a balance between transparency and security:

1. **Security considerations**: Proprietary implementation helps prevent certain attack vectors
2. **Team anonymity**: Protecting developer privacy and reducing attack surfaces
3. **Focus on protocol**: Emphasizing specification over implementation details
4. **Quality control**: Ensuring a stable, well-tested reference implementation

### Can I build my own Zenyt implementation?

Yes, independent implementations are encouraged based on the published specifications and protocol documentation. The technical documentation provides sufficient detail for qualified developers to create compatible implementations.

### How do I verify the software without source code?

- **SHA256 checksums**: All releases include SHA256 hashes for binary integrity verification
- **Official distribution channels**: Only download from verified official sources
- **Community review**: Public discussion of features and behavior
- **Protocol compliance**: Implementations must follow published specifications

## Technical Questions

### What platforms are supported?

**Currently distributed:**
- **Linux**: x86_64 (Ubuntu, Debian, CentOS, Arch)

**Planned support:**
- **Linux**: ARM64 (Raspberry Pi and other ARM64 systems)
- **Windows**: x86_64 (Windows 10/11)  
- **macOS**: x86_64, ARM64 (Intel and Apple Silicon)

### How do I report bugs or request features?

- **Discord**: https://discord.gg/Mvqaa5k
- **GitHub Issues**: https://github.com/PoC-Consortium/Zenyt/issues
- **Feature requests**: Submit through CIP (Coin Improvement Proposal) process

### What are the system requirements?

**Minimum requirements:**
- 2GB RAM
- 1GB storage space
- Broadband internet connection

**Recommended:**
- 4GB+ RAM
- 2GB+ storage space
- Stable internet connection
- GPU with OpenCL support (optional, for acceleration)

## Network Questions

### How do I connect to other peers?

Jantar uses automatic peer discovery through:
- **mDNS**: Local network discovery
- **Bootstrap nodes**: Seed peers maintained by the network
- **Relay nodes**: NAT traversal assistance

### What ports does Jantar use?

- **Default P2P port**: 4001 (configurable)
- **Metrics port**: 9090 (optional, configurable)
- **RPC port**: Unix socket by default (configurable)

### How do I set up a bootstrap node?

Bootstrap nodes require:
- Public IP address
- Stable internet connection
- 24/7 uptime
- Proper firewall configuration

See [Bootstrap Node Guide](../admin/bootstrap-nodes.md) for detailed setup instructions.


---

*Last updated: 2025-08-23*