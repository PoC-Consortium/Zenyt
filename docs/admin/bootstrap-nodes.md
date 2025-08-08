# Jantar Bootstrap Node Guide

This comprehensive guide covers setting up, running, and connecting to bootstrap nodes for Jantar P2P network connectivity.

## Quick Start

### For Users - Connecting to Bootstrap Nodes

Add bootstrap nodes to your `jantar.cfg`:
```yaml
bootstrap_nodes:
  - "/ip4/95.216.167.159/tcp/4001"   # EU
  - "/ip4/134.209.23.45/tcp/4001"    # US East
  - "/ip6/2001:db8::1/tcp/4001"      # IPv6 example
```

When starting Jantar, you'll see immediate feedback:
```
🌐 Bootstrap nodes:
  [1] /ip4/95.216.167.159/tcp/4001 ... ✅ dialing
  [2] /ip4/134.209.23.45/tcp/4001 ... ✅ dialing
  [3] /ip6/2001:db8::1/tcp/4001 ... ❌ failed: Transport error

🎯 Bootstrap node connected: 12D3KooWBootstrap1...
```

## Running a Bootstrap Node

### Configuration

Create a `bootstrap.cfg` file:

```yaml
name: "Bootstrap-YourServerName"
port: 4001  # Standard Jantar port

# Multiple listen addresses (NEW)
listen_addresses:
  - "0.0.0.0:4001"      # All IPv4 interfaces
  - "[::]:4001"         # All IPv6 interfaces
  # Or specific interfaces:
  # - "192.168.1.100:4001"
  # - "10.0.0.5:4001"
  # - "[2001:db8::1]:4001"

# Storage settings
download_dir: "/tmp/jantar-bootstrap"
iroh_storage_path: "/tmp/jantar-iroh"

# Bootstrap node specific settings
enable_mdns: false  # Disable local discovery (not needed on servers)
auto_update: false  # Bootstrap nodes should be manually updated

# Empty bootstrap list (this IS a bootstrap node)
bootstrap_nodes: []

# Optional: Limit storage to prevent abuse
max_storage_size: 104857600  # 100MB
min_file_size: 999999999     # Effectively disable file storage
storage_limit_strategy: "ignore_newest"

# Logging
log_level: "info"
```

### Running Methods

#### Method 1: Daemon Mode (NEW - Recommended for servers)

```bash
# Run in daemon mode (no interactive prompt)
./jantar-linux-aarch64 --daemon -c bootstrap.cfg
```

Features of daemon mode:
- No interactive prompt
- Responds to systemd signals (SIGTERM, SIGINT)
- Perfect for server deployments
- Runs until stopped by signal

#### Method 2: Systemd Service (Recommended for production)

Create `/etc/systemd/system/jantar-bootstrap.service`:

```ini
[Unit]
Description=Jantar P2P Bootstrap Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=jantar
Group=jantar
WorkingDirectory=/opt/jantar

# Use daemon mode
ExecStart=/opt/jantar/jantar --daemon -c /opt/jantar/bootstrap.cfg

# Restart on failure
Restart=on-failure
RestartSec=10

# Security hardening
PrivateTmp=true
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/jantar/logs /opt/jantar/iroh_storage

# Resource limits (optional)
LimitNOFILE=65536
MemoryLimit=1G

[Install]
WantedBy=multi-user.target
```

#### Installation Steps

1. **Create user and directories:**
```bash
sudo useradd -r -s /bin/false jantar
sudo mkdir -p /opt/jantar/{logs,iroh_storage,downloads}
sudo chown -R jantar:jantar /opt/jantar
```

2. **Copy binary and config:**
```bash
sudo cp jantar-linux-* /opt/jantar/jantar
sudo chmod +x /opt/jantar/jantar
sudo cp bootstrap.cfg /opt/jantar/
sudo chown jantar:jantar /opt/jantar/{jantar,bootstrap.cfg}
```

3. **Install and start service:**
```bash
sudo cp jantar-bootstrap.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable jantar-bootstrap
sudo systemctl start jantar-bootstrap
```

4. **Check status:**
```bash
sudo systemctl status jantar-bootstrap
sudo journalctl -u jantar-bootstrap -f
```

### Getting Your Bootstrap Address

After starting, note the peer ID from logs:
```
🔑 libp2p ID: 12D3KooWDpJ3HrX5cvPb7z4cVyfhH8CprWMkZeFVYtAqkLh5Rz5m
📡 libp2p listening on 0.0.0.0:4001
```

