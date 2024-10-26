# Part 1 - The Underlay

In this section, we are going to build the underlay network. As seen from the home page, the 10.10.10.0/24 is the prefix that we are going to use for the loopbacks. 

## Features required

We are going to use OSPF as the underlay, thus the feature has to be turn on using the command `feature ospf`

```
spine01# show feature | i ospf | i enabled
ospf                   1          enabled(not-running)
ospf                   2          enabled(not-running)
ospf                   3          enabled(not-running)
ospf                   4          enabled(not-running)
ospf                   5          enabled(not-running)
ospf                   6          enabled(not-running)
ospf                   7          enabled(not-running)
ospf                   8          enabled(not-running)
ospf                   9          enabled(not-running)
ospf                   10         enabled(not-running)
ospf                   11         enabled(not-running)
ospf                   12         enabled(not-running)
ospf                   13         enabled(not-running)
ospf                   14         enabled(not-running)
ospf                   15         enabled(not-running)
ospf                   16         enabled(not-running)
```

## Bring up OSPF and Loopback reachability

Once the feature turned on, let's proceed to enable OSPF and the loopback, `R` represent router node ID:

Spines:

```
router ospf 65100
  router-id 10.10.10.R
  maximum-paths 2
!
interface ethernet 1/1-3
  ip router ospf 65100 area 0
  ip ospf network point-to-point
!
interface loopback10
  ip address 10.10.10.R/32
  ip ospf network point-to-point
  ip router ospf 65100 area 0.0.0.0
```
Leaf's:

```
router ospf 65100
  router-id 10.10.10.R
  maximum-paths 2

interface ethernet 1/1-2
  ip router ospf 65100 area 0
  ip ospf network point-to-point 

interface loopback10
  ip address 10.10.10.R/32
  ip ospf network point-to-point
  ip router ospf 65100 area 0.0.0.0
```

Let's validate neighborship and loopback reachability on both spine switches:

Spine01

```
spine01# show ip ospf neighbors 
 OSPF Process ID 65100 VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.10.10.3        1 FULL/ -          00:02:00 10.1.3.3        Eth1/1 
 10.10.10.4        1 FULL/ -          00:01:48 10.1.4.4        Eth1/2 
 10.10.10.5        1 FULL/ -          00:01:34 10.1.5.5        Eth1/3 
spine01# ping 10.10.10.3 source-interface loopback 10 c 1
PING 10.10.10.3 (10.10.10.3): 56 data bytes
64 bytes from 10.10.10.3: icmp_seq=0 ttl=254 time=10.847 ms

--- 10.10.10.3 ping statistics ---
1 packets transmitted, 1 packets received, 0.00% packet loss
round-trip min/avg/max = 10.847/10.847/10.847 ms
spine01# ping 10.10.10.4 source-interface loopback 10 c 1
PING 10.10.10.4 (10.10.10.4): 56 data bytes
64 bytes from 10.10.10.4: icmp_seq=0 ttl=254 time=30.631 ms

--- 10.10.10.4 ping statistics ---
1 packets transmitted, 1 packets received, 0.00% packet loss
round-trip min/avg/max = 30.631/30.631/30.631 ms
spine01# ping 10.10.10.5 source-interface loopback 10 c 1
PING 10.10.10.5 (10.10.10.5): 56 data bytes
64 bytes from 10.10.10.5: icmp_seq=0 ttl=254 time=17.639 ms

--- 10.10.10.5 ping statistics ---
1 packets transmitted, 1 packets received, 0.00% packet loss
round-trip min/avg/max = 17.639/17.639/17.639 ms
```

Spine02
```
spine02# show ip ospf neighbors 
 OSPF Process ID 65100 VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.10.10.3        1 FULL/ -          00:03:22 10.2.3.3        Eth1/1 
 10.10.10.4        1 FULL/ -          00:03:04 10.2.4.4        Eth1/2 
 10.10.10.5        1 FULL/ -          00:02:48 10.2.5.5        Eth1/3 
spine02# ping 10.10.10.3 source-interface loopback 10 c 1
PING 10.10.10.3 (10.10.10.3): 56 data bytes
64 bytes from 10.10.10.3: icmp_seq=0 ttl=254 time=12.567 ms

--- 10.10.10.3 ping statistics ---
1 packets transmitted, 1 packets received, 0.00% packet loss
round-trip min/avg/max = 12.567/12.567/12.567 ms
spine02# ping 10.10.10.4 source-interface loopback 10 c 1
PING 10.10.10.4 (10.10.10.4): 56 data bytes
64 bytes from 10.10.10.4: icmp_seq=0 ttl=254 time=15.752 ms

--- 10.10.10.4 ping statistics ---
1 packets transmitted, 1 packets received, 0.00% packet loss
round-trip min/avg/max = 15.752/15.752/15.752 ms
spine02# ping 10.10.10.5 source-interface loopback 10 c 1
PING 10.10.10.5 (10.10.10.5): 56 data bytes
64 bytes from 10.10.10.5: icmp_seq=0 ttl=254 time=11.36 ms

--- 10.10.10.5 ping statistics ---
1 packets transmitted, 1 packets received, 0.00% packet loss
round-trip min/avg/max = 11.36/11.36/11.36 ms
```

That's conclude part-1, let's proceed with part-2.


