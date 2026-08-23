## SSRF - Basic Attack Against Another Backend System

**Lab:** Basic SSRF against another back-end system

Server makes requests to internal network on behalf of attacker. Attacker scans internal IPs to find admin interface.

### The Vulnerability

**Normal request:**

GET /product?stockApi=http://192.168.0.100:8080/stock

Server fetches stock from internal IP (accessible to server, not attacker).

**Exploit:** Brute force internal IPs to find admin panel.

stockApi=http://192.168.0.1:8080/admin ✗ 404
stockApi=http://192.168.0.2:8080/admin ✗ 404
stockApi=http://192.168.0.220:8080/admin ✓ 200 (found!)


### The Attack

1. Intercept stock request
2. Change `stockApi` parameter to internal IP range
3. Use Intruder to brute force: 192.168.0.1 → 192.168.0.255
4. Filter for 200 status → find admin interface
5. Once found, change path to `/admin/delete?username=carlos`

### Why It's Vulnerable

- Server makes requests to internal network
- No validation on `stockApi` parameter
- Attacker uses server to access internal systems
- Internal IPs unreachable from internet, but reachable from server

### Prevention

- Whitelist allowed internal URLs
- Never trust user input for URLs
- Block private IP ranges (10.0.0.0/8, 192.168.0.0/16, 127.0.0.0/8)
- Use network segmentation

### What I learnt
Port 80 and Port 8080 are different ports 😭 (I tried my intruder attack on port 80 to no success)
- port 80 is the main http port
- port 8080 is an **alternate** http port used for testing, development and running secondary services (like this admin page)
- The alternative https port is 8443 (not specific to this lab though)
