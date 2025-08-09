# The TURN Relay Double Standard

## The Hypocrisy Explained

Your ISP operates massive relay infrastructure while forbidding you from running a simple TURN server. This document explains the double standard and how to navigate it.

## 🎭 The Double Standard

### What ISPs Do (Legal & Protected):
- **CGNAT**: Relay millions of connections through shared IPs
- **Transparent Proxies**: Intercept and relay your HTTP traffic
- **DNS Servers**: Relay all your DNS queries
- **Content Caching**: Store and relay popular content
- **VPN Services**: Sell relay services at premium prices
- **IPv6 Tunnels**: Relay IPv4 over IPv6 infrastructure

**Their Protection**: Safe harbor under DMCA §512(a), common carrier provisions

### What You Do (Same Thing, But "Illegal"):
- **TURN Server**: Relay connections for NAT traversal
- **Technical Function**: Identical to what ISPs do
- **Your Risk**: Full liability for relayed traffic
- **Your Protection**: None

## 📊 The Actual Differences

| Aspect | ISP Relay | Your Relay |
|--------|-----------|------------|
| **Technical Function** | Forward packets | Forward packets |
| **Content Inspection** | No | No |
| **Content Modification** | No | No |
| **User Control** | No | No |
| **Legal Status** | Protected "mere conduit" | Potentially liable |
| **Terms of Service** | "We're not responsible" | "You're fully responsible" |

## 🤔 Why This Exists

### 1. **Business Model Protection**
```
Residential Plan: $50/month - "No servers allowed"
Business Plan: $500/month - "Same service but you can run servers"
```

### 2. **Liability Hot Potato**
```
Copyright Holder → ISP: "Someone pirated our content!"
ISP → You: "It was this customer running a relay!"
You → ??? : "But I was just relaying like you do..."
Court: "You're liable."
```

### 3. **Control & Monitoring**
- ISPs want to see all traffic for:
  - Data mining
  - Ad injection
  - Traffic shaping
  - Law enforcement cooperation
- Relays break this visibility

### 4. **Competition Elimination**
Your free relay competes with:
- ISP's VPN service ($10/month)
- ISP's "static IP" addon ($15/month)
- ISP's "port forwarding" addon ($5/month)
- ISP's "business" upgrade ($450/month)

## 🎯 The Real Rules

### What They Say:
"Running servers violates our Terms of Service"

### What They Mean:
"Running servers that compete with our upsells or complicate our liability violates our business model"

### The Actual Risk Matrix:

| Service Type | ISP Cares? | Why |
|--------------|------------|-----|
| Personal website | No | No bandwidth, no liability |
| Game server | Maybe | Bandwidth usage |
| File sharing | Yes | Legal liability |
| TURN relay | YES | Liability + hides traffic |
| Mining | YES | Constant bandwidth |
| Tor exit | YES++ | Maximum liability |

## 🛡️ Protecting Yourself

### If You Must Run a TURN Server:

#### 1. **Document Everything**
```yaml
turn_config:
  # Legal protection documentation
  terms_of_use: |
    This TURN server is operated as a common carrier service.
    Operator exercises no control over transmitted content.
    Users are solely responsible for their traffic.
    
  legal_notice: |
    This server operates under the same principles as ISP routers.
    No content inspection, modification, or selection occurs.
    Operator claims same safe harbor protections as ISPs.
```

#### 2. **Restrict Heavily**
```yaml
turn_config:
  mode: builtin
  builtin:
    # Only for specific, known users
    allowed_users:
      - friend@example.com
      - family@example.com
    
    # Only relay to safe destinations
    allowed_peer_ips:
      - "1.1.1.1/32"  # Cloudflare DNS
      - "8.8.8.8/32"  # Google DNS
      - "192.168.0.0/16"  # Private networks
    
    # Severe limits
    max_bandwidth_per_user: 50000  # 50KB/s
    session_timeout: 300  # 5 minutes
    max_sessions: 10  # Total, not per user
```

#### 3. **Log Like an ISP**
```yaml
logging:
  # Keep same logs ISPs do
  connection_timestamps: true
  source_ips: true
  destination_ips: true
  bytes_transferred: true
  
  # But like ISPs, don't log:
  packet_contents: false
  application_data: false
  
  retention: "7 days"  # Match ISP standard
```

## 📜 The Honest Configuration

```yaml
# TURN Relay Configuration
# 
# THE DOUBLE STANDARD:
# ===================
# Your ISP's routers relay packets = "common carrier"
# Your TURN server relays packets = "potential liability"
# 
# Yes, it's the same technical function.
# Yes, it's hypocritical.
# No, pointing this out won't protect you legally.
# 
# THE REAL REASON IT'S BANNED:
# ============================
# 1. Competes with ISP's paid services
# 2. Complicates their liability narrative  
# 3. Reduces their traffic visibility
# 4. You don't have their legal team
#
# YOUR ACTUAL OPTIONS:
# ====================

turn:
  # Option 1: SAFE - Don't run a relay
  mode: none
  
  # Option 2: SAFER - Use someone else's relay
  # mode: external
  # external:
  #   servers:
  #     - url: "turns:company-with-lawyers.com:443"
  #       # Let them deal with the liability
  
  # Option 3: RISKY - Run your own
  # mode: builtin
  # 
  # If you choose this, you're stating:
  i_understand_the_double_standard: false  # Must be true
  i_accept_isp_may_terminate_service: false  # Must be true
  i_have_legal_representation: false  # Recommended true
  i_understand_i_am_responsible_for_all_relayed_traffic: false  # Must be true
  
  # Even though ISPs do the same thing and claim they're NOT responsible.
  # Welcome to the wonderful world of regulatory capture.
```

## 🤝 The Compromise

If you need TURN functionality:

### Best: Use External Services
Companies with legal teams and proper Terms of Service:
- Twilio TURN
- Xirsys
- CoTURN cloud providers

### Acceptable: Restricted Local Relay
```yaml
# Only for your own devices
turn:
  mode: builtin
  builtin:
    bind_address: "192.168.1.1"  # LAN only
    allowed_peer_ips: ["192.168.1.0/24"]  # LAN only
    credentials:
      - username: "my_phone"
        password: "$(openssl rand -hex 32)"
```

### Never: Open Internet Relay
```yaml
# DON'T DO THIS
turn:
  mode: builtin
  builtin:
    bind_address: "0.0.0.0"  # No!
    allowed_peer_ips: []  # No restrictions = lawsuit magnet
```

## 📚 Further Reading

- [EFF: ISP Safe Harbors](https://www.eff.org/issues/dmca-safe-harbors)
- [DMCA Section 512](https://www.law.cornell.edu/uscode/text/17/512)
- [Common Carrier Exemptions](https://en.wikipedia.org/wiki/Common_carrier)

## The Bottom Line

**It's not fair, but it's reality.**

ISPs have lawyers, lobbyists, and law on their side. You don't. Until the law recognizes that packet relay is packet relay regardless of who does it, you have to play by unfair rules.

**Recommendation**: Use `mode: external` and let someone else deal with the hypocrisy.

---
[← Back to Admin Guide](README.md) | [← Back to Index](../Index.md)