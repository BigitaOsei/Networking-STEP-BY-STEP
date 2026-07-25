# 3-Department Office Network Design

## Project Overview
This project designs and implements a small office network for three departments:
- **IT** (30 hosts)
- **HR** (20 hosts)
- **Management** (10 hosts)

The network uses **VLSM** for efficient IP addressing and **Router-on-a-Stick** for inter-VLAN routing.

### Technologies Used
- VLANs
- Trunking (802.1Q)
- Subinterfaces
- VLSM (Variable Length Subnet Masking)
- Inter-VLAN Routing

---

## Network Requirements
| Department   | Number of Hosts | VLAN ID |
|--------------|-----------------|---------|
| IT           | 30              | 10      |
| HR           | 20              | 20      |
| Management   | 10              | 30      |

**Base Network:** `192.168.10.0/24`

---

## IP Addressing Plan (VLSM)

| Department   | Network Address   | Subnet Mask       | Usable Range              | Broadcast     | Gateway          |
|--------------|-------------------|-------------------|---------------------------|---------------|------------------|
| IT           | 192.168.10.0      | 255.255.255.224   | 192.168.10.1 - 30         | 192.168.10.31 | 192.168.10.1     |
| HR           | 192.168.10.32     | 255.255.255.224   | 192.168.10.33 - 62        | 192.168.10.63 | 192.168.10.33    |
| Management   | 192.168.10.64     | 255.255.255.240   | 192.168.10.65 - 78        | 192.168.10.79 | 192.168.10.65    |

---

## Topology
Router(config)#encapsulation dot1Q 10
^
% Invalid input detected at '^' marker.
help me fix this errorThe error occurs because encapsulation dot1Q is not a global configuration command. It is only valid under a subinterface.
Correct way to configure it
ciscoRouter(config)# interface GigabitEthernet0/0.10     ← (or FastEthernet0/0.10, etc.)
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Key points

























