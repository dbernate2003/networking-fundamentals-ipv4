# Exercise 2 — Subnet Design (IPv4)

## Objective
Design and implement a network divided into 3 subnets using a single Class C address block, combining DHCP and static IP assignment in a Cisco Packet Tracer environment.

---

## Why subnets instead of classes?

In Exercise 1, each LAN received an entire network class, wasting millions of addresses for just 2 PCs per LAN. Subnetting solves this by dividing a single network into smaller, equally-sized blocks.

| Approach | Analogy | Problem |
|----------|---------|---------|
| Classes | Entire neighborhood for 2 people | Massive waste |
| Subnets | Neighborhood divided into exact lots | Efficient use |

---

## Network topology

```
            Router (Cisco 2911)
           /         |          \
      Switch0      Switch1      Switch2
      (2960)       (2960)       (2960)
       /   \        /   \        /   \
    PC0   PC1    PC2   PC3    PC4   PC5
    LAN1           LAN2         LAN3
```

---

## Subnetting calculation

Base network: `192.168.10.0` (Class C)

```
3 LANs required → next power of 2 = 4 → 2 bits borrowed

11111111.11111111.11111111.11|000000
                              ^^
                         subnet bits
                               ^^^^^^
                            host bits

2² = 4 subnets available (3 used, 1 reserved)
2⁶ - 2 = 62 usable hosts per subnet
Subnet mask: 255.255.255.192 (/26)
```

---

## Subnet table

| Subnet | Network address | Usable hosts      | Broadcast       |
|--------|----------------|-------------------|-----------------|
| 1      | 192.168.10.0   | .1 ~ .62          | 192.168.10.63   |
| 2      | 192.168.10.64  | .65 ~ .126        | 192.168.10.127  |
| 3      | 192.168.10.128 | .129 ~ .190       | 192.168.10.191  |
| 4      | 192.168.10.192 | (unused)          | 192.168.10.255  |

---

## Addressing table

| LAN  | Subnet           | Gateway          | Subnet mask           | Assignment |
|------|-----------------|------------------|-----------------------|------------|
| LAN1 | 192.168.10.0    | 192.168.10.1     | 255.255.255.192 /26   | DHCP       |
| LAN2 | 192.168.10.64   | 192.168.10.65    | 255.255.255.192 /26   | Static     |
| LAN3 | 192.168.10.128  | 192.168.10.129   | 255.255.255.192 /26   | DHCP       |

---

## Router configuration

### Step 1 — Interface setup
```bash
enable
configure terminal
hostname MY-ROUTER

# LAN1 - Subnet 1
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.192
no shutdown
exit

# LAN2 - Subnet 2
interface gigabitEthernet 0/1
ip address 192.168.10.65 255.255.255.192
no shutdown
exit

# LAN3 - Subnet 3
interface gigabitEthernet 0/2
ip address 192.168.10.129 255.255.255.192
no shutdown
exit
```

### Step 2 — DHCP configuration (LAN1 and LAN3)
```bash
# Exclude all gateway addresses
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.10.65
ip dhcp excluded-address 192.168.10.129

# DHCP pool for LAN1
ip dhcp pool DHCP-LAN1
network 192.168.10.0 255.255.255.192
default-router 192.168.10.1
exit

# DHCP pool for LAN3
ip dhcp pool DHCP-LAN3
network 192.168.10.128 255.255.255.192
default-router 192.168.10.129
exit
```

---

## PC configuration

### LAN1 — DHCP
Select DHCP in Desktop → IP Configuration on each PC.

### LAN2 — Static assignment
| PC  | IP address      | Subnet mask           | Gateway         |
|-----|-----------------|-----------------------|-----------------|
| PC2 | 192.168.10.66   | 255.255.255.192       | 192.168.10.65   |
| PC3 | 192.168.10.67   | 255.255.255.192       | 192.168.10.65   |

### LAN3 — DHCP
Select DHCP in Desktop → IP Configuration on each PC.

---

## Why configure a default gateway on every PC?

When a PC communicates within its own subnet, no gateway is needed.
When it needs to reach a different subnet, it must know the exit point:

```
Without gateway:
PC → "192.168.10.66 is not in my subnet"
   → "How do I get out?" → Unknown → packet lost ❌

With gateway:
PC → "192.168.10.66 is not in my subnet"
   → "I exit through 192.168.10.1" → Router → LAN2 ✅
```

---

## Verification commands

```bash
# Check all interfaces status
show ip interface brief

# Check assigned IPs
show ip dhcp binding
```

Expected output for `show ip interface brief`:
```
Interface             IP-Address       OK? Method Status   Protocol
GigabitEthernet0/0    192.168.10.1     YES manual up       up
GigabitEthernet0/1    192.168.10.65    YES manual up       up
GigabitEthernet0/2    192.168.10.129   YES manual up       up
```

---

## Connectivity tests

```bash
# Within same subnet (from PC0)
ping 192.168.10.2      # PC0 → PC1 (LAN1)

# Toward the router gateway
ping 192.168.10.1      # PC0 → Router

# Between different subnets
ping 192.168.10.66     # PC0 (LAN1) → PC2 (LAN2)
ping 192.168.10.130    # PC0 (LAN1) → PC4 (LAN3)
```

---

## Issues found and solutions

### Issue 1 — DHCP not assigning gateway
**Cause:** `default-router` command missing from DHCP pool.
**Solution:** Add `default-router` to each pool and run `ipconfig /renew` on affected PCs.

### Issue 2 — Ping fails between subnets
**Cause:** PC had no default gateway configured.
**Solution:** Set gateway in Desktop → IP Configuration or verify `default-router` in DHCP pool.

---

## Key concepts applied

- **Subnetting** → dividing one Class C network into 4 equal blocks of 64 addresses
- **Borrowed bits** → 2 bits taken from host portion to create subnet identifiers
- **62 usable hosts** → 2⁶ - 2 (subtract network address and broadcast)
- **/26 mask** → 24 original Class C bits + 2 borrowed bits = 26
- **Default gateway** → mandatory for inter-subnet communication
