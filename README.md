
Multi-Branch Corporate Network Design
Enterprise Network Simulation Portfolio Project

1. Project Overview & Context

In the modern enterprise landscape—particularly within high-security, high-availability sectors like banking, telecommunications, and distributed corporations—a robust and secure Wide Area Network (WAN) is the backbone of daily operations. 

This project involves the design, structural implementation, and end-to-end verification of a simulated three-site enterprise network built using **Cisco Packet Tracer 8.2.2**. The entire infrastructure is modeled after real-world corporate deployment architectures, focusing on scalability, strict data separation, logical traffic routing, and high-performance switching.

<img width="1542" height="563" alt="image" src="https://github.com/user-attachments/assets/ca053839-9e8b-4913-b8cd-58602516229a" />


Why This Project Matters
Rather than deploying a flat network layout, this project showcases a deep understanding of hierarchical network architecture. It demonstrates how to successfully bridge multiple geographic regions while managing internal department security policies, centralizing core network services, and optimizing bandwidth consumption over simulated point-to-point leased lines.

2. Business Scenario & Project Goals

The Scenario
A rapidly expanding enterprise requires a unified network infrastructure connecting its primary corporate headquarters in **Abuja** to its major regional operational hubs in **Lagos** and **Port Harcourt**. 

Each location houses distinct organizational departments (e.g., HR, Finance, IT, Sales, Operations) that generate unique traffic patterns and carry varying levels of data sensitivity. 

### Core Project Objectives
To meet corporate operational and security compliance mandates, the network was engineered to achieve the following five core objectives:

Logical Traffic Segmentation:** Isolate departmental broadcast domains utilizing IEEE 802.1Q Virtual Local Area Networks (VLANs) to boost performance and mitigate internal security risks.
Geographic Interconnectivity:** Establish high-reliability Wide Area Network (WAN) serial links to seamlessly route corporate data across diverse geographical regions.
Dynamic and Resilient Routing:** Deploy Open Shortest Path First (OSPF) Area 0 to facilitate automated, intelligent, and loop-free route discovery across all corporate segments.
Centralized Infrastructure Services:** Implement a single, centralized DHCP architecture at the Abuja Headquarters to dynamically manage and distribute IP allocations across all three sites.
Granular Security Enforcement:** Construct and deploy Extended Access Control Lists (ACLs) at the network core to enforce zero-trust properties between sensitive departments and restrict infrastructure access.

---

3. Structural Architecture & Design

The network is architected around a three-tier hierarchical layout (Core, Distribution, and Access), optimized per site based on operational scale and localized device availability.

3.1 Regional Site Matrix

| Site | Operational Role | Local Device Inventory | Active Departmental Profiles |
| :--- | :--- | :--- | :--- |
| **Abuja HQ** | Central Core / Data Center | Layer 3 Core Switch, WAN Gateway Router, Access Switches, Corporate Server | HR, Finance, IT, Management, Core Server Infrastructure |
| **Lagos Branch** | Regional Operations Hub | Edge WAN Router, Dual Access Switches | Sales, Finance, IT, Operations |
| **Port Harcourt** | Regional Operations Hub | Edge WAN Router, Dual Access Switches | Sales, Finance, IT, Operations |

3.2 WAN Backbone Design
Point-to-point links between routers utilize a highly efficient **`/30` subnetting architecture**, ensuring zero address wastage by allocating exactly two usable IP endpoints per physical link.

**Abuja HQ to Lagos Branch:** Connected via high-speed Serial interface (`10.0.1.0/30`), with the Abuja HQ acting as the Data Communications Equipment (DCE) providing the master clocking signal.
* **Abuja HQ to Port Harcourt Branch:** Connected via Serial interface (`10.0.2.0/30`), similarly controlled via master clocking from the HQ gateway.
* **Abuja Core Routing Path:** A dedicated GigabitEthernet connection (`10.0.0.0/30`) links the Abuja WAN Gateway Router directly to the Layer 3 Core Switch, ensuring ultra-low latency line-rate routing between the WAN and internal LAN segments.

3.3 IP Addressing Strategy
To prevent IP conflicts and maintain pristine route summarization properties, a unique, structural Class C IP scheme (`192.168.x.0/24`) was mapped out across the enterprise:

* Abuja HQ Segments:** Subnets span `192.168.10.0/24` through `192.168.40.0/24`, with a dedicated isolated subnet (`192.168.99.0/24`) for the data center server.
* Lagos Branch Segments:** Sequenced cleanly from `192.168.11.0/24` to `192.168.41.0/24`.
* Port Harcourt Segments:** Sequenced cleanly from `192.168.12.0/24` to `192.168.42.0/24`.

---

4. Implementation Strategy & Technical Phases

The project was executed through a systematic, multi-phase methodology to minimize structural configuration errors and facilitate modular testing.


Phase 1: Physical Topology -> Phase 2: Layer 2 VLAN/Trunks -> Phase 3: Inter-VLAN Routing
|
Phase 7: Security ACLs   <- Phase 6: Central DHCP  <- Phase 5: OSPF Area 0 <- Phase 4: WAN Links


Phase 1: Physical Topology & Layer 1 Connectivity
All network nodes were deployed across three distinct logic zones. Physical hardware modules (such as HWIC-2T high-speed serial cards) were retrofitted into Cisco 2911 ISR routers to establish physical WAN layer compatibility.

