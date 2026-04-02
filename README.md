# L2VPN Implementation using Open vSwitch & Mininet

## Project Overview
This project presents an advanced implementation of Layer 2 Virtual Private Networks (L2VPN) utilizing **VXLAN** tunneling over an emulated IP core network. The topology and simulation are built using the **Mininet** network emulator and **Open vSwitch (OVS)** software switches.

The primary goal of this project is to demonstrate network traffic isolation (Multi-tenancy) at Layer 2 for different clients (VPN A and VPN B) distributed across multiple physical sites, while sharing the same underlying physical infrastructure and access switches. This is achieved using MAC address forwarding (FDB - Forwarding Database) mechanisms and the OpenFlow protocol.

### Objectives:
* **VXLAN Encapsulation:** Utilizing Virtual Tunnel Endpoint (VTEP) interfaces on edge routers to encapsulate Ethernet frames into UDP/IP packets.
* **Traffic Steering:** Programming OVS switches to intelligently forward traffic between hosts and the edge router without the need for traditional VLANs.

---

## Tech Stack
* **Network Emulation:** Mininet
* **Switching & SDN:** Open vSwitch (OVS) with OpenFlow
* **Scripting Language:** Python 3 (Mininet topology scripts)
* **Traffic Analysis:** Wireshark

---

## Network Architecture and Topology
The script implements a topology consisting of four main sites connected via a core network. The architecture is divided into the following layers:

### 1. Core Layer (IP Underlay)
* **Nodes:** Linux system routers.
* **Role:** Providing pure IP transport (Underlay) using static routing. The core network is "unaware" of the existence of Virtual Private Networks (VPNs)—its sole purpose is to deliver IP packets between the edge routers.

### 2. Edge Layer (Edge Routers / VTEP)
* **Nodes:** Edge routers.
* **Role:** Acting as VXLAN tunnel endpoints (VTEPs). Each edge router contains:
    * A system bridge (`br-edge`) connecting local traffic with tunnel interfaces.
    * A **vxlan10** interface (VNI 10) for VPN A traffic.
    * A **vxlan20** interface (VNI 20) for VPN B traffic.

### 3. Access Layer
* **Nodes:** Open vSwitch switches.
* **Role:** Aggregating host connections and forwarding them (Uplink) to the edge router. These switches operate based on static OpenFlow rules managing traffic on specific ports, completely replacing traditional MAC-learning and VLAN-based switching.

### 4. Host Layer (End Users)
* Each of the 4 sites contains hosts assigned to their respective VPN instances.
* **VPN A:**
    * IP Addressing: `192.168.0.x/24`
    * MAC Space: `00:00:00:0a:xx:xx`
* **VPN B:**
    * IP Addressing: `192.168.1.x/24`
    * MAC Space: `00:00:00:0b:xx:xx`

---

## Forwarding Logic

When `Host 1` from **VPN A** (Site 1) attempts to communicate with `Host 2` from **VPN A** (Site 2):
1. The **Host** has a static ARP entry, so it immediately generates an Ethernet frame with the destination host's MAC address.
2. The **Switch (OVS)**, via an OpenFlow rule (Priority 100), directs the frame straight to the Uplink port towards the edge router.
3. The **Edge Router (VTEP)** analyzes its static FDB table. It discovers that the destination MAC address is located behind a remote IP address (belonging to another edge router) reachable via the `vxlan10` tunnel.
4. The L2 frame is encapsulated into a UDP/IP (VXLAN) packet and transmitted across the core network.
5. The remote edge router receives the packet, strips the VXLAN header, and forwards the original L2 frame to its local switch, which then delivers it to the destination host.
