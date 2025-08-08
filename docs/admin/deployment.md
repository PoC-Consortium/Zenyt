# Internet-Scale Deployment for Jantar

This document explores how to evolve Jantar from local network testing to internet-scale deployment using DHT and bootstrap nodes.

## Current State

Jantar currently uses:
- **mDNS** for local network peer discovery
- **Gossipsub** for message/file propagation
- **Bootstrap nodes** configuration (exists but not fully utilized)
- **No DHT** (Kademlia is imported but not implemented)

## Required Changes for Internet-Scale

### 1. Implement Kademlia DHT

The Kademlia DHT will enable:
- Global peer discovery beyond local networks
- Persistent peer routing tables
- NAT traversal assistance
- Decentralized bootstrap mechanism

**Implementation steps:**
```rust
// In JantarBehaviour struct
pub kad: libp2p::kad::Behaviour<kad::store::MemoryStore>,

// In behaviour creation
let kad = kad::Behaviour::new(
    peer_id,
    kad::store::MemoryStore::new(peer_id),
);

// Add bootstrap nodes to Kademlia
for addr in &config.bootstrap_nodes {
    if let Ok(multiaddr) = addr.parse::<Multiaddr>() {
        kad.add_address(&peer_id, multiaddr);
    }
}
```

### 2. Bootstrap Node Infrastructure

#### Option A: Dedicated Bootstrap Nodes
Deploy long-running Jantar instances on cloud VMs:
- AWS/GCP/DigitalOcean instances in multiple regions
- Static IP addresses
- High uptime guarantee
- Minimal resource usage (just routing, no storage)

**Example bootstrap config:**
```yaml
# bootstrap-node.cfg
name: "Bootstrap-US-East"
port: 4001
auto_update: false
enable_dht: true
bootstrap_mode: true  # New flag
```

#### Option B: Community Bootstrap Nodes
Allow trusted community members to run bootstrap nodes:
- Provide docker image for easy deployment
- Monitor uptime and reliability
- Rotate list periodically

### 3. Enhanced Network Configuration

Update config structure:
```yaml
# Enhanced jantar.cfg
network:
  # Enable DHT for internet-scale discovery
  enable_dht: true
  
  # Bootstrap nodes (global)
  bootstrap_nodes:
    - "/ip4/165.232.45.78/tcp/4001/p2p/12D3KooWBootstrap1..."
    - "/ip4/134.209.23.45/tcp/4001/p2p/12D3KooWBootstrap2..."
    - "/ip6/2001:db8::1/tcp/4001/p2p/12D3KooWBootstrap3..."
  
  # Relay nodes for NAT traversal
  relay_nodes:
    - "/p2p-circuit/p2p/12D3KooWRelay1..."
    - "/p2p-circuit/p2p/12D3KooWRelay2..."
  
  # Network behavior
  connection_limits:
    max_peers: 50
    max_pending_connections: 10
    connection_grace_period: 20s
```

### 4. NAT Traversal

Implement relay functionality for nodes behind NAT:
- Use libp2p's relay protocol
- AutoNAT for detecting NAT status
- Circuit relay for hole punching

```rust
// Enable relay client
.with_relay_client(noise::Config::new, yamux::Config::default)?

// For relay servers
.with_relay_server(relay::Config::default())?
```

### 5. Connection Management

Implement intelligent peer management:
- Prefer geographically close peers
- Maintain mix of old/new connections
- Implement peer scoring based on:
  - Response time
  - Uptime
  - Data availability
  - Geographic diversity

### 6. Deployment Strategy

#### Phase 1: Hybrid Mode (Current)
- Keep mDNS for local discovery
- Add DHT for internet peers
- Test with your global VMs

#### Phase 2: Bootstrap Infrastructure
- Deploy 3-5 bootstrap nodes globally
- Document connection process
- Test cross-continent connectivity

#### Phase 3: Community Expansion
- Release with default bootstrap nodes
- Allow custom bootstrap configuration
- Monitor network health

## Bootstrap Node Deployment Guide

### 1. Basic Bootstrap Node (VPS)
```bash
# Install dependencies
sudo apt update && sudo apt install -y docker.io

# Run bootstrap node
docker run -d \
  --name jantar-bootstrap \
  --restart unless-stopped \
  -p 4001:4001 \
  -v /opt/jantar:/data \
  jantar:latest \
  -c /data/bootstrap.cfg
```

### 2. Bootstrap Configuration
```yaml
# bootstrap.cfg
name: "Bootstrap-Region-1"
port: 4001
download_dir: "/tmp/jantar"  # Minimal storage
auto_update: false
enable_dht: true
max_storage_size: 104857600  # 100MB limit

# Don't participate in file storage
min_file_size: 999999999  # Effectively disable
```

### 3. Monitoring
```bash
# Check peer connections
curl http://localhost:9090/metrics | grep jantar_peers

# Check DHT health
curl http://localhost:9090/dht/stats
```

## Security Considerations

1. **Bootstrap Node Security**
   - Run with minimal privileges
   - Firewall rules (only P2P ports)
   - Rate limiting
   - No master keys on bootstrap nodes

2. **Sybil Attack Prevention**
   - Peer ID verification
   - Connection limits per IP
   - Proof of work for new peers (future)

3. **Privacy**
   - Optional Tor integration
   - Encrypted peer discovery
   - Minimal metadata exposure

## Testing Strategy

### Local Testing with Multiple Networks
```bash
# Simulate internet conditions
tc qdisc add dev eth0 root netem delay 100ms loss 1%

# Test with different NAT types
docker network create --driver bridge nat_strict
docker network create --driver bridge nat_moderate
```

### Global Testing Checklist
- [ ] Cross-continent peer discovery
- [ ] File propagation across NAT
- [ ] Bootstrap node failover
- [ ] Network partition recovery
- [ ] Update propagation globally

## Example Bootstrap Nodes

For initial testing, consider these providers:
1. **Hetzner** (EU) - Good bandwidth, affordable
2. **DigitalOcean** (Global) - Easy API, multiple regions
3. **Vultr** (Global) - Good Asian presence
4. **OVH** (EU/Canada) - DDoS protection included

## Future Enhancements

1. **GeoIP Integration**
   - Prefer regional peers
   - Optimize content routing

2. **Incentive Layer**
   - Reward reliable nodes
   - Penalize malicious behavior

3. **Mobile Considerations**
   - Lite clients
   - Push notifications
   - Battery optimization

## Implementation Priority

1. **High Priority**
   - Basic Kademlia DHT
   - Bootstrap node connection
   - NAT detection

2. **Medium Priority**
   - Relay functionality
   - Connection scoring
   - Geographic optimization

3. **Low Priority**
   - Tor integration
   - Mobile optimizations
   - Advanced DHT features

This approach will transform Jantar from a local network tool to a truly global P2P application while maintaining its simplicity and security.