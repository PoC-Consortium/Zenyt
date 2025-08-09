# TURN Server Security Considerations

## ⚠️ IMPORTANT SECURITY WARNING

**DO NOT run your own TURN server without understanding the risks and obtaining proper authorization.**

## Why TURN Servers Are Dangerous

### 1. Relay Attack Vector
TURN servers relay arbitrary traffic between clients. Attackers can:
- Use your server as a proxy for illegal activities
- Hide their identity behind your IP address
- Make you liable for relayed traffic

### 2. Bandwidth Abuse
- Attackers can relay large amounts of data through your server
- Your bandwidth costs can skyrocket
- ISPs may throttle or terminate service

### 3. ISP Policy Violations
Many ISPs explicitly prohibit running relay servers because:
- They enable circumvention of network policies
- They can be used for copyright infringement
- They pose security risks to the network

### 4. Legal Liability
You may be held responsible for:
- Copyright infringement through your relay
- Illegal content transmitted via your server
- Cyberattacks originating from your IP

## Secure Alternatives

### 1. Use External TURN Services

Instead of running your own, use reputable providers:

```yaml
# jantar.cfg - Using external TURN service
turn_servers:
  - url: "turns:relay.example.com:443"
    username: "your-account"
    password: "your-password"
    transport: "tls"
```

**Recommended Providers:**
- [Twilio TURN](https://www.twilio.com/stun-turn)
- [Xirsys](https://xirsys.com/)
- [Coturn.net](https://coturn.net/)
- [Metered TURN](https://www.metered.ca/stun-turn)

### 2. Use STUN-Only (No Relay)

STUN servers only help with NAT discovery, no relay:

```yaml
# Safe - STUN only
stun_servers:
  - "stun:stun.l.google.com:19302"
  - "stun:stun1.l.google.com:19302"
```

### 3. Time-Limited Credentials

If you MUST run a TURN server, use time-limited credentials:

```yaml
turn_config:
  use_time_limited_credentials: true
  credential_lifetime: 3600  # 1 hour
  shared_secret: "strong-random-secret-32-chars-minimum"
```

## If You MUST Run a TURN Server

### Prerequisites
1. **Written permission from your ISP**
2. **Dedicated server with DDoS protection**
3. **Legal consultation about liability**
4. **Comprehensive logging and monitoring**
5. **Abuse detection and prevention**

### Security Hardening

#### 1. Authentication
```yaml
# NEVER use default or weak credentials
turn_credentials:
  - username: "user1"
    password: "$(openssl rand -hex 32)"  # Strong random password
```

#### 2. Rate Limiting
```yaml
turn_limits:
  max_allocations_per_user: 2
  max_bandwidth_per_user: "10mbps"
  max_lifetime: 3600
```

#### 3. IP Restrictions
```yaml
turn_access:
  allowed_peer_ips:
    - "10.0.0.0/8"      # Private networks only
    - "192.168.0.0/16"
  denied_peer_ips:
    - "0.0.0.0/8"       # Block reserved ranges
    - "224.0.0.0/4"     # Block multicast
```

#### 4. Monitoring
```bash
# Monitor for abuse patterns
tail -f /var/log/turn.log | grep -E "(excessive|abuse|flood)"

# Check bandwidth usage
iftop -i eth0

# Monitor connections
netstat -anp | grep :3478
```

#### 5. Firewall Rules
```bash
# Only allow TURN from specific IPs
iptables -A INPUT -p udp --dport 3478 -s trusted.ip -j ACCEPT
iptables -A INPUT -p udp --dport 3478 -j DROP

# Rate limit connections
iptables -A INPUT -p udp --dport 3478 -m limit --limit 10/sec -j ACCEPT
```

## Detection of TURN Abuse

### Warning Signs
- Sudden bandwidth spike
- Connections from suspicious countries
- Multiple connections from same IP
- Connections lasting > 1 hour
- Traffic patterns matching known attacks

### Monitoring Commands
```bash
# Check active TURN sessions
turn-admin -l

# Monitor bandwidth per connection
nethogs

# Check for suspicious patterns
tcpdump -i any -n port 3478 | grep -v "your.domain"
```

## Compliance Requirements

### GDPR (Europe)
- Log retention policies
- User consent for relay
- Data processing agreements

### DMCA (USA)
- Registered DMCA agent
- Takedown procedures
- Counter-notice process

### Local Regulations
- Check local laws about:
  - Proxy services
  - Data relay
  - Telecommunications services

## Jantar's Approach

Jantar v0.1.11+ takes a security-first approach:

1. **NO built-in TURN server** - Removed to prevent abuse
2. **External TURN only** - Configure trusted providers
3. **STUN preferred** - Use relay only when necessary
4. **Time-limited credentials** - When TURN is needed
5. **Extensive warnings** - Users informed of risks

### Configuration Example

```yaml
# Safe Jantar configuration
network:
  # Use STUN for NAT discovery (safe)
  stun_servers:
    - "stun:stun.l.google.com:19302"
  
  # External TURN service (when needed)
  turn_servers:
    - url: "turns:trusted-provider.com:443"
      use_time_limited: true
      credential_lifetime: 3600
  
  # NEVER enable this without understanding risks
  run_own_turn_server: false  # DANGEROUS!
```

## Alternatives to TURN

### 1. Hole Punching
- Use libp2p's DCUtR (Direct Connection Upgrade through Relay)
- Works for many NAT types without relay

### 2. Relay Nodes
- Use application-level relay (not TURN)
- Better control over what's relayed
- Can inspect/filter traffic

### 3. IPv6
- Many CGNAT issues disappear with IPv6
- Direct connections possible

### 4. VPN/Tunnels
- WireGuard for private networks
- Tailscale for zero-config networking
- More control than TURN

## Summary

**TURN servers are powerful but dangerous tools:**

❌ **DON'T** run your own TURN server without:
- ISP permission
- Legal understanding
- Security expertise
- Monitoring capability

✅ **DO** instead:
- Use external TURN providers
- Prefer STUN-only when possible
- Implement time-limited credentials
- Monitor for abuse
- Have incident response plan

Remember: **You are responsible for all traffic relayed through your TURN server.**

---
[← Back to Admin Guide](README.md) | [← Back to Index](../Index.md)