ModeCommand valid?NotesRouter(config)#❌ NoGlobal configRouter(config-if)#❌ NoPhysical interfaceRouter(config-subif)#✅ YesSubinterface only
Typical router-on-a-stick setup
ciscoRouter(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# no ip address                  ← usually leave the physical interface without an IP

Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Common mistakes that cause this error

Typing the command in global config (Router(config)#)
Typing it on the physical interface instead of a subinterface
Using the wrong interface type/name (check with show ip interface brief)

Would you like me to also show the corresponding switch-side trunk configuration?
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
i am to write this but made a mistake how do i correct it
To correct a wrong IP address on a subinterface, just re-enter the correct ip address command. Cisco overwrites the previous one.
Example
If you typed something wrong:
ciscoRouter(config-subif)# ip address 192.168.10.1 255.255.255.0
Just type the correct one again:
ciscoRouter(config-subif)# ip address 192.168.10.1 255.255.255.0
(or whatever the correct IP/mask should be)
Other useful correction commands

























What you want to doCommandChange the IP addressip address <new-ip> <mask>Remove the IP completelyno ip addressCheck current IP on the interfaceshow ip interface brief or show running-config interface GigabitEthernet0/0.10Exit subinterface modeexit or end
Quick tip
You do not need to remove the old IP first. Simply typing a new ip address command replaces the previous one.
Would you like the full correct configuration sequence for the subinterface (including encapsulation)?yesHere’s the full correct configuration sequence for a subinterface (router-on-a-stick style):
1. Enter the physical interface and prepare it
ciscoRouter(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# no ip address
Router(config-if)# exit
2. Create the subinterface and configure it
ciscoRouter(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
3. (Optional) Add another VLAN/subinterface
ciscoRouter(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
4. Save the configuration
ciscoRouter(config)# end
Router# write memory
or
ciscoRouter# copy running-config startup-config

Quick verification commands
ciscoRouter# show ip interface brief
Router# show interfaces trunk          ← (on the switch side)
Router# show running-config | section interface
Notes:

Replace GigabitEthernet0/0 with whatever physical interface you’re using (e.g. FastEthernet0/0).
The number after the dot (.10) is just a label — it doesn’t have to match the VLAN ID, but most people make it match for clarity.
Make sure the switch port connected to this router interface is configured as a trunk.

Would you like the matching switch trunk configuration as well?Router(config-subif)#ip address 192.168.10.62 255.255.255.224
% 192.168.10.32 overlaps with GigabitEthernet0/0.10This error means the IP/subnet you just tried to configure overlaps with an existing subnet already assigned to GigabitEthernet0/0.10.
Why it happened

Mask 255.255.255.224 = /27
Network for 192.168.10.62 /27 is 192.168.10.32/27
(usable hosts: 192.168.10.33 – 192.168.10.62)

Cisco is telling you that GigabitEthernet0/0.10 already has an IP that falls inside (or is the same as) the 192.168.10.32/27 network.
How to fix it
Step 1 – Check what is already configured
ciscoRouter# show ip interface brief
or more specifically:
ciscoRouter# show running-config interface GigabitEthernet0/0.10
Step 2 – Decide what you want
You have two main choices:
Option A – Change the new IP to a different subnet
Example (using the next /27 network):
ciscoRouter(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.10.94 255.255.255.224
(Network would be 192.168.10.64/27)
Option B – Remove or change the overlapping IP on Gi0/0.10
ciscoRouter(config)# interface GigabitEthernet0/0.10
Router(config-subif)# no ip address
Then you can re-apply the IP you wanted on the correct subinterface.

Quick tip
Cisco does not allow two interfaces (or subinterfaces) to have overlapping IP ranges, even if they are different VLANs.
Would you like me to help you pick non-overlapping subnets for multiple VLANs? Just tell me how many VLANs/subinterfaces you need and the starting network.i think i am confused with this configuration thing,i am a beginner and do not know too much about the cisco configuration ,how to do configuration and do not have understanding for it too
can you take time to take me through well so i can build topology myself nd do cofiguration properlyNo problem at all. Let’s slow down and build this from zero, step by step, so you really understand what you’re doing.
I’ll explain why we do each thing, not just the commands.

1. What are we trying to achieve?
We want different VLANs (different networks) to talk to each other through a router.
This is called Router-on-a-Stick.
Simple example we will use:




















VLANNetworkRouter IP (Gateway)10192.168.10.0 /27192.168.10.120192.168.10.32 /27192.168.10.33
(You can change the numbers later once you understand.)

2. Topology you should build (in Packet Tracer or GNS3)
textPC1 -------- Switch -------- Router
PC2 --------

PC1 → belongs to VLAN 10
PC2 → belongs to VLAN 20
Switch has one cable going to the Router (this cable will carry both VLANs → this is a trunk)


3. Understanding the key idea
On the router, one physical port can handle many VLANs by creating subinterfaces.
Example:

GigabitEthernet0/0     → physical port
GigabitEthernet0/0.10  → subinterface for VLAN 10
GigabitEthernet0/0.20  → subinterface for VLAN 20

Each subinterface needs:

encapsulation dot1Q <vlan-id> → tells the router which VLAN this subinterface belongs to
An IP address → this becomes the default gateway for that VLAN


4. Full correct configuration (copy this style)
A. Router configuration
ciscoRouter> enable
Router# configure terminal

! First prepare the physical interface
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# no ip address
Router(config-if)# exit

! Create subinterface for VLAN 10
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.224
Router(config-subif)# exit

! Create subinterface for VLAN 20
Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.10.33 255.255.255.224
Router(config-subif)# exit

Router(config)# end
Router# write memory
B. Switch configuration (very important)
ciscoSwitch> enable
Switch# configure terminal

! Create the VLANs
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name IT
Switch(config-vlan)# exit

! Put PC1 into VLAN 10
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit

! Put PC2 into VLAN 20
Switch(config)# interface FastEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit

! Make the port connected to the Router a TRUNK
Switch(config)# interface FastEthernet0/24          ← change this to the port you used
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

Switch(config)# end
Switch# write memory
C. PC configuration

PC1:
IP: 192.168.10.10
Subnet mask: 255.255.255.224
Default Gateway: 192.168.10.1

PC2:
IP: 192.168.10.40
Subnet mask: 255.255.255.224
Default Gateway: 192.168.10.33



5. Why you got the “overlap” error earlier
You tried to put two IPs that belong to the same network on two different interfaces.
Cisco does not allow that.
In the example above I carefully chose:

VLAN 10 → network 192.168.10.0 /27   (hosts .1 to .30)
VLAN 20 → network 192.168.10.32 /27  (hosts .33 to .62)

They do not overlap.

6. How to check if everything is working
On the Router:
ciscoRouter# show ip interface brief
Router# show interfaces trunk          ← this one usually works better on the switch
On the Switch:
ciscoSwitch# show vlan brief
Switch# show interfaces trunk
From PC1 try:
textping 192.168.10.33
ping 192.168.10.40
If both pings work → congratulations, inter-VLAN routing is working!

Would you like me to continue with a very detailed explanation of why we use subinterfaces and encapsulation, or would you prefer we first practice with this exact topology and then change the IP numbers together?Can i upload the so you help meYes, you can share it with me — that will make it much easier to help you correctly.
Here’s what works best:
Best options (in order):

Screenshot of your Packet Tracer topology
(just take a clear picture of the whole network)
Screenshot of the configuration you already typed
(especially the router and switch interfaces)
Text of the configuration
You can copy-paste the output of these commands:ciscoshow running-config
show ip interface brief
show vlan brief
Packet Tracer file (.pkt)
Unfortunately I cannot open .pkt files directly, but if you describe the topology or send screenshots + the configuration, I can still guide you perfectly.


Just upload/send whatever you have (screenshot or text), and tell me:

What problem you’re facing right now, or
What you want to achieve

I’ll look at it carefully and guide you step by step.Design a 3-department office network: HR (20 hosts), IT (30 hosts), Management (10 hosts). Take 192.168.10.0/24 and split it into 3 subnets. Write out all addresses for each.
this is the project i am working onHere’s a clean and correct subnetting design for your project.
Original Network
192.168.10.0 /24
Host Requirements





























DepartmentHosts NeededMinimum addresses requiredBest PrefixIT3032/27HR2032/27Management1016/28
We use VLSM (Variable Length Subnet Mask) and assign the largest subnet first.

Final Subnet Allocation
1. IT Department (30 hosts)

































ItemValueNetwork Address192.168.10.0Subnet Mask255.255.255.224 (/27)First Usable IP192.168.10.1Last Usable IP192.168.10.30Broadcast Address192.168.10.31Total usable hosts30
Recommended Gateway (Router subinterface): 192.168.10.1

2. HR Department (20 hosts)

































ItemValueNetwork Address192.168.10.32Subnet Mask255.255.255.224 (/27)First Usable IP192.168.10.33Last Usable IP192.168.10.62Broadcast Address192.168.10.63Total usable hosts30 (more than enough)
Recommended Gateway: 192.168.10.33

3. Management Department (10 hosts)

































ItemValueNetwork Address192.168.10.64Subnet Mask255.255.255.240 (/28)First Usable IP192.168.10.65Last Usable IP192.168.10.78Broadcast Address192.168.10.79Total usable hosts14
Recommended Gateway: 192.168.10.65

Summary Table (easy to copy)





































DepartmentNetworkMaskUsable RangeBroadcastGatewayIT192.168.10.0255.255.255.224192.168.10.1 – 30192.168.10.31192.168.10.1HR192.168.10.32255.255.255.224192.168.10.33 – 62192.168.10.63192.168.10.33Management192.168.10.64255.255.255.240192.168.10.65 – 78192.168.10.79192.168.10.65

Would you like me to also give you the full router + switch configuration using these exact subnets (with subinterfaces for Router-on-a-Stick)?yes please and how to build the topology and with configuration which i have good understanding and can explain my self to my leecturerHere’s a complete, clear, and easy-to-explain solution for your project.

1. Topology Design (How to build it in Packet Tracer)
Devices you need:

1 Router (e.g. 2911 or 1941)
1 Switch (e.g. 2960)
3 PCs (one for each department)
Optional: 1 more PC if you want to test more hosts

Connections:
textPC-IT  --------→ Switch Fa0/1
PC-HR  --------→ Switch Fa0/2
PC-MGMT--------→ Switch Fa0/3
Switch Fa0/24 --→ Router GigabitEthernet0/0
Visual layout:
text[Router]
             |
          (Trunk)
             |
          [Switch]
     /       |       \
  PC-IT    PC-HR    PC-MGMT
 (VLAN 10) (VLAN 20) (VLAN 30)

2. IP Addressing Plan (use this exact plan)





































DepartmentVLANNetworkSubnet MaskGateway (Router)PC Example IPIT10192.168.10.0255.255.255.224192.168.10.1192.168.10.10HR20192.168.10.32255.255.255.224192.168.10.33192.168.10.40Management30192.168.10.64255.255.255.240192.168.10.65192.168.10.70

3. Full Configuration
A. Router Configuration
ciscoRouter> enable
Router# configure terminal
Router(config)# hostname Office-Router

! Prepare the physical interface
Office-Router(config)# interface GigabitEthernet0/0
Office-Router(config-if)# no shutdown
Office-Router(config-if)# no ip address
Office-Router(config-if)# exit

! ===== IT Department (VLAN 10) =====
Office-Router(config)# interface GigabitEthernet0/0.10
Office-Router(config-subif)# encapsulation dot1Q 10
Office-Router(config-subif)# ip address 192.168.10.1 255.255.255.224
Office-Router(config-subif)# exit

! ===== HR Department (VLAN 20) =====
Office-Router(config)# interface GigabitEthernet0/0.20
Office-Router(config-subif)# encapsulation dot1Q 20
Office-Router(config-subif)# ip address 192.168.10.33 255.255.255.224
Office-Router(config-subif)# exit

! ===== Management Department (VLAN 30) =====
Office-Router(config)# interface GigabitEthernet0/0.30
Office-Router(config-subif)# encapsulation dot1Q 30
Office-Router(config-subif)# ip address 192.168.10.65 255.255.255.240
Office-Router(config-subif)# exit

Office-Router(config)# end
Office-Router# write memory
B. Switch Configuration
ciscoSwitch> enable
Switch# configure terminal
Switch(config)# hostname Office-Switch

! Create VLANs
Office-Switch(config)# vlan 10
Office-Switch(config-vlan)# name IT
Office-Switch(config-vlan)# exit

Office-Switch(config)# vlan 20
Office-Switch(config-vlan)# name HR
Office-Switch(config-vlan)# exit

Office-Switch(config)# vlan 30
Office-Switch(config-vlan)# name MANAGEMENT
Office-Switch(config-vlan)# exit

! Assign access ports to VLANs
Office-Switch(config)# interface FastEthernet0/1
Office-Switch(config-if)# switchport mode access
Office-Switch(config-if)# switchport access vlan 10
Office-Switch(config-if)# exit

Office-Switch(config)# interface FastEthernet0/2
Office-Switch(config-if)# switchport mode access
Office-Switch(config-if)# switchport access vlan 20
Office-Switch(config-if)# exit

Office-Switch(config)# interface FastEthernet0/3
Office-Switch(config-if)# switchport mode access
Office-Switch(config-if)# switchport access vlan 30
Office-Switch(config-if)# exit

! Make the port connected to the Router a Trunk
Office-Switch(config)# interface FastEthernet0/24
Office-Switch(config-if)# switchport mode trunk
Office-Switch(config-if)# exit

Office-Switch(config)# end
Office-Switch# write memory
C. PC Configuration





























PCIP AddressSubnet MaskDefault GatewayPC-IT192.168.10.10255.255.255.224192.168.10.1PC-HR192.168.10.40255.255.255.224192.168.10.33PC-MGMT192.168.10.70255.255.255.240192.168.10.65

4. How to Test
From any PC, try these pings:
textping 192.168.10.1      ← Gateway of IT
ping 192.168.10.33     ← Gateway of HR
ping 192.168.10.65     ← Gateway of Management
ping 192.168.10.40     ← PC in HR
ping 192.168.10.70     ← PC in Management
If all pings succeed → your inter-VLAN routing is working perfectly.

5. How to Explain this to your Lecturer (simple & clear)
You can say:
“I designed a 3-department network using Router-on-a-Stick.I took the network 192.168.10.0/24 and divided it using VLSM:
IT needs 30 hosts → I gave it a /27 subnet
HR needs 20 hosts → another /27 subnet
Management needs 10 hosts → a /28 subnet
On the switch, I created three VLANs (10, 20, 30) and put each department’s PC into its own VLAN using access ports.The link between the switch and the router is a trunk so that traffic from all three VLANs can travel on one cable.On the router, I created three subinterfaces. Each subinterface has:
encapsulation dot1Q to identify the VLAN
An IP address that acts as the default gateway for that department
This way, devices in different VLANs can communicate through the router.”

Would you like me to also give you:

The exact commands to verify everything (show commands), or
A short written report/explanation you can submit?
please work on real life projrcts on put them on my github accoutI understand what you’re asking, but I need to be clear about what I can and cannot do:
What I cannot do:

I cannot log into your GitHub account
I cannot push files or create repositories on your GitHub
I cannot access any of your private accounts

What I can do for you:
I can help you create real-life networking projects (like the one we just did) that are good for GitHub, and then guide you on how to upload them yourself.
Here are some good real-life beginner-to-intermediate networking projects you can put on GitHub:









































ProjectDescriptionDifficultyGood for Portfolio1. Office Network (3 Departments)The exact project we just did (Router-on-a-Stick + VLSM)Beginner★★★★☆2. Small Company NetworkAdd DHCP, DNS, and basic ACLsBeginner+★★★★☆3. Multi-floor OfficeMultiple switches + VLANs + Inter-VLAN routingIntermediate★★★★★4. Secure Office NetworkAdd ACLs to control traffic between departmentsIntermediate★★★★★5. Dual Router NetworkTwo routers with static routing or OSPFIntermediate★★★★☆

Would you like me to:

Finish the current project properly (add documentation, explanation, verification steps) so it’s ready for GitHub?
Create a new real-life project for you (choose one from the list above)?
Show you exactly how to upload a networking project to GitHub (step-by-step)?

Just tell me which option you prefer (1, 2, or 3), or if you have another project idea in mind.1Here’s a complete, GitHub-ready version of your project.
You can create a new repository on GitHub (for example: 3-Department-Office-Network) and upload these files.

Recommended GitHub Structure
text3-Department-Office-Network/
├── README.md
├── Topology/
│   └── topology-description.md
├── Configurations/
│   ├── Router-Config.txt
│   └── Switch-Config.txt
├── Addressing-Plan/
│   └── IP-Addressing.md
└── Verification/
    └── Testing-Steps.md

1. README.md (Main file – copy this)
Markdown# 3-Department Office Network Design

## Project Overview
This project designs and implements a small office network for three departments:
- **IT** (30 hosts)
- **HR** (20 hosts)
- **Management** (10 hosts)

The network uses **VLSM** for efficient IP addressing and **Router-on-a-Stick** for inter-VLAN routing.

### Technologies Used
- VLANs
- Trunking (802.1Q)
- Subinterfaces
- VLSM (Variable Length Subnet Masking)
- Inter-VLAN Routing

---

## Network Requirements
| Department   | Number of Hosts | VLAN ID |
|--------------|-----------------|---------|
| IT           | 30              | 10      |
| HR           | 20              | 20      |
| Management   | 10              | 30      |

**Base Network:** `192.168.10.0/24`

---

## IP Addressing Plan (VLSM)

| Department   | Network Address   | Subnet Mask       | Usable Range              | Broadcast     | Gateway          |
|--------------|-------------------|-------------------|---------------------------|---------------|------------------|
| IT           | 192.168.10.0      | 255.255.255.224   | 192.168.10.1 - 30         | 192.168.10.31 | 192.168.10.1     |
| HR           | 192.168.10.32     | 255.255.255.224   | 192.168.10.33 - 62        | 192.168.10.63 | 192.168.10.33    |
| Management   | 192.168.10.64     | 255.255.255.240   | 192.168.10.65 - 78        | 192.168.10.79 | 192.168.10.65    |

---

## Topology
[Router]
|
Trunk Link
|
[Switch]
/       |       
PC-IT    PC-HR    PC-MGMT
(VLAN 10) (VLAN 20) (VLAN 30)
text**Connections:**
- PC-IT    → Switch FastEthernet0/1 (Access – VLAN 10)
- PC-HR    → Switch FastEthernet0/2 (Access – VLAN 20)
- PC-MGMT  → Switch FastEthernet0/3 (Access – VLAN 30)
- Switch FastEthernet0/24 → Router GigabitEthernet0/0 (Trunk)

---

## How to Implement

1. Build the topology in Cisco Packet Tracer or GNS3
2. Configure the Switch (VLANs + Access ports + Trunk)
3. Configure the Router (Subinterfaces)
4. Configure the PCs with correct IP addresses
5. Test connectivity

---

## Verification Commands

**On Router:**
```cisco
show ip interface brief
show ip route
show running-config
On Switch:
ciscoshow vlan brief
show interfaces trunk
show running-config
From PCs:
textping <gateway>
ping <PC in another department>

Author
[Your Name]
Student – Networking Project
text---

### 2. Configurations/Router-Config.txt

```cisco
! ========================================
! Router Configuration
! 3-Department Office Network
! ========================================

enable
configure terminal
hostname Office-Router

! Physical Interface
interface GigabitEthernet0/0
 no shutdown
 no ip address
 exit

! IT Department - VLAN 10
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.224
 exit

! HR Department - VLAN 20
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.10.33 255.255.255.224
 exit

! Management Department - VLAN 30
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.10.65 255.255.255.240
 exit

end
write memory

3. Configurations/Switch-Config.txt
cisco! ========================================
! Switch Configuration
! 3-Department Office Network
! ========================================

enable
configure terminal
hostname Office-Switch

! Create VLANs
vlan 10
 name IT
 exit

vlan 20
 name HR
 exit

vlan 30
 name MANAGEMENT
 exit

! Access Ports
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
 exit

! Trunk Port to Router
interface FastEthernet0/24
 switchport mode trunk
 exit

end
write memory

4. Addressing-Plan/IP-Addressing.md
Markdown# IP Addressing Plan

## Base Network
192.168.10.0/24

## Subnet Calculations (VLSM)

### IT Department (30 hosts)
- Required: 32 addresses → /27
- Network: 192.168.10.0/27
- Mask: 255.255.255.224
- Usable: 192.168.10.1 – 192.168.10.30
- Broadcast: 192.168.10.31
- Gateway: 192.168.10.1

### HR Department (20 hosts)
- Required: 32 addresses → /27
- Network: 192.168.10.32/27
- Mask: 255.255.255.224
- Usable: 192.168.10.33 – 192.168.10.62
- Broadcast: 192.168.10.63
- Gateway: 192.168.10.33

### Management Department (10 hosts)
- Required: 16 addresses → /28
- Network: 192.168.10.64/28
- Mask: 255.255.255.240
- Usable: 192.168.10.65 – 192.168.10.78
- Broadcast: 192.168.10.79
- Gateway: 192.168.10.65

5. Verification/Testing-Steps.md
Markdown# Testing & Verification Steps

## 1. Check Router Interfaces
```cisco
Office-Router# show ip interface brief
Expected: All three subinterfaces should show "up/up"
2. Check VLANs on Switch
ciscoOffice-Switch# show vlan brief
3. Check Trunk
ciscoOffice-Switch# show interfaces trunk
4. Connectivity Tests (from PCs)








































TestCommandExpected ResultIT Gatewayping 192.168.10.1SuccessHR Gatewayping 192.168.10.33SuccessMGMT Gatewayping 192.168.10.65SuccessIT → HRping 192.168.10.40SuccessIT → MGMTping 192.168.10.70SuccessHR → MGMTping 192.168.10.70Success
text---

### How to Upload to GitHub (Simple Steps)

1. Go to [github.com](https://github.com) and create a **New Repository**
2. Name it: `3-Department-Office-Network`
3. Click **Create repository**
4. Click **Add file → Create new file**
5. Copy and paste the content of each file above
6. Commit the changes

---

Would you like me to also create a short **Project Report** (in case your lecturer asks for a written document)?how to create respositoryHere’s the simplest way to create a GitHub repository:
Step-by-step (Website method – easiest for beginners)

Go to https://github.com and log in to your account.
On the top right, click the “+” icon.
Click “New repository”.
Fill in the details:
Repository name: 3-Department-Office-Network (or any name you like)
Description (optional): Cisco Packet Tracer project - 3 Department Office Network using VLSM and Router-on-a-Stick
Choose Public (recommended so people can see it)
Do NOT check “Add a README file” yet (we will add files ourselves)

Click the green button “Create repository”.


After creating the repository
You will see a page with instructions.
The easiest way for beginners is:

Click “Add file” → “Create new file”
Name the file README.md
Paste the README content I gave you earlier
Scroll down and click “Commit new file”

Then repeat the same process for the other files (Router-Config.txt, Switch-Config.txt, etc.).

Would you like me to also show you the next steps after creating the repository (how to add all the project files one by one)?this is my interfaceI can see your GitHub repository:
Repository name: Networking-STEP-BY-STEP
Owner: BigitaOsei
Current file: config.pkt
You’re already inside your repository — good.
How to add the new project files
Follow these steps carefully:
Step 1: Go back to the main page of the repository

Click on Networking-STEP-BY-STEP (the blue text near the top left).

Step 2: Create a new folder for the project (recommended)

Click the Add file button (top right)
Click Create new file
In the name box, type this exactly:

text3-Department-Office-Network/README.md
(This will automatically create a folder called 3-Department-Office-Network and a file called README.md inside it)

Paste the README.md content I gave you earlier
Scroll down and click Commit new file


Step 3: Add the other files
Repeat the same process for the remaining files:

























File to createWhat to paste3-Department-Office-Network/Router-Config.txtRouter configuration3-Department-Office-Network/Switch-Config.txtSwitch configuration3-Department-Office-Network/IP-Addressing.mdIP Addressing plan3-Department-Office-Network/Testing-Steps.mdVerification steps

Tip:
Every time you create a new file, make sure the path starts with:
text3-Department-Office-Network/
This keeps all the project files neatly inside one folder.

Would you like me to guide you one file at a time?
Just say “next” and I’ll tell you exactly what to do for the next file.cannot find the add fileI can see your repository main page. The “Add file” button is a bit hidden sometimes.
Here’s exactly where to find it:

Look at the top of the file list (where you see config.pkt).
On the right side, next to the green “Code” button, you should see a button that says:Add file ▼(It has a small arrow next to it)
Click on Add file.
Then choose Create new file.


Alternative way (if you still can’t see it):

Click the green “Code” button.
Or simply refresh the page (F5 or Ctrl + R).
Sometimes the button only appears when you are logged in properly.


Can you check again and tell me if you can now see the “Add file” button?
If you still can’t find it, take another screenshot of the top part of the page (especially the area near the green “Code” button) and send it to me. I’ll guide you from there.3-Department-Office-Network/README.md
where to i put this nowYou are now on the Create new file page.
Do this exactly:

Look at the box where it says “Name your file...”
Delete whatever is written there.
Type this exactly (copy and paste it):

text3-Department-Office-Network/README.md

After typing that, GitHub will automatically create a folder called 3-Department-Office-Network.
Now scroll down to the big empty text area and paste the README content I gave you earlier.
After pasting, scroll to the bottom and click the green button “Commit new file”.


Would you like me to paste the full README content again so you can easily copy it?3-Department-Office-Network/README.md
where do i put thisYou are currently looking at an old commit page. That’s why you can’t create a new file from here.
Do this to go back to the correct place:

At the top left, click on this blue text:Networking-STEP-BY-STEP(It is next to your username BigitaOsei)
This will take you back to the main page of your repository (where you can see the config.pkt file).
Once you are on the main page, look for the button “Add file” (near the green “Code” button) and click it.
Then choose “Create new file”.
Now in the filename box, type:

text3-Department-Office-Network/README.md

Please click Networking-STEP-BY-STEP at the top and tell me when you are back on the main page. I’ll guide you from there.at the main page nowGreat! Now follow these steps carefully:
Step 1:
Look near the top right of the file list (next to the green Code button).
You should see a button that says:
Add file ▼
Click on Add file.
Step 2:
A small menu will appear. Click on:
Create new file
Step 3:
You will now see a page with a box that says “Name your file...”
In that box, type (or copy-paste) exactly this:
text3-Department-Office-Network/README.md
Step 4:
After typing the name, a big empty text area will appear below.
Tell me when you reach this point, and I will give you the full content to paste into that box.this is nothing like that but add a file via uploadI think I know the problem.
Looking at your previous screenshots, I can see “Sign in” and “Sign up” buttons in the top right corner. This means you are not logged in to your GitHub account.
Please do this:

Click the Sign in button (top right).
Log in with your GitHub username and password.
After logging in, go back to your repository:
https://github.com/BigitaOsei/Networking-STEP-BY-STEP
Once you are logged in, you will see the full options, including:
Add file → Create new file



Can you try signing in and then tell me if the “Add file” button appears?done signing inPerfect! Now that you are signed in, let’s continue.
Do these steps:

Make sure you are on the main page of your repository:
https://github.com/BigitaOsei/Networking-STEP-BY-STEP
Look near the top, next to the green Code button.You should now see a button that says:Add file ▼
Click Add file.
Then click Create new file.
In the box that says “Name your file…”, type this exactly:

text3-Department-Office-Network/README.md

After typing the name, a large empty box will appear below for the content.

Tell me when you reach the large empty box, and I will give you the full README content to paste.continueGreat. Now paste this content into the big empty box:
Markdown# 3-Department Office Network Design

## Project Overview
This project designs and implements a small office network for three departments:
- **IT** (30 hosts)
- **HR** (20 hosts)
- **Management** (10 hosts)

The network uses **VLSM** for efficient IP addressing and **Router-on-a-Stick** for inter-VLAN routing.

### Technologies Used
- VLANs
- Trunking (802.1Q)
- Subinterfaces
- VLSM (Variable Length Subnet Masking)
- Inter-VLAN Routing

---

## Network Requirements
| Department   | Number of Hosts | VLAN ID |
|--------------|-----------------|---------|
| IT           | 30              | 10      |
| HR           | 20              | 20      |
| Management   | 10              | 30      |

**Base Network:** `192.168.10.0/24`

---

## IP Addressing Plan (VLSM)

| Department   | Network Address   | Subnet Mask       | Usable Range              | Broadcast     | Gateway          |
|--------------|-------------------|-------------------|---------------------------|---------------|------------------|
| IT           | 192.168.10.0      | 255.255.255.224   | 192.168.10.1 - 30         | 192.168.10.31 | 192.168.10.1     |
| HR           | 192.168.10.32     | 255.255.255.224   | 192.168.10.33 - 62        | 192.168.10.63 | 192.168.10.33    |
| Management   | 192.168.10.64     | 255.255.255.240   | 192.168.10.65 - 78        | 192.168.10.79 | 192.168.10.65    |

---

## Topology
[Router]
|
Trunk Link
|
[Switch]
/       |       
PC-IT    PC-HR    PC-MGMT
(VLAN 10) (VLAN 20) (VLAN 30)

**Connections:**
- PC-IT    → Switch FastEthernet0/1 (Access – VLAN 10)
- PC-HR    → Switch FastEthernet0/2 (Access – VLAN 20)
- PC-MGMT  → Switch FastEthernet0/3 (Access – VLAN 30)
- Switch FastEthernet0/24 → Router GigabitEthernet0/0 (Trunk)

---

## How to Implement

1. Build the topology in Cisco Packet Tracer or GNS3
2. Configure the Switch (VLANs + Access ports + Trunk)
3. Configure the Router (Subinterfaces)
4. Configure the PCs with correct IP addresses
5. Test connectivity

---

## Verification Commands

**On Router:**
```cisco
show ip interface brief
show ip route
show running-config

On switch
show vlan brief
show interfaces trunk
show running-config

PC
ping <gateway>
ping <PC in another department>

Author
BigitaOsei
Student – Networking Project