Phase 2: Layer 2 Segmentation & Trunking
VLANs were created and uniformly named across all access layer switches. Inter-switch uplinks were configured strictly as 802.1Q trunk lines, allowing a single physical link to securely transport traffic for multiple distinct departmental networks simultaneously.

Phase 3: Core Inter-VLAN Routing
* At Headquarters:** A high-performance Layer 3 Multilayer Switch (`Cisco 3650`) was chosen to handle inter-VLAN routing natively using Switch Virtual Interfaces (SVIs). This ensures that intra-HQ departmental traffic is routed at hardware wire-speed without burdening the perimeter WAN gateway router.
* At Regional Branches:** A **Router-on-a-Stick (RoaS)** topology was implemented on the `2911` branch routers. Using single physical interfaces divided into logical, tagged 802.1Q subinterfaces, the branch routers successfully perform multi-VLAN routing with minimal hardware overhead.

Phase 4: Dynamic WAN Routing Protocol (OSPF Area 0)
To eliminate the administrative overhead and fragility of static routing, **OSPF (Open Shortest Path First)** was chosen as the enterprise Interior Gateway Protocol (IGP). 
* A single, high-speed backbone area (**Area 0**) was established across the Layer 3 Switch and all three WAN routers.
* Wildcard masking was meticulously applied to advertise only local valid subnets, enabling rapid route convergence and automated path selection.

Phase 5: Centralized Infrastructure Services (DHCP)
To streamline IT administration, a central server stationed in the Abuja Data Center handles IP distribution for the entire enterprise. 
* Twelve individual DHCP scopes were engineered on the central server, assigning default gateways, lease parameters, and DNS details specific to each branch department.
* Because DHCP requests rely heavily on local layer 2 broadcasts (which are fundamentally blocked by Layer 3 routers and core switches), the `ip helper-address` directive was deployed globally on all SVIs and branch subinterfaces. This converts local broadcast discovery traffic into targeted unicast streams, safely guiding them across the WAN directly to the central server.

Phase 6: Advanced Security & Policy Enforcement
With all sites fully reachable, a strict zero-trust security policy was implemented at the core via Extended Access Control Lists (ACLs). Applied inbound on the core SVIs, the security rules enforce the following business-critical policies:
1. Server Farm Protection:** Only the specialized IT department network (`192.168.30.0/24`) is granted full access to the corporate server subnet. All other corporate segments are structurally denied.
2. Corporate Segregation (Least Privilege):** A bidirectional traffic block was established between the Finance and HR departments. While both departments can access general network resources and communicate with external branches, they are completely barred from probing each other's internal local subnets.



5. Verification, Testing & Troubleshooting Summary

A network design is only as good as its proven stability. Post-implementation, rigorous testing was executed across all layers to ensure seamless validation.

5.1 End-to-End Test Matrix
* Intra-Site & Cross-Site Reachability:** Endpoints across all three sites achieved complete connectivity via ping verification, validating that the OSPF topology tables were fully populated and functional.
* DHCP Verification:** Endpoints across Lagos and Port Harcourt successfully bypassed local routing boundaries, pulled accurate configurations from the Abuja server, and populated their respective IP configurations without generating APIPA anomalies.
* **Security Policy Compliance:** Extended ACLs behaved exactly as modeled. Pings from Finance hosts to HR hosts failed with explicit "Destination host unreachable" notices, while pings from IT hosts to the Data Center Server completed with zero packet loss.

5.2 Real-World Engineering Lessons Learned
* Clock-Rate Synchronization:** Simulated Serial links initially refused to initialize until proper master clocking signals (`clock rate 64000`) were manually bound to the designated DCE edge boundaries on the Abuja gateway router.
* OSPF Network Boundary Alignment:** Mismatched subnet masks on point-to-point connections initially stalled OSPF neighbor formation. Aligning masks precisely to identical `/30` configurations immediately brought up adjacency states to `FULL`.
* Overcoming Overlapping Subnets:** Early design iterations where regional sites utilized matching VLAN subnets resulted in major OSPF routing loops. Redesigning the infrastructure with unique, non-overlapping blocks per region immediately resolved all global routing conflicts.


## 6. Technical Skills Demonstrated Portfolio Value

This portfolio project stands as a definitive proof-of-concept for enterprise-grade network design, consolidating several key administrative competencies:

* Architectural Engineering:** Mastery of multi-site WAN scaling, point-to-point link optimization, and precise VLSM (Variable Length Subnet Mask) engineering.
* Advanced Switching & Routing Infrastructure:** Hands-on competence configuring Layer 3 Multi-Layer Switches, SVIs, Router-on-a-Stick subinterface convergence, and native 802.1Q encapsulation parameters.
* Dynamic IGP Administration:** Full understanding of Link-State routing mechanics, OSPF wildcard neighbor configurations, and systematic routing table optimization.
* Enterprise Security Management:** Proficiency with multi-stage Extended ACL design, inbound switch filtering strategies, and implementing the corporate Principle of Least Privilege at the network layer.
* Systematic Troubleshooting:** Skillful deployment of network diagnostics (`show ip route`, `show ip ospf neighbor`, `ping`, `traceroute`) to perform deep packet path analysis, map topology faults, and execute precision remediation.
```
