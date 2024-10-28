# Flooding Behaviour

The current fabric running with ingress replication, in an event that the ingress leaf received an arp for `VLAN 200`. The arp will be flooded throughout the fabric. `leaf05` will receive the arp request coming from `host06`.

## ARP Suppression

A feature that allow or change the flooding behaviour by suppressing the ARP request at the ingress leaf. The ingress switch will response to the arp request coming for the host. At the moment you will see that there is no arp cache on the local switch:
```
leaf03# show ip arp suppression-cache detail 

Flags: + - Adjacencies synced via CFSoE
       L - Local Adjacency
       R - Remote Adjacency
       L2 - Learnt over L2 interface
       PS - Added via L2RIB, Peer Sync
       RO - Dervied from L2RIB Peer Sync Entry

Ip Address      Age      Mac Address    Vlan Physical-ifindex    Flags    Remote Vtep Addrs
```

We have to configure TCAM region for this to work.
```
leaf03(config-if-nve-vni)# suppress-arp 
ERROR: Please configure TCAM region for Ingress ARP-Ether ACL before configuring ARP supression.
```
There is no space allocated:
```
leaf03# show hardware access-list tcam region | i Eth
               Ingress ARP-Ether ACL [arp-ether] size =    0 
```

We can take from `RACL`
```
leaf03# show hardware access-list tcam region | i RACL
                                IPV4 RACL [racl] size = 1536 
                           IPV6 RACL [ipv6-racl] size =    0 
                       Egress IPV4 RACL [e-racl] size =  768 
                  Egress IPV6 RACL [e-ipv6-racl] size =    0 
                 Ingress RACL v4 & v6 [racl-all] size =    0 
leaf03# conf t
Enter configuration commands, one per line. End with CNTL/Z.
leaf03(config)# hardware access-list tcam region racl 512
Warning: Please save config and reload the system for the configuration to take  effect
```
You need to reload the switch
```
leaf03# show hardware access-list tcam region | i racl
                                IPV4 RACL [racl] size =  512 
                           IPV6 RACL [ipv6-racl] size =    0 
                       Egress IPV4 RACL [e-racl] size =  768 
                  Egress IPV6 RACL [e-ipv6-racl] size =    0 
                 Ingress RACL v4 & v6 [racl-all] size =    0
```

We can see that `RACL` space has been reduce, let's allocate to Ingress ARP-Ether ACL and reload the switch
```
leaf03(config)# hardware access-list tcam region arp-ether 256 double-wide 
Warning: Please save config and reload the system for the configuration to take  effect
leaf03(config)# 2024 Oct 28 03:33:03 leaf03 %$ VDC-1 %$ pltfm_config[8001]: WARNING: Configuring  the arp-ether region without "double-wide" is deprecated and can result in silent non-vxlan packet drops. Use the "double-wide" keyword when carving TCAM space for the arp-ether region.
```

Not the tcam space allocated:
```
leaf03# show hardware access-list tcam region | i eth
               Ingress ARP-Ether ACL [arp-ether] size =  256 (double-wide)
```

Let's turn on the arp suppresion on `VLAN100`
```
interface nve1
  member vni 2000
    suppress-arp
```

## Let's Ping

Let ping the gateway from `host09` and `host10`
```
host10#ping 192.168.200.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.200.254, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 2/3/6 ms
```
host10
```
host09#ping 192.168.200.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.200.254, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/3/7 ms
```

Now you can see the arp suppression cache on `leaf03`
```
leaf03# show ip arp suppression-cache detail | be Address
Ip Address      Age      Mac Address    Vlan Physical-ifindex    Flags    Remote Vtep Addrs

192.168.200.9   00:01:09 aabb.cc00.9000  200 Ethernet1/6         L
192.168.200.10  00:01:09 aabb.cc00.a000  200 (null)              R        10.10.10.4  
leaf03# 
```

## Digest

When turning on the ARP snooping for a particular `L2VNI`, the ingress switch will then response to the ARP request on behalf of the remote endpoint. This will reduce the flooding within the fabric as it is note necessary. This should be validate from packet capture, let's debug on `host09`. Before that we make sure the arp isn't cached on `host09`
```
host09#show ip arp                
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.200.9           -   aabb.cc00.9000  ARPA   Ethernet0/0
Internet  192.168.200.254         0   0001.0001.0001  ARPA   Ethernet0/0
host09#debug arp
ARP packet debugging is on
```

Let's trigger an arp request on `host09` while turning on debug arp on both hosts
host09
```
host09#clear ip arp 192.168.200.10
host09#
*Oct 28 04:07:43.810: IP ARP: sent req src 192.168.200.9 aabb.cc00.9000, dst 192.168.200.10 aabb.cc00.a000 Ethernet0/0
*Oct 28 04:07:43.813: IP ARP: rcvd rep src 192.168.200.10 aabb.cc00.a000, dst 192.168.200.9 Ethernet0/0
*Oct 28 04:07:43.813: IP ARP: creating entry for IP address: 192.168.200.10, hw: aabb.cc00.a000
```

host10
```
host10#debug arp 
ARP packet debugging is on
host10#   
host10#
host10#!!!No arp received
host10#
```
Let test on `VLAN100`
host06
```
host06#terminal width 512
host06#show ip arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.100.6           -   aabb.cc00.6000  ARPA   Ethernet0/0
Internet  192.168.100.254        16   0001.0001.0001  ARPA   Ethernet0/0
```
Let's trigger arp from `host06` to `host07`
host06
```
host06#ping 192.168.100.7 r 2
Type escape sequence to abort.
Sending 2, 100-byte ICMP Echos to 192.168.100.7, timeout is 2 seconds:
*Oct 28 04:10:26.592: IP ARP: creating incomplete entry for IP address: 192.168.100.7 interface Ethernet0/0
*Oct 28 04:10:26.592: IP ARP: sent req src 192.168.100.6 aabb.cc00.6000, dst 192.168.100.7 0000.0000.0000 Ethernet0/0
*Oct 28 04:10:26.639: IP ARP: rcvd rep src 192.168.100.7 aabb.cc00.7000, dst 192.168.100.6 Ethernet0/0.!
Success rate is 50 percent (1/2), round-trip min/avg/max = 26/26/26 ms
```

host07
```
host07#
*Oct 28 04:10:26.616: IP ARP: rcvd req src 192.168.100.6 aabb.cc00.6000, dst 192.168.100.7 Ethernet0/0
*Oct 28 04:10:26.616: IP ARP: creating entry for IP address: 192.168.100.6, hw: aabb.cc00.6000
*Oct 28 04:10:26.616: IP ARP: sent rep src 192.168.100.7 aabb.cc00.7000, dst 192.168.100.6 aabb.cc00.6000 Ethernet0/0
host07#
```

As we can see, the arp did reached `host07` on `leaf04`. That's the mechanics of ARP suppression. The feature configured on ingress leaf.
