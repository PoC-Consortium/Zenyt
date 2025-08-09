# Jantar Administrator Guide

This guide is for system administrators, network operators, and those running Jantar nodes in production environments.

## 📑 Table of Contents

1. [Bootstrap Nodes](bootstrap-nodes.md) - Setting up network entry points
2. [Deployment](deployment.md) - Large-scale network deployment
3. [System Services](systemd.md) - Running as a system service
4. [RPC Interface](rpc-interface.md) - Remote control for daemons
5. [Monitoring](monitoring.md) - Metrics, logs, and observability
6. [Security](security.md) - Hardening and best practices
7. [TURN Relay Setup](turn-relay.md) - Setting up TURN for NAT traversal
8. [TURN Security](turn-security.md) - Critical security warnings for TURN
9. [TURN Double Standard](turn-relay-hypocrisy.md) - The ISP hypocrisy explained
10. [Master Nodes](master-nodes.md) - Operating update distribution nodes

## 🎯 Administration Overview

As a Jantar administrator, you may be responsible for:
- Running bootstrap nodes that help peers discover each other
- Operating master nodes that distribute updates
- Managing TURN relay servers for NAT traversal
- Monitoring network health and performance
- Ensuring security and availability

## 🏗️ Infrastructure Components

### Bootstrap Nodes
Entry points for new peers joining the network. Bootstrap nodes should:
- Have stable public IP addresses
- Be highly available (99%+ uptime)
- Have good network connectivity
- Run on ports accessible through firewalls (443, 4001)

### Master Nodes
Special nodes with signing keys that can:
- Distribute official updates
- Add files to the P2P network
- Crawl network topology
- Should be highly secured

### TURN Relay Servers
Help peers connect through restrictive NATs:
- Required for CGNAT environments
- Standard ports: 3478 (STUN/TURN)
- Can be resource-intensive under load
- Should be geographically distributed

## 📊 Monitoring Stack

### Metrics Collection
- **Prometheus** for metrics aggregation
- **Grafana** for visualization
- Metrics endpoint: `http://localhost:9090/metrics`

### Key Metrics to Monitor
- Connection success rates
- TURN relay usage
- File distribution statistics
- Network latency
- Error rates

### Log Management
- Logs stored in `logs/` directory
- Structured logging with timestamps
- Log rotation recommended
- Centralized logging for multiple nodes

## 🔒 Security Considerations

### Network Security
- Use firewalls to limit access
- Enable rate limiting
- Configure peer blacklists
- Monitor for unusual activity

### Node Security
- Secure master node private keys
- Use separate user accounts
- Enable SELinux/AppArmor if available
- Regular security updates

### Data Security
- Configure storage limits
- Set file size restrictions
- Regular backups of configuration
- Monitor disk usage

## 🚀 Deployment Scenarios

### Small Network (< 100 peers)
- 1-2 bootstrap nodes
- 1 master node
- Optional TURN relay

### Medium Network (100-1000 peers)
- 3-5 bootstrap nodes
- 2 master nodes (redundancy)
- 1-2 TURN relays
- Prometheus monitoring

### Large Network (1000+ peers)
- 5-10 bootstrap nodes (geographically distributed)
- 3+ master nodes
- Multiple TURN relays per region
- Full monitoring stack
- Load balancing considerations

## 📋 Operational Checklist

### Daily Tasks
- [ ] Check node availability
- [ ] Monitor error logs
- [ ] Review metrics dashboards
- [ ] Verify disk space

### Weekly Tasks
- [ ] Review connection statistics
- [ ] Check for updates
- [ ] Analyze performance trends
- [ ] Test backup procedures

### Monthly Tasks
- [ ] Security audit
- [ ] Capacity planning review
- [ ] Update documentation
- [ ] Performance optimization

## 🛠️ Common Administrative Tasks

### Starting a Bootstrap Node
```bash
./jantar -c bootstrap.cfg
```

### Monitoring Node Status
```bash
# Check peers
./jantar -e "peers"

# Check version
./jantar -e "version"

# View metrics
curl http://localhost:9090/metrics
```

### Distributing Updates (Master Node)
```bash
./jantar -c master.cfg
> add-file jantar-0.1.11-x86_64.AppImage
```

## 📚 Next Steps

- Set up [Bootstrap Nodes](bootstrap-nodes.md)
- Configure [System Services](systemd.md)
- Enable [Monitoring](monitoring.md)
- Review [Security Best Practices](security.md)

## 🆘 Administrator Support

- Technical documentation: [Developer Guide](../development/README.md)
- Network planning: [Deployment Guide](deployment.md)
- Troubleshooting: [Admin Troubleshooting](troubleshooting.md)

---
[← Back to Documentation Index](../Index.md)