Your bootstrap address for others to use:
```
/ip4/YOUR.SERVER.IP.ADDRESS/tcp/4001
```

Or with peer ID (optional but more secure):
```
/ip4/YOUR.SERVER.IP.ADDRESS/tcp/4001/p2p/12D3KooWDpJ3HrX5cvPb7z4cVyfhH8CprWMkZeFVYtAqkLh5Rz5m
```

## Verifying Operation

### Check Port Binding

```bash
# Check listening ports
netstat -tlpn | grep 4001
ss -tlnp | grep jantar

# Expected output:
# tcp  0  0  0.0.0.0:4001  0.0.0.0:*  LISTEN  12345/jantar
# tcp6 0  0  :::4001       :::*       LISTEN  12345/jantar
```

### Test Connectivity

From another machine:
```bash
# Test TCP connection
telnet YOUR.SERVER.IP 4001

# Or use nc
nc -zv YOUR.SERVER.IP 4001
```

### Monitor Logs

```bash
# Systemd logs
sudo journalctl -u jantar-bootstrap -f

# Application logs
tail -f /opt/jantar/logs/*.log
```

## Network Architectures

### Local Testing (mDNS)
```yaml
# No bootstrap nodes needed
bootstrap_nodes: []
```
Uses mDNS for automatic local discovery.

### Internet with Single Bootstrap
```yaml
bootstrap_nodes:
  - "/ip4/95.216.167.159/tcp/4001"
```
All nodes connect through one bootstrap.

### Resilient Multi-Bootstrap
```yaml
bootstrap_nodes:
  - "/ip4/95.216.167.159/tcp/4001"   # EU
  - "/ip4/167.99.234.12/tcp/4001"    # US East
  - "/ip4/128.199.45.67/tcp/4001"    # Asia
```
Nodes can connect through any available bootstrap.

## Understanding Ports

When running a bootstrap node, you may see multiple ports:

- **TCP 4001** (or configured): Main libp2p port for gossipsub/chat
- **UDP 5353**: mDNS for local discovery (can be disabled)
- **Random UDP ports**: iroh-blobs QUIC protocol for file distribution

This is normal behavior. Only the configured TCP port needs to be accessible from outside.

## Firewall Configuration

Open the required port:

```bash
# Ubuntu/Debian with ufw
sudo ufw allow 4001/tcp comment 'Jantar Bootstrap'

# CentOS/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=4001/tcp
sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -p tcp --dport 4001 -j ACCEPT
```

## Multiple Interface Support (NEW)

Jantar now supports binding to multiple network interfaces:

### Examples

**Single interface, multiple ports:**
```yaml
listen_addresses:
  - "192.168.1.100:4001"
  - "192.168.1.100:4002"
```

**Multiple interfaces, same port:**
```yaml
listen_addresses:
  - "192.168.1.100:4001"  # LAN
  - "10.8.0.1:4001"       # VPN
  - "203.0.113.1:4001"    # Public IP
```

**IPv4 and IPv6:**
```yaml
listen_addresses:
  - "0.0.0.0:4001"        # All IPv4
  - "[::]:4001"           # All IPv6
  - "[2001:db8:85a3::8a2e:370:7334]:4001"  # Specific IPv6
```

## Troubleshooting

### Bootstrap Node Issues

**Port already in use:**
```bash
# Find what's using the port
sudo lsof -i :4001
sudo netstat -tlpn | grep 4001
```

**Service won't start:**
```bash
# Check logs
sudo journalctl -xe -u jantar-bootstrap

# Check config syntax
./jantar -c bootstrap.cfg -e "version"
```

**No connections:**
1. Check firewall rules
2. Verify public IP is correct
3. Test locally first: `telnet localhost 4001`
4. Check listen_addresses configuration

### Client Connection Issues

**Connection Refused:**
```
❌ failed: Transport([("/ip4/x.x.x.x/tcp/4001", ConnectionRefused)])
```
- Bootstrap node not running
- Firewall blocking port
- Wrong IP address

**Invalid Address:**
```
❌ invalid address: invalid multiaddr
```
- Use correct format: `/ip4/IP/tcp/PORT`
- No spaces or special characters

**No Route to Host:**
- Network connectivity issue
- NAT/routing problem
- Try IPv6 if available

## Security Recommendations

1. **Run as non-root user** (handled by systemd service)
2. **Don't use master nodes as bootstrap nodes**
3. **Rate limiting** (future feature)
4. **Monitor for abuse:**
   ```bash
   # Check connection count
   ss -tn | grep :4001 | wc -l
   
   # Check bandwidth usage
   iftop -P -n -f "port 4001"
   ```
