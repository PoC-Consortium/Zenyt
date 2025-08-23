# Metrics Dashboard & Monitoring

Jantar includes comprehensive metrics monitoring with both a modern web dashboard and Prometheus-compatible API endpoints.

## Enabling Web Server & Metrics

### Modern Configuration (Recommended)

Add the following to your configuration file:

```yaml
web:
  enabled: true           # Enable web server
  port: 9090             # Default port, can be changed
  bind_address: "127.0.0.1"  # Localhost only for security
  prometheus: true       # Enable /metrics endpoint
  dashboard: true        # Enable interactive dashboard
  api: true             # Enable JSON API endpoints
  cli: false            # Web CLI (optional, disabled by default)
```


When web server is enabled, the following endpoints become available:
- **Interactive Dashboard**: `http://localhost:9090/` (if dashboard: true)
- **Prometheus Metrics**: `http://localhost:9090/metrics` (if prometheus: true)
- **System Metrics API**: `http://localhost:9090/api/system` (if api: true)
- **Network Metrics API**: `http://localhost:9090/api/network` (if api: true)
- **Blockchain Metrics API**: `http://localhost:9090/api/blockchain` (if api: true)
- **Web CLI Interface**: `http://localhost:9090/cli/` (if cli: true)

## Web Dashboard

The interactive dashboard provides real-time monitoring with:
- **Dark Theme**: Optimized for monitoring environments
- **Live Charts**: Network activity and resource usage graphs
- **Auto-Refresh**: Updates every 5 seconds
- **Mobile Responsive**: Works on different screen sizes

### Dashboard Features
- **Network Status**: Connected peers, gossipsub messages, connection success rates
- **System Resources**: CPU usage, memory usage, disk space, uptime
- **Blockchain Status**: ZENYT balance, mempool size, transaction processing
- **File Distribution**: Available files, distribution stats, iroh storage size

## API Endpoints

The metrics system provides JSON API endpoints for programmatic access:

### `/api/system` - System Resource Metrics
Returns current system resource usage:
```json
{
  "memory_percent": "65.2",
  "cpu_percent": "12.8", 
  "disk_usage": "2.3 GB",
  "uptime": "2h 30m",
  "iroh_storage": "1.2 GB",
  "threads": 18,
  "file_descriptors": 156
}
```

### `/api/network` - Network & P2P Metrics  
Returns P2P network status and statistics:
```json
{
  "connected_peers": 3,
  "gossip_messages": 42,
  "connection_rate": "85.7",
  "available_files": 8,
  "files_distributed": 12,
  "download_rate": "2.4 MB/s"
}
```

### `/api/blockchain` - ZENYT Blockchain Metrics
Returns blockchain and transaction data:
```json
{
  "balance": "1000.50",
  "mempool_size": 3,
  "transactions_processed": 156,
  "validation_queue": 0,
  "blockchain_height": 245
}
```

### `/api/metrics` - General Metrics Status
Returns overall metrics system information:
```json
{
  "status": "active",
  "total_metrics": 45,
  "last_updated": "2025-01-20T10:30:00Z"
}
```

## Available Metrics

### P2P Connection Metrics
- `p2p_connection_attempts_total` - Total connection attempts
- `p2p_direct_connections_total` - Successful direct connections
- `p2p_relayed_connections_total` - Connections via TURN relay
- `p2p_connection_failures_total` - Failed connection attempts
- `p2p_connection_duration_seconds` - Connection establishment time

### CGNAT/NAT Detection
- `cgnat_detections_total` - CGNAT/symmetric NAT detections
- `cgnat_affected_peers_ratio` - Ratio of peers behind CGNAT
- `nat_traversal_duration_seconds` - NAT traversal operation time

### Peer Discovery
- `peers_discovered_total` - Peers discovered via mDNS
- `connected_peers_count` - Currently connected peers
- `connecting_peers_count` - Peers in connecting state

### Chat Messaging
- `chat_messages_sent_total` - Messages sent
- `chat_messages_received_total` - Messages received
- `chat_message_size_bytes` - Message size distribution

### File Distribution
- `files_distributed_total` - Files distributed via P2P
- `file_downloads_total` - File downloads
- `file_distribution_duration_seconds` - Distribution time
- `available_files_count` - Files available in network

### Update Mechanism
- `update_checks_total` - Update checks performed
- `updates_applied_total` - Updates successfully applied

### Network Health
- `network_latency_ms` - Average network latency
- `gossipsub_messages_published_total` - Gossipsub messages published
- `gossipsub_messages_received_total` - Gossipsub messages received

### Rate Limiting
- `rate_limited_connections_total` - Rate-limited connections
- `rate_limiter_active_limits` - Active rate limits

### TURN Relay (if enabled)
- `turn_relay_active_sessions` - Active TURN sessions
- `turn_relay_bytes_sent_total` - Bytes sent through relay
- `turn_relay_bytes_received_total` - Bytes received through relay

## Integration with Prometheus

1. **Configure Prometheus** to scrape Jantar metrics:

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'jantar'
    static_configs:
      - targets: ['localhost:9090']
    scrape_interval: 15s
```

2. **Example Queries**:

```promql
# Connection success rate
rate(p2p_direct_connections_total[5m]) / 
  rate(p2p_connection_attempts_total[5m])

# Average connection time
rate(p2p_connection_duration_seconds_sum[5m]) /
  rate(p2p_connection_duration_seconds_count[5m])

# Peers behind CGNAT
cgnat_affected_peers_ratio

# Message throughput
rate(chat_messages_sent_total[1m])
```

## Grafana Dashboard

Create visualizations for:
- Connection success rates
- Peer discovery timeline
- File distribution metrics
- Network latency trends
- CGNAT detection rates

## Example Configuration

```yaml
# Full example with metrics and TURN relay
name: "MonitoredNode"
port: 4444

metrics:
  enabled: true
  port: 9090

turn_relay:
  enabled: true
  port: 3478
  realm: "jantar.p2p"
```

## Viewing Metrics

1. **Direct Access**: Browse to `http://localhost:9090/metrics`
2. **Using curl**: `curl http://localhost:9090/metrics`
3. **Prometheus Server**: Configure scraping as shown above
4. **Grafana**: Import metrics from Prometheus

## Performance Impact

The metrics system has minimal performance impact:
- Metrics are updated in-memory
- HTTP endpoint only serves when accessed
- No persistent storage required
- Negligible CPU/memory overhead

---
[← Back to User Guide](README.md) | [← Back to Index](../Index.md)