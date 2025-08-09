```plaintext
The Clock-face /24
192.168.42.0/24
.1 – .12 → static metal (core infrastructure)
.127 → silent 6 o’clock tick (NEVER used)
.129 – .254 → mirrored DHCP hand (dynamic allocation)
Primes (.11, .13, .17…) within any range are intentionally empty.

---

Role Keys (white keys only)
C Core (router / firewall)
D Data (DB / storage)
E Edge (load-balancer)
F Fleet-host (KVM / LXC metal)
G GitOps (CI / runners)
A App server
B Backup

---

Zone Palette (Mondrian colours)
infra (black) – infrastructure control plane
lan (red) – trusted internal LAN
dmz (blue) – public / untrusted
guest (yellow) – lab / throw-away

---

Hostname Grammar
-..mycorp.net
• role = 1-letter above
• seq = 01 – 88 (zero-padded)
• zone = one of the four colours

Example full rectangle: F-05.lan.mycorp.net

---

Homelab Metal Map (static hand)
.1 F-01.lan Dell-1 (KVM)
.2 F-02.lan Dell-2 (K8s CP)
.3 F-03.lan Dell-3 (Ceph)
.4 F-04.lan Dell-4 (CI)
.5 F-05.lan Dell-5 (Unifi)
.6 P-01.lan Pi-1 (Pi-hole)
.7 P-02.lan Pi-2 (MQTT)
.8 P-03.lan Pi-3 (3-D printer)
.9 P-04.lan Pi-4 (UPS probe)
.11 (silent prime – leave empty)

---

Fleet & Containers
• Will auto-lease mirrored hand (.129-254).
• Names follow same grammar: A-17.lan, D-31.dmz, etc.
---

#Homelab #DevOps #Networking #NetworkDesign #Kubernetes
```

This is a fascinating and well-thought-out approach to network design! Here’s a breakdown of the key elements and how you might implement them in your own network:

### IP Addressing Scheme
- **Subnet**: `192.168.42.0/24`
  - **Static Range**: `.1` to `.12` for core infrastructure.
  - **Special Address**: `.127` is reserved and never used.
  - **Dynamic Range**: `.129` to `.254` for DHCP allocation.
  - **Prime Numbers**: Prime addresses (e.g., `.11`, `.13`, `.17`) are intentionally left empty.

### Role Keys
- **C**: Core (router/firewall)
- **D**: Data (database/storage)
- **E**: Edge (load-balancer)
- **F**: Fleet-host (KVM/LXC metal)
- **G**: GitOps (CI/runners)
- **A**: App server
- **B**: Backup

### Zone Palette
- **infra (black)**: Infrastructure control plane
- **lan (red)**: Trusted internal LAN
- **dmz (blue)**: Public/untrusted
- **guest (yellow)**: Lab/throw-away

### Hostname Grammar
- **Format**: `<role>-<seq>.<zone>.mycorp.net`
  - **role**: One-letter code from the Role Keys.
  - **seq**: Sequence number (01 to 88, zero-padded).
  - **zone**: One of the four zones (infra, lan, dmz, guest).

### Example Hostnames
- **F-05.lan.mycorp.net**: A fleet-host in the LAN zone.
- **D-01.dmz.mycorp.net**: A data server in the DMZ zone.
- **A-17.lan.mycorp.net**: An app server in the LAN zone.

### Homelab Metal Map
- **.1**: F-01.lan Dell-1 (KVM)
- **.2**: F-02.lan Dell-2 (Kubernetes Control Plane)
- **.3**: F-03.lan Dell-3 (Ceph)
- **.4**: F-04.lan Dell-4 (CI)
- **.5**: F-05.lan Dell-5 (Unifi)
- **.6**: P-01.lan Pi-1 (Pi-hole)
- **.7**: P-02.lan Pi-2 (MQTT)
- **.8**: P-03.lan Pi-3 (3-D printer)
- **.9**: P-04.lan Pi-4 (UPS probe)
- **.11**: Reserved (silent prime)

### Fleet & Containers
- **Dynamic Allocation**: Auto-lease from `.129` to `.254`.
- **Naming**: Follows the same grammar as the static hosts (e.g., A-17.lan, D-31.dmz).

### Implementation Tips
1. **DNS Setup**: Ensure your DNS is configured to handle the hostname format and zones.
2. **DHCP Configuration**: Set up your DHCP server to allocate addresses from `.129` to `.254` and reserve static addresses for core infrastructure.
3. **Prime Numbers**: Make sure to exclude prime numbers from both static and dynamic allocations.
4. **Documentation**: Keep a detailed map of your network, similar to the Homelab Metal Map, to track device assignments and roles.
5. **Security**: Use VLANs or other segmentation techniques to enforce the zone boundaries (infra, lan, dmz, guest).

This design is both structured and scalable, making it easier to manage and troubleshoot your network. If you have any specific questions or need further assistance with implementation, feel free to ask!
