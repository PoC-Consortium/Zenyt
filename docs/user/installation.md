# Installation Guide

Jantar is distributed as a standalone binary that doesn't require installation. Simply download and run!

## 📥 Download Options

### Linux (x86_64)
```bash
# Download the AppImage
wget https://github.com/yourrepo/releases/jantar-0.2.0-x86_64.AppImage

# Make it executable
chmod +x jantar-0.2.0-x86_64.AppImage

# Run it
./jantar-0.2.0-x86_64.AppImage
```

### Linux ARM64 (Raspberry Pi)
```bash
# Download the ARM64 AppImage
wget https://github.com/yourrepo/releases/jantar-0.2.0-aarch64.AppImage

# Make it executable
chmod +x jantar-0.2.0-aarch64.AppImage

# Run it
./jantar-0.2.0-aarch64.AppImage
```

### Windows
```powershell
# Download jantar-0.2.0-x86_64.exe
# Run from Command Prompt or PowerShell
jantar-0.2.0-x86_64.exe
```

### macOS
```bash
# Coming soon - use cargo build for now
cargo build --release
./target/release/jantar
```

## 🚀 First Run

When you first run Jantar, it automatically:
1. Creates a default configuration file (`jantar.cfg`)
2. Initializes storage directories
3. Starts connecting to the P2P network

```bash
# First run - creates default config
./jantar

# Run with custom config
./jantar -c alice.cfg
```

## 📁 File Locations

Jantar creates these directories on first run:

### Configuration
- **Linux/macOS**: `./jantar.cfg` (current directory)
- **Windows**: `.\jantar.cfg` (current directory)

### Storage
- **iroh_storage/**: P2P file storage
- **logs/**: Application logs
- **downloads/**: Received files (configurable)

### Command History
- `.jantar_history_<node_name>` - Per-node command history

## 🔧 System Requirements

### Minimum Requirements
- **OS**: Linux, Windows 10+, macOS 11+
- **RAM**: 256MB
- **Disk**: 100MB for application + storage space
- **Network**: Internet connection

### Recommended
- **RAM**: 1GB+
- **Disk**: 10GB+ for file storage
- **Network**: Broadband connection
- **Ports**: Allow outbound TCP connections

## 🐳 Docker (Optional)

```dockerfile
# Dockerfile example
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y wget
WORKDIR /app
RUN wget https://github.com/yourrepo/releases/jantar-0.2.0-x86_64.AppImage
RUN chmod +x jantar-0.2.0-x86_64.AppImage
CMD ["./jantar-0.2.0-x86_64.AppImage"]
```

## ⚡ Quick Start

1. **Download** the appropriate binary for your platform
2. **Make executable** (Linux/macOS): `chmod +x jantar-*`
3. **Run**: `./jantar` or `jantar.exe`
4. **Check status**: Type `peers` to see connected nodes

## 🔄 Updating

Jantar can update itself through the P2P network:

```bash
# Check for updates
[Alice]> update

# Update to specific version
[Alice]> sys update 0.1.13
```

## 🛠️ Building from Source

If binaries aren't available for your platform:

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Clone repository
git clone https://github.com/yourrepo/jantar.git
cd jantar

# Build release version
cargo build --release

# Run
./target/release/jantar
```

## ❓ Troubleshooting Installation

### Linux: "Permission denied"
```bash
chmod +x jantar-*.AppImage
```

### Linux: "FUSE not available"
```bash
# Option 1: Install FUSE
sudo apt install fuse  # Debian/Ubuntu
sudo yum install fuse  # RHEL/CentOS

# Option 2: Extract and run
./jantar-*.AppImage --appimage-extract
./squashfs-root/AppRun
```

### Windows: "Unknown publisher" warning
This is normal for unsigned binaries. Click "More info" → "Run anyway"

### macOS: "Cannot be opened because the developer cannot be verified"
```bash
# Remove quarantine attribute
xattr -d com.apple.quarantine jantar
```

## 📚 Next Steps

- [Getting Started](getting-started.md) - Basic usage guide
- [Configuration](configuration.md) - Customize your node
- [Commands Reference](commands.md) - All available commands

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)