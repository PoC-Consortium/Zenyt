# Monitoring & Observability

Jantar provides comprehensive monitoring capabilities for production deployments through its integrated metrics dashboard and API endpoints.

## Dashboard Overview

The web-based dashboard provides real-time monitoring of node status, network health, and system resources.

### Accessing the Dashboard

**Default URL**: `http://localhost:9090/`

**Configuration**:
```yaml
metrics:
  enabled: true
  port: 9090
```

### Security Considerations

- **Localhost Only**: Dashboard binds to 127.0.0.1 by default for security
- **No Authentication**: Currently no built-in authentication (plan for reverse proxy)
- **Internal Networks**: Only expose on trusted networks

## Production Monitoring Setup

### 1. External Access via Reverse Proxy

For remote monitoring, use a reverse proxy with authentication:

**Nginx Configuration**:
```nginx
server {
    listen 443 ssl;
    server_name jantar-monitor.example.com;
    
    # SSL configuration
    ssl_certificate /path/to/certificate.pem;
    ssl_certificate_key /path/to/private.key;
    
    # Basic authentication
    auth_basic "Jantar Monitoring";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    location / {
        proxy_pass http://127.0.0.1:9090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /api/ {
        proxy_pass http://127.0.0.1:9090/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 2. API Integration with External Tools

#### Prometheus Integration
While Jantar doesn't expose Prometheus format by default, you can scrape the API endpoints:

**scrape_config.yml**:
```yaml
scrape_configs:
  - job_name: 'jantar-nodes'
    static_configs:
      - targets: ['node1:9090', 'node2:9090']
    metrics_path: '/api/metrics'
    scrape_interval: 30s
```

#### Grafana Dashboards
Use the JSON API endpoints as data sources for custom dashboards.

#### Alerting with curl
```bash
#!/bin/bash
# Check if node is healthy
RESPONSE=$(curl -s http://localhost:9090/api/system)
PEERS=$(echo $RESPONSE | jq -r '.connected_peers // 0')

if [ "$PEERS" -lt 1 ]; then
    echo "ALERT: Jantar node has no peers connected"
    # Send notification
fi
```

## Key Metrics for Production

### Critical Alerts
Monitor these metrics for production health:

**Network Health**:
- `connected_peers` < 1 (isolated node)
- `connection_rate` < 50% (networking issues)
- `gossip_messages` not increasing (network partition)

**System Resources**:
- `memory_percent` > 90% (memory pressure)
- `cpu_percent` > 80% (sustained load)
- `disk_usage` approaching limits
- `file_descriptors` approaching system limits

**Blockchain Status**:
- `mempool_size` growing continuously (processing issues)
- `validation_queue` > 10 (validation bottleneck)

### Performance Baselines

**Normal Operation**:
- CPU: 5-15% average
- Memory: 50-200MB typical
- Peers: 2-10 connected
- File descriptors: 50-200

**Under Load**:
- CPU: 20-40% during file transfers
- Memory: May increase during large transfers
- Network: Varies with P2P activity

## Log Analysis

### Application Logs
Jantar logs network events to rotating files:
```bash
# View recent network events
tail -f ~/.jantar/logs/network.log

# Search for connection issues
grep -i "error\|failed\|timeout" ~/.jantar/logs/network.log
```

### System Integration
For production, integrate with system logging:

**rsyslog configuration** (`/etc/rsyslog.d/jantar.conf`):
```
# Forward Jantar logs to centralized logging
if $programname == 'jantar' then @@log-server:514
& stop
```

## Health Check Endpoints

### Basic Health Check
```bash
# Simple connectivity test
curl -f http://localhost:9090/api/metrics
```

### Detailed Health Assessment
```bash
#!/bin/bash
# Comprehensive health check

echo "=== Jantar Node Health Check ==="

# Check API accessibility
if ! curl -s http://localhost:9090/api/metrics > /dev/null; then
    echo "❌ Dashboard API unreachable"
    exit 1
fi

# Check system resources
SYSTEM=$(curl -s http://localhost:9090/api/system)
MEMORY=$(echo $SYSTEM | jq -r '.memory_percent // "0"' | sed 's/[^0-9.]//g')
CPU=$(echo $SYSTEM | jq -r '.cpu_percent // "0"' | sed 's/[^0-9.]//g')

if (( $(echo "$MEMORY > 90" | bc -l) )); then
    echo "⚠️  High memory usage: ${MEMORY}%"
fi

if (( $(echo "$CPU > 80" | bc -l) )); then
    echo "⚠️  High CPU usage: ${CPU}%"
fi

# Check network connectivity
NETWORK=$(curl -s http://localhost:9090/api/network)
PEERS=$(echo $NETWORK | jq -r '.connected_peers // 0')

if [ "$PEERS" -eq 0 ]; then
    echo "❌ No peers connected"
    exit 1
elif [ "$PEERS" -lt 2 ]; then
    echo "⚠️  Low peer count: $PEERS"
else
    echo "✅ Healthy peer connections: $PEERS"
fi

echo "✅ Node appears healthy"
```

## Dashboard Customization

### Corporate Environments
For corporate deployments, you may want to:

1. **Rebrand the interface** (modify dashboard HTML)
2. **Add custom metrics** (extend API endpoints)
3. **Integrate with existing tools** (SNMP, Nagios, etc.)

### Mobile Access
The dashboard is responsive and works on mobile devices for on-the-go monitoring.

## Troubleshooting

### Dashboard Not Loading
1. Check port binding: `netstat -tlnp | grep 9090`
2. Verify metrics enabled in configuration
3. Check application logs for startup errors
4. Test API endpoints directly with curl

### Missing Metrics Data
1. Ensure system monitor is running
2. Check file system permissions for /proc access
3. Verify background tasks are not crashing

### High Resource Usage
1. Monitor file descriptor count
2. Check for memory leaks in long-running processes
3. Review P2P connection patterns
4. Consider rate limiting aggressive peers

## Future Enhancements

Planned monitoring improvements:
- Built-in authentication
- Configurable alert thresholds  
- Email/webhook notifications
- Historical data retention
- Multi-node fleet management

For the latest monitoring features, check the project documentation and release notes.