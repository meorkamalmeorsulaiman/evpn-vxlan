# EVPN-VxLAN

This will the learning notes for EVPN VxLAN

## Lab Setup

The lab setup will consist of 2 spines and 3 leaf switches that stretch two layer-2 domains - 100 & 200. Below are the topology:

![Topology](https://github.com/meorkamalmeorsulaiman/evpn-vxlan/blob/main/images/topology.jpg)

Loopbacks and MAC details

| Hostname | loopback       | Mac Address    |
|----------|----------------|----------------|
| spine01  | 10.10.10.1     | n/a            |
| spine02  | 10.10.10.2     | n/a            |
| leaf03   | 10.10.10.3     | n/a            |
| leaf04   | 10.10.10.4     | n/a            |
| leaf05   | 10.10.10.5     | n/a            |
| host06   | 192.168.100.6  | aabb.cc00.6000 |
| host07   | 192.168.100.7  | aabb.cc00.7000 |
| host08   | 192.168.100.8  | aabb.cc00.8000 |
| host09   | 192.168.200.9  | aabb.cc00.9000 |
| host10   | 192.168.200.10 | aabb.cc00.a000 |

## Section

We device the lab into separated section:
- Part-1: Building the underlay
- Part-2: Working on p2p VxLAN tunnels
- Part-3: Forwarding BUM traffic
- Part-4: Turning on BGP-EVPN on VxLAN Fabric
- Part-5: Inter-Subnet forwarding
