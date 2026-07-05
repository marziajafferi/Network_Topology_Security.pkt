# Secure Network Topology with DMZ, VLANs, and Access Controls

This project demonstrates a security-focused enterprise network design implemented in Cisco Packet Tracer. The network is designed with strict segmentation, a Demilitarized Zone (DMZ) for public-facing services, Access Control Lists (ACLs) for traffic filtering, and Port Security to prevent unauthorized device access.

## 🛠️ Environment & Tools
* **Simulation Tool:** Cisco Packet Tracer
* **Hardware Simulated:** Cisco 2911 Routers, Cisco 2960 Switches, End Devices, Web Server.
* **Protocols/Features:** VLANs, 802.1Q Trunking, Router-on-a-Stick (Inter-VLAN Routing), Standard & Extended ACLs, Port Security.

## 📐 Network Architecture & IP Schema

The network is divided into distinct security zones. A Core Router acts as the Firewall and Gateway, routing traffic between the internal VLANs and the external DMZ.

| Zone / VLAN | Name | IP Subnet | Default Gateway |
| :--- | :--- | :--- | :--- |
| VLAN 10 | Management | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | Staff | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | Guest | 192.168.30.0/24 | 192.168.30.1 |
| VLAN 99 | DMZ | 10.0.99.0/24 | 10.0.99.1 |

## 🚀 Implementation Steps

### Step 1: VLAN Segmentation (Access Layer Switch)
Configure the access layer switch to separate traffic into VLANs 10, 20, and 30. The port connecting to the Core Router is configured as a Trunk.

```cisco
! Enter global configuration mode
enable
configure terminal
hostname AccessSwitch

! Create VLANs
vlan 10
 name Management
vlan 20
 name Staff
vlan 30
 name Guest
exit

! Configure Access Ports (Example: FastEthernet 0/1 to Management)
interface range fa0/1 - 5
 switchport mode access
 switchport access vlan 10
exit

! Configure Access Ports (Example: FastEthernet 0/6 to Staff)
interface range fa0/6 - 10
 switchport mode access
 switchport access vlan 20
exit

! Configure Trunk Port to Core Router
interface gig0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
```

### Step 2: Core Router & DMZ Configuration (Router-on-a-Stick)
The Core Router acts as the firewall. It uses subinterfaces to route internal traffic and a physical interface to connect to the DMZ switch.

```cisco
enable
configure terminal
hostname CoreRouter

! Configure DMZ Interface (Connected to DMZ Switch)
interface gig0/0
 ip address 10.0.99.1 255.255.255.0
 no shutdown
exit

! Configure Trunk Interface (Connected to Access Switch)
interface gig0/1
 no shutdown
exit

! Subinterface for VLAN 10 (Management)
interface gig0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

! Subinterface for VLAN 20 (Staff)
interface gig0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

! Subinterface for VLAN 30 (Guest)
interface gig0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
```

### Step 3: Access Control Lists (ACLs)
We implement Standard and Extended ACLs on the Core Router to enforce security policies.

**1. Standard ACL:** Block a specific simulated malicious IP range (e.g., `203.0.113.0/24`) from entering the internal network.
```cisco
! Block external threat range
access-list 10 deny 203.0.113.0 0.0.0.255
access-list 10 permit any
```

**2. Extended ACL:** Restrict HTTP/HTTPS traffic strictly to the DMZ Web Server (`10.0.99.10`). Prevent internal VLANs from hosting web services to the outside world.
```cisco
! Allow HTTP/HTTPS traffic only to the DMZ Web Server
access-list 100 permit tcp any host 10.0.99.10 eq 80
access-list 100 permit tcp any host 10.0.99.10 eq 443

! Deny internal networks from acting as web servers
access-list 100 deny tcp 192.168.10.0 0.0.0.255 any eq 80
access-list 100 deny tcp 192.168.20.0 0.0.0.255 any eq 80
access-list 100 deny tcp 192.168.30.0 0.0.0.255 any eq 80

! Permit all other internal IP traffic
access-list 100 permit ip any any
exit
```

**Applying the ACLs:**
```cisco
! Apply Standard ACL to block traffic coming from the "Internet" (Simulated via Loopback or another router)
interface gig0/2
 ip access-group 10 in
exit

! Apply Extended ACL inbound on the internal trunk interface
interface gig0/1
 ip access-group 100 in
exit
```

### Step 4: Port Security (Access Switch)
To prevent unauthorized devices from plugging into the network, configure Port Security on the access ports. We use the "sticky" MAC feature and set the violation mode to "shutdown".

```cisco
configure terminal
! Apply to Staff ports as an example
interface range fa0/6 - 10
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
exit
exit

! Save the configuration
write memory
```

## 🧪 Verification & Testing

To ensure the infrastructure is secure and functioning, perform the following checks in Packet Tracer:

1. **Inter-VLAN Routing:** Ping from a Staff PC (VLAN 20) to the Management PC (VLAN 10). The ping should succeed.
2. **DMZ Access:** Open a web browser on a Guest PC (VLAN 30) and navigate to `http://10.0.99.10`. The web page should load.
3. **Port Security Testing:**
   * Plug a laptop into a secured port. The port should err-disable (turn red in Packet Tracer).
   * Use the `show port-security interface fa0/6` command on the switch to verify the violation.
   * To restore: `shutdown` then `no shutdown` on the interface.
4. **ACL Testing:**
   * Ping the `203.0.113.1` address from an internal PC. It should fail due to Standard ACL 10.
   * Use `show access-lists` on the router to see hit counters incrementing.

## 📁 Repository Contents
* `Network_Topology_Security.pkt` - The main Cisco Packet Tracer file containing the complete lab setup.
* `README.md` - This documentation file.

---

### 💡 Text for your Portfolio Website
*(Copy and paste this block into the projects section of your personal website)*

**Project: Enterprise Network Security & DMZ Infrastructure**
**Tech Stack:** Cisco IOS, VLANs, ACLs, Port Security, Cisco Packet Tracer

**Overview:**
Designed and simulated a highly secure enterprise network topology focusing on traffic segmentation and strict access controls. The architecture features a Demilitarized Zone (DMZ) isolating public-facing web servers from the internal corporate network.

**Key Achievements:**
* Implemented Layer 2 segmentation using VLANs (Management, Staff, Guest) with 802.1Q trunking and Router-on-a-Stick for inter-VLAN routing.
* Configured a DMZ environment separated by a firewall router to securely expose web services without compromising internal network integrity.
* Engineered Standard and Extended Access Control Lists (ACLs) to block malicious IP ranges and enforce strict HTTP/HTTPS traffic routing exclusively to the DMZ.
* Hardened access-layer switches by deploying Port Security (Sticky MAC, Violation Shutdown) to completely mitigate unauthorized physical network access.
