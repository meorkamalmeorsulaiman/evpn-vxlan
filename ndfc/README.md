# Deploy Multi-Site Fabric with NDFC

This section is an example use-case of deploying VxLAN-EVPN fabric and migration it from legacy network. This lab section is a walkthru steps of deploying VxLAN-EVPN fabric using NDFC. Several walk-thru will be devided as below:
- Part 1-5 - Create a Multi-site fabric
- Part 5-7 - Integrating legacy network with the fabric
- Part 8 - Establish external connectivity
- Part 9 - Migrating the gateway from legacy to the fabric

## Lab Setup

The lab will build multi-site fabric with 2 seperate data-center. Each DC running different AS number. A layer 3 network as inter-side network will providing connectivity between two fabrics. A simple traditional network and connected to new fabric. This will help to simulate the migration steps. 2 external connectivity will be use to validate connection toward outside of the fabric. Each segment in the lab were devided into multiple AS:
- AS64521 - DC01
- AS64522 - DC02
- AS64531 - ISN01
- AS64512 - Legacy
- AS64541 - ISP01
- AS64542 - ISP02

![Lab Topo](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-9/images/ndfc/0-lab-topo.png)

## BUM Traffic - ARP Request

Below illustrate the forwarding within the fabric where ARP Request generated. 

![Lab ARP Req](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-9/images/ndfc/0-lab-arp-req.png)

Disecting above diagram, where H18 is trying to resolve H11 IP address:
1. An ARP request send by H18 to the leaf03
2. leaf03 will learn the H18 mac address and add into it's mac table. Since the VLAN is part of the VNI, it will sent an update too via BGP to it's peer. Finally, the switch will encapsulate the ARP request and flood it in the entire fabric with the following details:
- Packet is VxLAN encapsulated with VLAN2300 VNI
- The source IP would be the Anycast VTEP as leaf03 and leaf04 are VPC pair
- The destination IP would be the multicast group configured under the VTEP
3. Packet will duplicated to several destination based on the spine01 OIL which is bgw07 and leaf05
4. Once the VxLAN packet arrived, leaf05 will decapsulate based on the VNI and forward toward H11