# networking-fundamentals-ipv4
Learning IPv4 addressing, network classes, and subnets using Cisco Packet Tracer.
A hands-on learning project where I implement core computer networking concepts: IPv4 addressing, network classes, subnets, and Cisco device configuration via CLI.

---

## Academic sources

- **Textbook:** Castaño Ribes, Rafael Jesús — *Redes Locales*, Units 6 and 7
- **Practical implementation:** Classroom videos and Cisco Networking Academy — Introduction to Packet Tracer certificate

---

## Learning objectives

- Understand IPv4 addressing using network classes (A, B, C)
- Implement subnets using variable-length subnet masks (VLSM)
- Configure Cisco routers and switches via CLI
- Set up static and dynamic (DHCP) IP assignment
- Verify network connectivity using diagnostic commands

---

## Concepts covered

### IPv4 Addressing
- 32-bit structure divided into network and host portions
- Network classes A, B and C with their ranges and masks
- Reserved addresses: network address, broadcast and assignable IPs
- CIDR prefix notation (/8, /16, /24, /26...)
- Private vs public IP addresses

### Subnetting
- Dividing a network into smaller subnets
- Calculating borrowed bits based on required number of subnets
- Available hosts formula: 2ⁿ - 2
- Subnet mask and its binary representation

### Cisco Packet Tracer
- Router interface configuration via CLI
- DHCP server configuration on a router
- Static IP assignment on end devices
- Verification using `show ip interface brief` and `ping`

---

## Exercise 1 — Class-based Network Design

### Topology
```
            Router (Cisco 2911)
           /         |          \
      Switch0      Switch1      Switch2
      (2960)       (2960)       (2960)
       /   \        /   \        /   \
    PC0   PC1    PC2   PC3    PC4   PC5
    LAN1           LAN2         LAN3
```

### Addressing table

| LAN  | Class | Network       | Gateway      | Subnet mask       | Assignment |
|------|-------|---------------|--------------|-------------------|------------|
| LAN1 | A     | 10.0.0.0      | 10.0.0.1     | 255.0.0.0 /8      | Static     |
| LAN2 | B     | 172.16.0.0    | 172.16.0.1   | 255.255.0.0 /16   | DHCP       |
| LAN3 | C     | 192.168.1.0   | 192.168.1.1  | 255.255.255.0 /24 | DHCP       |

---

## Exercise 2 — Subnet Design

*(Coming soon)*

### Topology
Same physical topology as Exercise 1.

### Addressing table

| LAN  | Subnet           | Gateway          | Subnet mask          | Assignment |
|------|-----------------|------------------|----------------------|------------|
| LAN1 | 192.168.10.0    | 192.168.10.1     | 255.255.255.192 /26  | DHCP       |
| LAN2 | 192.168.10.64   | 192.168.10.65    | 255.255.255.192 /26  | Static     |
| LAN3 | 192.168.10.128  | 192.168.10.129   | 255.255.255.192 /26  | DHCP       |

### Why /26?
```
3 subnets required → next power of 2 = 4 → 2 bits borrowed
Class C: 8 host bits
  - 2 borrowed bits → 2² = 4 subnets available
  - 6 remaining bits → 2⁶ - 2 = 62 hosts per subnet
  - Mask: 11111111.11111111.11111111.11000000 = 255.255.255.192
```

---

## Issues found and solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| `DHCP request failed` | Pool configured with wrong network | Delete pool with `no ip dhcp pool` and reconfigure |
| `Default Gateway: 0.0.0.0` | Missing `default-router` in DHCP pool | Add `default-router` to pool configuration |
| Interface not available | Router 1941 only has 2 GigabitEthernet interfaces | Use router 2911 which has 3 interfaces |
| First ping times out | ARP resolution process on router | Normal behavior — subsequent packets respond correctly |

---

## Certifications in progress

- [x] Introduction to Packet Tracer — Cisco Networking Academy
- [ ] Networking Basics — Cisco Networking Academy
- [ ] Network Addressing and Basic Troubleshooting
- [ ] Network Support and Security

---

## Repository structure

```
networking-fundamentals-ipv4/
 ├── unit-1-classes/
 │   ├── topology.pkt
 │   ├── notes.md
 │   └── screenshots/
 ├── unit-2-subnets/
 │   ├── topology.pkt
 │   ├── notes.md
 │   └── screenshots/
 ├── theory-notes/
 │   └── ipv4-summary.md
 ├── certifications/
 └── README.md
```

---

## Next steps

- [ ] Complete Cisco Networking Academy certifications
- [ ] Explore VLAN configuration
- [ ] Study routing protocols (RIP, OSPF)

---

*Active learning project — Systems Engineering*
