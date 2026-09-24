# Deploy Multi-Site Fabric with NDFC

This section is an example use-case of deploying VxLAN-EVPN fabric and migration it from legacy network. This lab section is a walkthru steps of deploying VxLAN-EVPN fabric using NDFC. Several walk-thru will be devided as below:
- Part 1-5 - Create a Multi-site fabric
- Part 5-7 - Integrating legacy network with the fabric
- Part 8 - Establish external connectivity
- Part 9 - Migrating the gateway from legacy to the fabric

## Lab Setup

Each segment in the lab were devided into multiple AS:
- AS64521 - DC01
- AS64522 - DC02
- AS64531 - ISN01
- AS64512 - Legacy
- AS64541 - ISP01
- AS64542 - ISP02

The lab will build multi-site fabric with 2 seperated DC running their own AS number. One layer 3 network running AS64531 will providing connectivity between DC01 and DC02. The legacy network created as simple network to simulate migration steps and 2 ISPs are to simulate external connectivity.

![Lab Topo](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/ndfc-9/images/ndfc/0-lab-topo.png)