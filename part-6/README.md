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