5. **Regular updates** - Keep Jantar binary updated
6. **Log rotation** - Configure logrotate for `/opt/jantar/logs`

## Best Practices

### Geographic Distribution
- Place bootstrap nodes in different regions
- Consider latency for your user base
- Use anycast IPs if possible (advanced)

### Monitoring
- Set up uptime monitoring (e.g., UptimeRobot)
- Monitor connection counts
- Alert on service failures
- Track bandwidth usage

### Reliability
- Use systemd's automatic restart
- Multiple bootstrap nodes for redundancy
- Regular health checks
- Backup configurations

## Complete Setup Script

Here's a complete script to set up a bootstrap node:

```bash
#!/bin/bash
# Complete Jantar Bootstrap Node Setup

# Configuration
JANTAR_VERSION="0.1.10"
JANTAR_USER="jantar"
JANTAR_DIR="/opt/jantar"
JANTAR_PORT="4001"

# Create user
sudo useradd -r -s /bin/false $JANTAR_USER 2>/dev/null

# Create directories
sudo mkdir -p $JANTAR_DIR/{logs,iroh_storage,downloads}

# Create configuration
cat <<EOF | sudo tee $JANTAR_DIR/bootstrap.cfg
name: "Bootstrap-$(hostname)"
port: $JANTAR_PORT

# Listen on all interfaces
listen_addresses:
  - "0.0.0.0:$JANTAR_PORT"
  - "[::]:$JANTAR_PORT"

# Bootstrap specific
enable_mdns: false
auto_update: false
bootstrap_nodes: []

# Storage
download_dir: "$JANTAR_DIR/downloads"
iroh_storage_path: "$JANTAR_DIR/iroh_storage"
max_storage_size: 104857600
min_file_size: 999999999
storage_limit_strategy: "ignore_newest"

log_level: "info"
EOF

# Download Jantar (adjust URL as needed)
# sudo wget -O $JANTAR_DIR/jantar https://github.com/jantar/releases/...
# For now, copy from local
sudo cp ./jantar-linux-* $JANTAR_DIR/jantar

# Set permissions
sudo chown -R $JANTAR_USER:$JANTAR_USER $JANTAR_DIR
sudo chmod +x $JANTAR_DIR/jantar

# Create systemd service
cat <<EOF | sudo tee /etc/systemd/system/jantar-bootstrap.service
[Unit]
Description=Jantar Bootstrap Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=$JANTAR_USER
Group=$JANTAR_USER
WorkingDirectory=$JANTAR_DIR

ExecStart=$JANTAR_DIR/jantar --daemon -c $JANTAR_DIR/bootstrap.cfg

Restart=on-failure
RestartSec=10

# Security
PrivateTmp=true
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=$JANTAR_DIR/logs $JANTAR_DIR/iroh_storage

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable jantar-bootstrap
sudo systemctl start jantar-bootstrap

# Configure firewall
if command -v ufw &> /dev/null; then
    sudo ufw allow $JANTAR_PORT/tcp comment 'Jantar Bootstrap'
elif command -v firewall-cmd &> /dev/null; then
    sudo firewall-cmd --permanent --add-port=$JANTAR_PORT/tcp
    sudo firewall-cmd --reload
fi

# Get public IP
PUBLIC_IP=$(curl -s ifconfig.me)

echo "========================================="
echo "Bootstrap node setup complete!"
echo "========================================="
echo "Status: sudo systemctl status jantar-bootstrap"
echo "Logs:   sudo journalctl -u jantar-bootstrap -f"
echo ""
echo "Your bootstrap address:"
echo "/ip4/$PUBLIC_IP/tcp/$JANTAR_PORT"
echo "========================================="
```

## Advanced Topics

### Future Features

- **Relay Support**: NAT traversal for nodes behind firewalls
- **DHT Integration**: Full decentralization without fixed bootstrap nodes
- **DNS Discovery**: Use DNS records for bootstrap discovery
- **Peer Exchange**: Nodes share known peers automatically

### Monitoring Dashboard

Consider setting up monitoring:
- Grafana + Prometheus for metrics
- Log aggregation with Loki
- Custom health check endpoint (future feature)

## Next Steps

1. **Test locally** first with two instances
2. **Deploy to VPS** with public IP
3. **Share bootstrap address** with users
4. **Monitor and maintain** the service
5. **Add redundancy** with multiple bootstrap nodes

The bootstrap node is essential for internet-scale Jantar deployments. With daemon mode and multi-interface support, it's now production-ready!