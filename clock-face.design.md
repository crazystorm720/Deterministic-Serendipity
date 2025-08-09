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

#Homelab #DevOps #Networking #NetworkDesign #Kubernetes
