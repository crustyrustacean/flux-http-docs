## Chapter 1 Cheatsheet

### OSI Model (7 Layers)
From top to bottom:
1. **Application** - User-facing protocols (HTTP, FTP, SMTP)
2. **Presentation** - Encryption, compression, formatting
3. **Session** - Session management
4. **Transport** - TCP/UDP, ports, reliability
5. **Network** - IP addresses, routing
6. **Data Link** - Ethernet, Wi-Fi, MAC addresses
7. **Physical** - Cables, electrical signals

*Note: Theoretical model; internet uses TCP/IP instead*

### TCP/IP Stack (4 Layers)
1. **Application Layer** - HTTP, DNS, SSH (combines OSI layers 5-7)
2. **Transport Layer** - TCP (reliable) or UDP (fast)
3. **Internet Layer** - IP, routing across networks
4. **Link Layer** - Ethernet, Wi-Fi (combines OSI layers 1-2)

### IP Addresses

**IPv4**: 32-bit, dotted decimal notation
```
192.168.1.1
8.8.8.8
```

**IPv6**: 128-bit, hexadecimal notation
```
2001:0db8:85a3::8a2e:0370:7334
::1 (loopback)
```

**Special Addresses**:
- `127.0.0.1` / `::1` - Loopback (localhost)
- `0.0.0.0` / `::` - All interfaces (for binding servers)
- `192.168.x.x`, `10.x.x.x` - Private (non-routable)

### Ports

**Range**: 0-65535 (16-bit number)

**Common Ports**:
- 20/21 - FTP
- 22 - SSH
- 25 - SMTP (email)
- 53 - DNS
- 80 - HTTP
- 443 - HTTPS
- 3306 - MySQL
- 5432 - PostgreSQL
- 8000 - Common dev port

**Port Ranges**:
- 0-1023: Well-known (require privileges)
- 1024-49151: Registered
- 49152-65535: Dynamic/ephemeral

**Socket Address Format**:
```
93.184.216.34:80        (IPv4)
[2001:db8::1]:443       (IPv6)
localhost:8000
```

### DNS

**Purpose**: Translates hostnames → IP addresses

**Record Types**:
- **A** - Hostname to IPv4
- **AAAA** - Hostname to IPv6  
- **CNAME** - Alias to another hostname
- **MX** - Mail servers
- **NS** - Nameservers for domain
- **TXT** - Arbitrary text data

**Resolution Process**:
1. Check local cache (browser, OS)
2. Query DNS resolver (ISP or public like 8.8.8.8)
3. Resolver queries root → TLD → authoritative nameserver
4. Return IP address and cache it

**TTL**: Time (in seconds) a DNS record can be cached

**Common Commands**:
```bash
# Lookup DNS records
nslookup example.com
dig example.com

# Flush DNS cache
# macOS:
sudo dscacheutil -flushcache
# Windows:
ipconfig /flushdns
# Linux:
sudo systemd-resolve --flush-caches
```

### Quick Reference for Development

**Bind to localhost only** (development):
```rust
TcpListener::bind("127.0.0.1:8000")
```

**Bind to all interfaces** (production):
```rust
TcpListener::bind("0.0.0.0:8000")
```

**Make it configurable**:
```rust
let addr = env::var("BIND_ADDRESS")
    .unwrap_or_else(|_| "127.0.0.1:8000".to_string());
```

**Connection anatomy**:
```
Client: 192.168.1.100:54321 → Server: 93.184.216.34:80
       (random ephemeral)           (well-known HTTP)
```