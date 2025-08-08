# Running Jantar as a systemd Service

## Daemon Mode

Jantar now supports a proper daemon mode for systemd compatibility:

```bash
jantar --daemon -c /path/to/config.cfg
```

In daemon mode:
- No interactive prompt is shown
- Responds properly to systemd signals (SIGTERM, SIGINT)
- Logs to file (see logs/ directory)
- Runs until stopped by signal

## Systemd Service File

Create `/etc/systemd/system/jantar.service`:

```ini
[Unit]
Description=Jantar P2P Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=jantar
Group=jantar
WorkingDirectory=/opt/jantar

# Use daemon mode
ExecStart=/opt/jantar/jantar --daemon -c /opt/jantar/config.cfg

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

## Installation Steps

1. Create user and directories:
```bash
sudo useradd -r -s /bin/false jantar
sudo mkdir -p /opt/jantar/{logs,iroh_storage}
sudo chown -R jantar:jantar /opt/jantar
```

2. Copy binary and config:
```bash
sudo cp jantar-linux-* /opt/jantar/jantar
sudo chmod +x /opt/jantar/jantar
sudo cp your-config.cfg /opt/jantar/config.cfg
sudo chown jantar:jantar /opt/jantar/{jantar,config.cfg}
```

3. Install and start service:
```bash
sudo cp jantar.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable jantar
sudo systemctl start jantar
```

4. Check status:
```bash
sudo systemctl status jantar
sudo journalctl -u jantar -f
```

## Configuration for Bootstrap Nodes

For bootstrap nodes, use this configuration:

```yaml
name: "Bootstrap-YourName"
port: 4001

# Listen on specific interfaces
listen_addresses:
  - "0.0.0.0:4001"      # All IPv4 interfaces
  - "[::]:4001"         # All IPv6 interfaces
  # Or specify specific IPs:
  # - "192.168.1.100:4001"
  # - "10.0.0.5:4001"
  # - "[2001:db8::1]:4001"

# Bootstrap specific
enable_mdns: false      # Disable local discovery
auto_update: false      # Manual updates only
bootstrap_nodes: []     # This IS a bootstrap

# Storage limits
download_dir: "/opt/jantar/downloads"
iroh_storage_path: "/opt/jantar/iroh_storage"
max_storage_size: 104857600  # 100MB
storage_limit_strategy: "ignore_newest"
```

## Multiple Network Interfaces

Jantar now supports binding to multiple network interfaces:

### Configuration Examples

1. **Single interface, multiple ports:**
```yaml
listen_addresses:
  - "192.168.1.100:4001"
  - "192.168.1.100:4002"
```

2. **Multiple interfaces, same port:**
```yaml
listen_addresses:
  - "192.168.1.100:4001"  # LAN interface
  - "10.8.0.1:4001"       # VPN interface
  - "203.0.113.1:4001"    # Public IP
```

3. **IPv4 and IPv6:**
```yaml
listen_addresses:
  - "0.0.0.0:4001"                    # All IPv4
  - "[::]:4001"                        # All IPv6
  - "[2001:db8:85a3::8a2e:370:7334]:4001"  # Specific IPv6
```

4. **Multiaddr format (advanced):**
```yaml
listen_addresses:
  - "/ip4/0.0.0.0/tcp/4001"
  - "/ip6/::/tcp/4001"
  - "/ip4/127.0.0.1/tcp/4002"
```

## Logging

Logs are written to:
- Console output (captured by systemd/journald)
- File: `logs/YYYY-MM-DD_HH:MM:SS_jantar_PID.log`

View logs:
```bash
# Systemd journal
sudo journalctl -u jantar -f

# Log files
tail -f /opt/jantar/logs/*.log
```

## Firewall Configuration

Open required ports:

```bash
# For Ubuntu/Debian with ufw
sudo ufw allow 4001/tcp comment 'Jantar P2P'

# For CentOS/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=4001/tcp
sudo firewall-cmd --reload

# For iptables
sudo iptables -A INPUT -p tcp --dport 4001 -j ACCEPT
```

## Monitoring

Check if service is listening:

```bash
# Check listening ports
ss -tlnp | grep jantar
netstat -tlnp | grep 4001

# Check process
ps aux | grep jantar

# Check connections
ss -tnp | grep jantar
```

## Troubleshooting

### Service won't start

Check logs:
```bash
sudo journalctl -xe -u jantar
```

Common issues:
- Port already in use
- Config file syntax error
- Permissions on directories
- Binary not executable

### No network connectivity

1. Check firewall rules
2. Verify listen_addresses configuration
3. Check network interface is up
4. Test with telnet: `telnet your-ip 4001`

### High resource usage

Adjust limits in config:
- `max_storage_size`: Limit blob storage
- `storage_limit_strategy`: Control what happens when full
- `peer_blacklist`: Block abusive peers

## Security Recommendations

1. **Run as non-root user** (already in service file)
2. **Use systemd security features** (already configured)
3. **Regular updates**: Monitor for Jantar updates
4. **Network isolation**: Use firewall to limit access
5. **Resource limits**: Set memory and CPU limits if needed
6. **Log rotation**: Configure logrotate for log files

## Example Bootstrap Node Setup

Complete setup for a public bootstrap node:

```bash
#!/bin/bash
# Setup script for Jantar bootstrap node

# Create user
sudo useradd -r -s /bin/false jantar

# Create directories
sudo mkdir -p /opt/jantar/{logs,iroh_storage,downloads}

# Create config
cat <<EOF | sudo tee /opt/jantar/config.cfg
name: "Bootstrap-$(hostname)"
port: 4001
listen_addresses:
  - "0.0.0.0:4001"
  - "[::]:4001"
enable_mdns: false
auto_update: false
bootstrap_nodes: []
download_dir: "/opt/jantar/downloads"
iroh_storage_path: "/opt/jantar/iroh_storage"
max_storage_size: 104857600
storage_limit_strategy: "ignore_newest"
log_level: "info"
EOF

# Set permissions
sudo chown -R jantar:jantar /opt/jantar

# Copy binary (adjust path)
sudo cp ./jantar-linux-* /opt/jantar/jantar
sudo chmod +x /opt/jantar/jantar

# Create service
cat <<EOF | sudo tee /etc/systemd/system/jantar.service
[Unit]
Description=Jantar Bootstrap Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=jantar
Group=jantar
WorkingDirectory=/opt/jantar
ExecStart=/opt/jantar/jantar --daemon -c /opt/jantar/config.cfg
Restart=on-failure
RestartSec=10
PrivateTmp=true
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/jantar/logs /opt/jantar/iroh_storage /opt/jantar/downloads

[Install]
WantedBy=multi-user.target
EOF

# Start service
sudo systemctl daemon-reload
sudo systemctl enable jantar
sudo systemctl start jantar

# Open firewall
sudo ufw allow 4001/tcp comment 'Jantar P2P'

echo "Bootstrap node setup complete!"
echo "Check status: sudo systemctl status jantar"
echo "View logs: sudo journalctl -u jantar -f"
echo "Your bootstrap address: /ip4/$(curl -s ifconfig.me)/tcp/4001"
```