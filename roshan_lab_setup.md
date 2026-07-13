# EVE-NG Packet Broker Lab — Setup Guide



## 1. Topology

```
VPC1 ---------- Gi0/2                      Gi0/1 ---------- ens4 [Ubuntu Node]
 10.0.0.1/24              [Packet Broker]                    (mgmt / SSH to broker)
VPC2 ---------- Gi0/4                      Gi0/3 ---------- ens5 [Ubuntu Node]
 10.0.0.2/24                                                (monitoring / promiscuous capture)
```

- **VPC1 / VPC2**: test traffic sources, connected directly to the Packet Broker's data ports.
- **Packet Broker**: switches VPC1 ↔ VPC2 traffic normally over Gi0/2 and Gi0/4, and simultaneously mirrors both ports out Gi0/3 toward the monitoring interface. Also has a dedicated management port (Gi0/1) for SSH access.
- **Ubuntu Node**: ens3 for ssh from host then two interfaces — `ens4` for SSH/management into the Pacektbroker, `ens5` for passively capturing the mirrored traffic (promiscuous mode).

---

## 2. IP Addressing

| Node/Interface | IP Address | Purpose |
|---|---|---|
| VPC1 | 10.0.0.1/24 | Test traffic source |
| VPC2 | 10.0.0.2/24 | Test traffic source |
| Packet Broker Vlan99 (mgmt SVI) | 172.16.100.1/24 | SSH management, reached via Gi0/1 |
| Ubuntu `ens4` | 172.16.100.2/24 | Management path to Packetbroker |
| Ubuntu `ens5` | none needed | Promiscuous capture only |
| Ubuntu `ens3` (for ssh) | DHCP / home network | General SSH access to the Ubuntu node itself |

---

## 3. VLAN Plan

| VLAN | Purpose | Ports |
|---|---|---|
| 10 | Data VLAN — VPC1/VPC2 production traffic | Gi0/2, Gi0/4 |
| 99 | Management VLAN — broker SSH access | Gi0/1 |

> Gi0/3 is configured as an access port but its role is purely as a `monitor session` destination — once a port is a SPAN destination, it stops participating in normal switching/STP for that purpose, so exact VLAN assignment on Gi0/3 is not important, just leave it as vlan 10 or any unused vlan for consistency.

---

## 4. Node Configurations

### 4.1 VPC1
```
VPC1> ip 10.0.0.1/24
```

### 4.2 VPC2
```
VPC2> ip 10.0.0.2/24
```

### 4.3 Packet Broker 

```
enable
configure terminal
hostname Packet-Broker
ip domain-name lab.local

! --- Data ports: normal switching between VPC1 and VPC2 ---
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface GigabitEthernet0/4
 switchport mode access
 switchport access vlan 10
 no shutdown

! --- Mirror port: sends copies of VPC1/VPC2 traffic to the monitoring host ---
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 10
 no shutdown

monitor session 1 source interface GigabitEthernet0/2 , GigabitEthernet0/4
monitor session 1 destination interface GigabitEthernet0/3

! --- Management port + SVI for SSH ---
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 99
 no shutdown

interface Vlan99
 ip address 172.16.100.1 255.255.255.0
 no shutdown

! --- SSH setup ---
crypto key generate rsa modulus 1024
ip ssh version 2

username admin privilege 15 secret YourStrongPasswordHere
line vty 0 4
 login local
 transport input ssh
exit

enable secret YourEnableSecretHere
end
write memory
```

### 4.4 Ubuntu Node

**Network config** (`/etc/netplan/01-broker-lab.yaml`):
```yaml
network:
  version: 2
  ethernets:
    ens4:
      addresses:
        - 172.16.100.2/24
      dhcp4: false
    ens5:
      dhcp4: false
```
```bash
sudo netplan apply
```

**Promiscuous mode on `ens5`** (receives mirrored traffic):
```bash
sudo ip link set ens5 promisc on
```

**Persist across reboots** (`/etc/systemd/system/ens5-promisc.service`):
```ini
[Unit]
Description=Set ens5 to promiscuous mode
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/sbin/ip link set ens5 promisc on
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ens5-promisc.service
```

**Install tcpdump if needed:**
```bash
sudo apt install tcpdump -y
```

---

## 5. SSH from Ubuntu Node to Packet Broker

SSH server only offers legacy key exchange algorithms; add a client config entry so you don't need to pass flags manually every time.
Open ssh config file and add following

`vi ~/.ssh/config`:
```
Host packet-broker
    HostName 172.16.100.1
    User admin
    KexAlgorithms +diffie-hellman-group14-sha1
    HostKeyAlgorithms +ssh-rsa
    Ciphers +aes128-cbc
```
```bash
ssh packet-broker
```

---

## 6. End-to-End Test

1. Start capture on the Ubuntu node:
   ```bash
   sudo tcpdump -i ens5 -n icmp
   ```
2. From VPC1, ping VPC2:
   ```
   VPC1> ping 10.0.0.2 -c 20
   ```
3. Confirm VPC1 ↔ VPC2 ICMP traffic appears in the `ens5` capture, even though the Ubuntu node isn't a party to that conversation.
4. Confirm management SSH still works independently:
   ```bash
   ssh packet-broker
   ```

**Sanity check it's a real mirror, not routing:** `ping 10.0.0.1` from the Ubuntu node should fail — it has no L3 reachability into VLAN 10, confirming `ens5` is passively observing via SPAN only.

---

## 7. Troubleshooting Reference

| Symptom | Likely Cause | Fix |
|---|---|---|
| No traffic on `ens5` at all | `monitor session` not active, or Gi0/3 down | `show monitor session 1` on the packetbroker — confirm operational status and that Gi0/3 shows `connected` |
| VPC1 can't reach VPC2 | VLAN mismatch on Gi0/2/Gi0/4, or one port not `no shutdown` | `show interfaces status` — confirm both ports `connected` and VLAN 10 |
| SSH KEX error | IOSv legacy crypto vs modern OpenSSH client defaults | Use the `~/.ssh/config` entry in Section 5 |
| SSH password fails after KEX succeeds | `username`/`secret` missing or VTY not set to `login local` | `show running-config \| include username` / `\| section line vty` |
| `ens5` loses promiscuous mode after reboot | Not persisted | Confirm the systemd unit in Section 4.4 is enabled |

---
