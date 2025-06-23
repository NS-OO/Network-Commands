# Network-Vlan config

To set up Inter-VLAN Routing on your management (Layer 3) switch, you'll essentially be creating Switch Virtual Interfaces (SVIs)—logical interfaces serving as gateways for each VLAN—and enabling IP routing. Here's a streamlined guide:
________________________________________
🔧 Step 1: Create VLANs & Assign Ports
On your managed switch, define your VLANs and assign ports:
bash
CopyEdit
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name Admin
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name Finance
! Repeat for additional VLANs
Assign access ports:
bash
CopyEdit
Switch(config)# interface range Gig0/1–10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10

Switch(config)# interface range Gig0/11–20
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Configure a trunk link to carry multiple VLANs:
bash
CopyEdit
Switch(config)# interface Gig0/24
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
________________________________________
🔁 Step 2: Enable Layer 3 Routing
Activate routing capabilities:
bash
CopyEdit
Switch(config)# ip routing
This allows the switch to route between VLAN subnets like a router on a stick—but entirely internally. 
________________________________________
🛣️ Step 3: Create SVIs (Default Gateways)
Create an SVI for each VLAN and assign its IP (gateway):
bash
CopyEdit
Switch(config)# interface Vlan10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown

Switch(config)# interface Vlan20
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
Each SVI acts as the default gateway for its VLAN, enabling the switch to route traffic between them. 
________________________________________
💡 Step 4: Configure Hosts & Routing
•	On client devices, set the default gateway to the respective SVI IP (e.g., 192.168.10.1 for VLAN 10).
•	Create a default route on the switch pointing to your firewall for internet access:
bash
CopyEdit
Switch(config)# ip route 0.0.0.0 0.0.0.0 <firewall-IP>
•	Ensure your firewall has return routes for each VLAN subnet back to the switch.
For multiple VLANs, add static routes on the firewall—or use a routing protocol like OSPF if supported. 
________________________________________
🔍 Example Overview
Component	Command / IP
VLAN 10 SVI	interface Vlan10 → 192.168.10.1/24
VLAN 20 SVI	interface Vlan20 → 192.168.20.1/24
Access/trunk ports	Assign ports via switchport access/trunk
IP routing	ip routing
Default route	ip route 0.0.0.0 0.0.0.0 <firewall-IP>
________________________________________
✅ Final Checklist
1.	VLANs created & ports assigned
2.	Trunk link configured to carry VLANs
3.	ip routing enabled
4.	SVIs created with IPs and up
5.	Hosts use SVI IPs as gateways
6.	Default Internet route on switch towards firewall
7.	Firewall routes back to VLAN subnets
________________________________________
⚠️ Gotchas
•	Ensure trunk mode is explicitly set (avoid DTP auto negotiation) 
•	Avoid using VLAN 1 as your native VLAN for security
•	Remember: SVIs are required only for VLANs that need routing 

To assign IP addressing along with subnet masks for 4 VLANs starting from the base IP 10.0.0.10, we need to decide how many hosts per VLAN you expect. Here’s a clean and scalable scheme assuming each VLAN has up to 254 hosts, which corresponds to a /24 subnet mask.
________________________________________
✅ VLAN IP Addressing Plan (Starting from 10.0.0.10)
VLAN Name	VLAN ID	Subnet	Subnet Mask	Gateway IP (Switch IP)
VLAN10	10	10.0.0.0/24	255.255.255.0	10.0.0.1
VLAN20	20	10.0.1.0/24	255.255.255.0	10.0.1.1
VLAN30	30	10.0.2.0/24	255.255.255.0	10.0.2.1
VLAN40	40	10.0.3.0/24	255.255.255.0	10.0.3.1
________________________________________
🖥️ Sample Switch Configuration Lines (for Layer 3 switch)
bash
CopyEdit
interface vlan 10
 ip address 10.0.0.1 255.255.255.0
 no shutdown

interface vlan 20
 ip address 10.0.1.1 255.255.255.0
 no shutdown

interface vlan 30
 ip address 10.0.2.1 255.255.255.0
 no shutdown

interface vlan 40
 ip address 10.0.3.1 255.255.255.0
 no shutdown
Each device in VLAN10 would get an address like 10.0.0.10, 10.0.0.11, etc., with 10.0.0.1 as the gateway.
