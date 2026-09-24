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

## Forwarding

Below illustrate the forwarding within the fabric where ARP Request generated. 

![Lab ARP Req](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-9/images/ndfc/0-lab-arp-req.png)