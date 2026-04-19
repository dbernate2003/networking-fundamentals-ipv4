# Exercise 1 — Network Classes (IPv4)

## Objective
Design and implement a network with 3 LANs using IPv4 class-based addressing, combining static IP assignment and DHCP in a Cisco Packet Tracer environment.

---

## Network topology

```
            Router (Cisco 2911)
           /         |          \
     Switch0       Switch1      Switch2
     (2960)        (2960)       (2960)
      /   \         /   \        /   \
   PC0   PC1     PC2   PC3    PC4   PC5
   LAN1            LAN2         LAN3
```

---

## Addressing table

| LAN  | Class | Network       | Gateway      | Subnet mask       | Assignment |
|------|-------|---------------|--------------|-------------------|------------|
| LAN1 | A     | 10.0.0.0      | 10.0.0.1     | 255.0.0.0 /8      | Static     |
| LAN2 | B     | 172.16.0.0    | 172.16.0.1   | 255.255.0.0 /16   | DHCP       |
| LAN3 | C     | 192.168.1.0   | 192.168.1.1  | 255.255.255.0 /24 | DHCP       |

---

## Router configuration

### Step 1 — Interface setup
```bash
enable
configure terminal
hostname MY-ROUTER

# LAN1 - Class A
interface gigabitEthernet 0/0
ip address 10.0.0.1 255.0.0.0
no shutdown
exit

# LAN2 - Class B
interface gigabitEthernet 0/1
ip address 172.16.0.1 255.255.0.0
no shutdown
exit

# LAN3 - Class C
interface gigabitEthernet 0/2
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

### Step 2 — DHCP configuration
```bash
# Exclude gateway addresses from DHCP pools
ip dhcp excluded-address 172.16.0.1
ip dhcp excluded-address 192.168.1.1

# DHCP pool for LAN2
ip dhcp pool DHCP-LAN2
network 172.16.0.0 255.255.0.0
default-router 172.16.0.1
exit

# DHCP pool for LAN3
ip dhcp pool DHCP-LAN3
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
exit
```

---

## PC configuration

### LAN1 — Static assignment
| PC  | IP address | Subnet mask | Gateway   |
|-----|------------|-------------|-----------|
| PC0 | 10.0.0.2   | 255.0.0.0   | 10.0.0.1  |
| PC1 | 10.0.0.3   | 255.0.0.0   | 10.0.0.1  |

### LAN2 and LAN3 — DHCP
Select DHCP in Desktop → IP Configuration on each PC.
Verify with:
```bash
ipconfig /renew
ipconfig
```

---

## Verification commands

```bash
# Check all interfaces status
show ip interface brief

# Check DHCP pools
show ip dhcp pool

# Check assigned IPs
show ip dhcp binding
```

Expected output for `show ip interface brief`:
```
Interface           IP-Address    OK? Method Status   Protocol
GigabitEthernet0/0  10.0.0.1      YES manual up       up
GigabitEthernet0/1  172.16.0.1    YES manual up       up
GigabitEthernet0/2  192.168.1.1   YES manual up       up
```

---

## Connectivity tests

```bash
# Within same LAN (from PC0)
ping 10.0.0.3       # PC0 → PC1 (LAN1)

# Toward the router gateway
ping 10.0.0.1       # PC0 → Router

# Between different LANs
ping 172.16.0.2     # PC0 (LAN1) → PC2 (LAN2)
ping 192.168.1.2    # PC0 (LAN1) → PC4 (LAN3)
```

---

## Issues found and solutions

### Issue 1 — DHCP request failed
**Cause:** DHCP pool was configured with the wrong network address (LAN3 address assigned to LAN2 pool).
**Solution:**
```bash
no ip dhcp pool DHCP-LAN2
# Recreate pool with correct network
ip dhcp pool DHCP-LAN2
network 172.16.0.0 255.255.0.0
default-router 172.16.0.1
```

### Issue 2 — Default gateway 0.0.0.0 after DHCP
**Cause:** `default-router` command was missing from the DHCP pool configuration.
**Solution:** Add `default-router` to each pool and run `ipconfig /renew` on affected PCs.

### Issue 3 — First ping always times out
**Cause:** Normal ARP resolution process — the router needs one packet to learn the destination MAC address.
**Solution:** Not an error. Subsequent packets respond correctly.

---

## Key concepts applied

- **Class A** → 8 network bits, 24 host bits → up to 16,777,214 hosts per network
- **Class B** → 16 network bits, 16 host bits → up to 65,534 hosts per network
- **Class C** → 24 network bits, 8 host bits → up to 254 hosts per network
- **DHCP** automatically assigns IP, subnet mask, and default gateway to each PC
- **Gateway** is the router interface IP — the exit door for each LAN
- **`no shutdown`** is required to activate router interfaces (off by default)
