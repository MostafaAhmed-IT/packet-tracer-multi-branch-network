# Multi-Branch Network Design — Cisco Packet Tracer


A complete four-branch company network designed and configured in Cisco Packet Tracer as the final project of a hands-on CCNA course.


## Topology overview

- Four branch sites (R1, R2, R3 and R4) around one central ISP router, joined by serial WAN links running PPP with CHAP authentication

- Each branch has 3 switches and 8 PCs — two PCs in each of its four VLANs, sixteen VLANs across the whole project

- Departments separated into VLANs, grouped by branch: R1 has VLANs 10, 20, 30 and 40; R2 has 50, 60, 70 and 80; R3 has 90, 100, 110 and 120; R4 has 130, 140, 150 and 160

- Inter-VLAN routing by router-on-a-stick, per-VLAN DHCP on each branch router, OSPF between the routers, and ACLs controlling traffic between the branches

- One central server (Server0, 192.168.1.1) on R1's own LAN holds the TFTP backup of every router's configuration, receives R1's syslog logging, and is the web server the ACL rules protect; the project canvas also assigns it NTP for R1


## Addressing

| Branch | VLANs | LAN networks | Gateway in each VLAN | DHCP pools | WAN link (branch ↔ ISP) |
|---|---|---|---|---|---|
| R1 | 10, 20, 30, 40 | 192.168.10.0/24 – 192.168.40.0/24 | .100 | v10 – v40 | 201.10.10.200 ↔ 201.10.10.100 |
| R2 | 50, 60, 70, 80 | 192.168.50.0/24 – 192.168.80.0/24 | .100 | v50 – v80 | 201.20.20.200 ↔ 201.20.20.100 |
| R3 | 90, 100, 110, 120 | 192.168.90.0/24 – 192.168.120.0/24 | .100 | v90 – v120 | 201.30.30.200 ↔ 201.30.30.100 |
| R4 | 130, 140, 150, 160 | 192.168.130.0/24 – 192.168.160.0/24 | .100 | v130 – v160 | 201.40.40.200 ↔ 201.40.40.100 |

- Every LAN follows one pattern: network 192.168.\<VLAN\>.0/24, gateway .100 on the branch router's subinterface, addresses handed out by DHCP starting from .1

- Server network: 192.168.1.0/24 on R1's physical interface — Server0 is 192.168.1.1 with gateway 192.168.1.100

- On every WAN link the ISP end is .100 and the branch end is .200; the ISP supplies the serial clock (clock rate 64000)


## Key configurations

- VLANs and 802.1Q trunking, with VTP (domain scal, password 1234) creating each branch's four VLANs once on a server switch and the other switches learning them as clients

- EtherChannel bundles (LACP) between the switches of each branch

- Spanning Tree (PVST+) with the root bridge split between each branch's switches — primary for one half of the VLANs, secondary for the other half

- Router-on-a-stick inter-VLAN routing, one subinterface per VLAN, the router as gateway (.100) in every VLAN

- DHCP pools per VLAN on the branch routers (network and default-router; no DNS server is set in this project)

- PPP with CHAP authentication on the serial WAN links — each branch holds username ISP password 1234, the ISP demands CHAP and supplies the clock

- OSPF (process 1, area 0) between all five routers, plus a default route on every router pointing at its neighbour

- Named extended ACLs enforcing the project's traffic rules: SCAL on R1 (PC23 denied HTTP to the server, PC30 denied ping to the server, no ping from VLAN 70 to VLAN 10) and scal on R2 (no ping from VLANs 130 and 140 to VLAN 80), each closed by permit ip any any

- TFTP backup of every router's running configuration to Server0 (copy run tftp), and syslog logging from R1 to the same server

- Step-by-step verification with ping between VLANs and branches, and show commands: show ip route, show ip ospf neighbor, show access-lists, show vtp status, show etherchannel summary


## Configurations

- [R1](configs/R1.txt) — branch 1 router: subinterfaces for VLANs 10–40, the server network, DHCP pools v10–v40, PPP/CHAP serial, OSPF, default route, syslog, and the SCAL ACL

- [R2](configs/R2.txt) — branch 2 router: subinterfaces for VLANs 50–80, DHCP pools v50–v80, PPP/CHAP serial, OSPF, default route, and the scal ACL

- [R3](configs/R3.txt) — branch 3 router: subinterfaces for VLANs 90–120, DHCP pools v90–v120, PPP/CHAP serial, OSPF and default route

- [R4](configs/R4.txt) — branch 4 router: subinterfaces for VLANs 130–160, DHCP pools v130–v160, PPP/CHAP serial, OSPF and default route

- [ISP](configs/ISP.txt) — the central router: four serial links with PPP, CHAP and clock rate 64000, OSPF, and one default route toward each branch

- [Switch template](configs/switch-template.txt) — the per-branch switch pattern (VTP server and clients, trunks, EtherChannel, PVST+ root split, access ports) with the branch's four VLANs as placeholders


## Tools

- Cisco Packet Tracer


The .pkt project file will be added so anyone can download and explore the topology.
