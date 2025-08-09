# Prometheus Metrics

Jantar includes built-in Prometheus metrics for monitoring network health, peer connections, and file distribution.

## Enabling Metrics

Add the following to your configuration file:

```yaml
metrics:
  enabled: true
  port: 9090  # Default port, can be changed
```

When enabled, metrics are available at: `http://localhost:9090/metrics`

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