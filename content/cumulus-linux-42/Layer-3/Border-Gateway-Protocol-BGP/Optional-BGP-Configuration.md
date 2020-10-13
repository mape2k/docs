---
title: Optional BGP Configuration
author: Cumulus Networks
weight: 814
toc: 3
---
This section describes optional configuration. The steps provided in this section assume that you already configured basic BGP as described in {{<link url="Basic-BGP-Configuration" >}}.

## Peer Groups

Instead of specifying properties of each individual peer, you can define one or more peer groups and associate all the attributes common to that peer session to a peer group. A peer needs to be attached to a peer group only once, when it then inherits all address families activated for that peer group.

{{%notice note%}}

If the peer you want to add to a group already exists in the BGP configuration, delete it first, than add it to the peer group.

{{%/notice%}}

The following example commands create a peer group called SPINE that includes two external peers.

{{< tabs "34 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor SPINE peer-group
cumulus@leaf01:~$ net add bgp neighbor SPINE remote-as external
cumulus@leaf01:~$ net add bgp neighbor 169.254.10.101 peer-group SPINE
cumulus@leaf01:~$ net add bgp neighbor 169.254.10.102 peer-group SPINE
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# neighbor SPINE peer-group
leaf01(config-router)# neighbor SPINE remote-as external
leaf01(config-router)# neighbor 169.254.10.101 peer-group SPINE
leaf01(config-router)# neighbor 169.254.10.102 peer-group SPINE
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$

```

{{< /tab >}}

{{< /tabs >}}

For an unnumbered configuration, you can use a single command to configure a neighbor and attach it to a peer group.

{{< tabs "16 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor swp51 interface peer-group SPINE
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
leaf01(config-router)# neighbor swp51 interface peer-group SPINE
```

{{< /tab >}}

{{< /tabs >}}

## BGP Dynamic Neighbors

*BGP dynamic neighbor* provides BGP peering to a group of remote neighbors within a specified range of IPv4 or IPv6 addresses for a BGP peer group. You can configure each range as a subnet IP address.

You configure dynamic neighbors using the `bgp listen range <ip-address> peer-group <group>` command. After you configure the dynamic neighbors, a BGP speaker can listen for, and form peer relationships with, any neighbor in the IP address range and mapped to a peer group.

The following example commands create the peer group SPINE and configure BGP peering to remote neighbors within the address range 169.254.10.101/24.

{{< tabs "36 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor SPINE peer-group
cumulus@leaf01:~$ net add bgp neighbor SPINE remote-as external
cumulus@leaf01:~$ net add bgp listen range 169.254.10.101/24 peer-group SPINE
cumulus@leaf01:~$ net add bgp listen limit 5
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

The `net add bgp listen limit` command limits the number of dynamic peers. The default value is *100*.

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# bgp listen range 169.254.10.101/24 peer-group SPINE
leaf01(config-router)# bgp listen limit 5
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

The `bgp listen limit` command limits the number of dynamic peers. The default value is *100*.

{{< /tab >}}

{{< /tabs >}}

These commands produce an IPv4 configuration that looks like this:

```
router bgp 65101
  neighbor SPINE peer-group
  neighbor SPINE remote-as external
  bgp listen limit 5
  bgp listen range 169.254.10.101/24 peer-group SPINE
```

## eBGP Multihop

The eBGP multihop option lets you use BGP to exchange routes with an external peer that is more than one hop away.

To establish a connection between two eBGP peers that are not directly connected:

{{< tabs "42 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor 10.10.10.101 remote-as external
cumulus@leaf01:~$ net add bgp neighbor 10.10.10.101 ebgp-multihop
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# neighbor 10.10.10.101 remote-as external
leaf01(config-router)# neighbor 10.10.10.101 ebgp-multihop
leaf01(config)# exit
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}

{{< /tabs >}}

## MD5-enabled BGP Neighbors

You can authenticate your BGP peer connection to prevent interference with your routing tables.

To enable MD5 authentication for BGP peers, set the same password on the each peer.

The following example commands set the password *mypassword* on BGP peers leaf01 and spine01:

{{< tabs "40 ">}}

{{< tab "NCLU Commands ">}}

{{< tabs "1273 ">}}

{{< tab "leaf01 ">}}

```
cumulus@leaf01:~$ net add bgp neighbor swp51 password mypassword
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}

{{< tab "spine01 ">}}

```
cumulus@spine01:~$ net add bgp neighbor swp1 password mypassword
cumulus@spine01:~$ net pending
cumulus@spine01:~$ net commit
```

{{< /tab >}}

{{< /tabs >}}

{{< /tab >}}

{{< tab "vtysh Commands ">}}

{{< tabs "1295 ">}}

{{< tab "leaf01 ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# neighbor swp51 password mypassword
leaf01(config-router)# exit
leaf01(config)# exit
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}

{{< tab "spine01 ">}}

```
cumulus@spine01:~$ sudo vtysh

spine01# configure terminal
spine01(config)# router bgp 65199
spine01(config-router)# neighbor swp1 password mypassword
spine01(config-router)# end
spine01# write memory
spine01# exit
cumulus@spine01:~$
```

{{< /tab >}}

{{< /tabs >}}

{{< /tab >}}

{{< /tabs >}}

You can confirm the configuration with the NCLU `net show bgp neighbor <neighbor>` command or the with the vtysh `show ip bgp neighbor <neighbor>` command.

{{< expand "net show bgp neighbor <neighbor> example" >}}

The following example shows that a session with the peer is established and that authentication is enabled (the output shows `Peer Authentication Enabled` towards the end):

```
cumulus@spine01:~$ net show bgp neighbor swp1
BGP neighbor on swp1: fe80::2294:15ff:fe02:7bbf, remote AS 65101, local AS 65199, external link
Hostname: leaf01
  BGP version 4, remote router ID 10.10.10.1, local router ID 10.10.10.101
  BGP state = Established, up for 00:00:39
  Last read 00:00:00, Last write 00:00:00
  Hold time is 9, keepalive interval is 3 seconds
  Neighbor capabilities:
    4 Byte AS: advertised and received
    AddPath:
      IPv4 Unicast: RX advertised IPv4 Unicast and received
    Route refresh: advertised and received(old & new)
    Address Family IPv4 Unicast: advertised and received
    Hostname Capability: advertised (name: spine01,domain name: n/a) received (name: leaf01,domain name: n/a)
    Graceful Restart Capability: advertised and received
      Remote Restart timer is 120 seconds
      Address families by peer:
        none
  Graceful restart information:
    End-of-RIB send: IPv4 Unicast
    End-of-RIB received: IPv4 Unicast
  Message statistics:
    Inq depth is 0
    Outq depth is 0
                         Sent       Rcvd
    Opens:                  2          2
    Notifications:          0          2
    Updates:              424        369
    Keepalives:           633        633
    Route Refresh:          0          0
    Capability:             0          0
    Total:               1059       1006
  Minimum time between advertisement runs is 0 seconds
  For address family: IPv4 Unicast
  Update group 1, subgroup 1
  Packet Queue length 0
  Community attribute sent to this neighbor(all)
  3 accepted prefixes
  Connections established 2; dropped 1
  Last reset 00:02:37,   Notification received (Cease/Other Configuration Change)
Local host: fe80::7c41:fff:fe93:b711, Local port: 45586
Foreign host: fe80::2294:15ff:fe02:7bbf, Foreign port: 179
Nexthop: 10.10.10.101
Nexthop global: fe80::7c41:fff:fe93:b711
Nexthop local: fe80::7c41:fff:fe93:b711
BGP connection: shared network
BGP Connect Retry Timer in Seconds: 10
Peer Authentication Enabled
Read thread: on  Write thread: on  FD used: 27
```

{{< /expand >}}

{{%notice note%}}

The MD5 password configured against a BGP listen-range peer group (used to accept and create dynamic BGP neighbors) is not enforced. This means that connections are accepted from peers that do not specify a password.

{{%/notice%}}

## Remove Private ASNs

If you use private ASNs in the datacenter, any routes you send out to the internet contain these private ASNs. You can remove all the private ASNs from routes to a specific neighbor.

The following example command removes private ASNs from routes sent to the neighbor on swp51 (an unnumbered interface):

```
cumulus@switch:~$ net add bgp neighbor swp51 remove-private-AS
```

You can replace the private ASNs with your public ASN with the following command:

```
cumulus@switch:~$ net add bgp neighbor swp51 remove-private-AS replace-AS
```

## ECMP

BGP supports equal-cost multipathing (ECMP). If a BGP node hears a certain prefix from multiple peers, it has all the information necessary to program the routing table and forward traffic for that prefix through all of these peers. BGP typically choses one best path for each prefix and installs that route in the forwarding table.

In Cumulus Linux, the *BGP multipath* option is enabled by default with the maximum number of paths set to 64 so that the switch can install multiple equal-cost BGP paths to the forwarding table and load balance traffic across multiple links.

To change the number of paths allowed, run the following commands. The example commands change the maximum number of paths to 120. You can set a value between 1 and 256. 1 disables the BGP multipath option.

{{< tabs "297 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp maximum-paths 120
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65101
switch(config-router)# address-family ipv4
switch(config-router-af)# maximum-paths 120
switch(config-router-af)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and vtysh commands save the configuration in the `address-family` stanza of the `/etc/frr/frr.conf` file. For example:

```
...
!
address-family ipv4 unicast
 network 10.1.10.0/24
 network 10.10.10.1/32
 maximum-paths 120
exit-address-family
...
```

When *BGP multipath* is enabled, only BGP routes from the same AS are load balanced. If the routes go across several different AS neighbors, even if the AS path length is same, they are not load balanced. To be able to load balance between multiple paths received from different AS neighbors, you need to set the *bestpath as-path multipath-relax* option.

{{< tabs "12 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp bestpath as-path multipath-relax
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65101
switch(config-router)# bgp bestpath as-path multipath-relax
switch(config-router)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and vtysh commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
  bgp router-id 10.0.0.1
  bgp bestpath as-path multipath-relax
...
```

{{%notice note%}}

When you disable the *bestpath as-path multipath-relax* option, EVPN type-5 routes do not use the updated configuration. Type-5 routes continue to use all available ECMP paths in the underlay fabric, regardless of ASN.

{{%/notice%}}

## Advertise IPv4 Prefixes with IPv6 Next Hops

{{<exlink url="https://tools.ietf.org/html/rfc5549" text="RFC 5549">}} defines the method used for BGP to advertise IPv4 prefixes with IPv6 next hops. The RFC does not make a distinction between whether the IPv6 peering and next hop values should be global unicast addresses (GUA) or link-local addresses. Cumulus Linux supports advertising IPv4 prefixes with IPv6 global unicast and link-local next hop addresses, with either *unnumbered* or *numbered* BGP.

When BGP peering uses IPv6 global addresses and IPv4 prefixes are being advertised and installed, IPv6 route advertisements are used to derive the MAC address of the peer so that FRR can create an IPv4 route with a link-local IPv4 next hop address (defined by RFC 3927). This is required to install the route into the kernel. These route advertisement settings are configured automatically when FRR receives an update from a BGP peer using IPv6 global addresses that contain an IPv4 prefix with an IPv6 next hop, and the enhanced-next hop capability has been negotiated.

To enable advertisement of IPv4 prefixes with IPv6 next hops over global IPv6 peerings, add the `extended-nexthop` capability to the global IPv6 neighbor statements on each end of the BGP sessions.

{{< tabs "18 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor 2001:db8:0002::0a00:0002 capability extended-nexthop
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65101
switch(config-router)# neighbor 2001:db8:0002::0a00:0002 capability extended-nexthop
switch(config-router)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and vtysh commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
  ...
  neighbor 2001:db8:0002::0a00:0002 capability extended-nexthop
...
```

Ensure that the IPv6 peers are activated under the IPv4 unicast address family; otherwise, all peers are activated in the IPv4 unicast address family by default. If `no bgp default ipv4-unicast` is configured, you need to explicitly activate the IPv6 neighbor under the IPv4 unicast address family as shown below:

{{< tabs "20 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor 2001:db8:0002::0a00:0002 capability extended-nexthop
cumulus@switch:~$ net add bgp ipv4 unicast neighbor 2001:db8:0002::0a00:0002 activate
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65101
switch(config-router)# neighbor 2001:db8:0002::0a00:0002 capability extended-nexthop
switch(config-router)# address-family ipv4 unicast
switch(config-router-af)# neighbor 2001:db8:0002::0a00:0002 activate
switch(config-router-af)# end
switch(config)# exit
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and vtysh commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
router-id 10.10.10.1
no bgp default ipv4-unicast
neighbor 2001:db8:0002::0a00:0002 remote-as external
neighbor 2001:db8:0002::0a00:0002 capability extended-nexthop
!
address-family ipv4 unicast
  neighbor 2001:db8:0002::0a00:0002 activate
exit-address-family
...
```

## Neighbor Maximum Prefixes

To configure the maximum number of route announcements (prefixes) that can be received from a BGP neighbor, run the vtysh `neighbor <peer> maximum-prefix <value>` command. For example:

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65001
leaf01(config-router)# neighbor swp51 maximum-prefix 3000
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

## Aggregate Addresses

To minimize the size of the routing table and save bandwidth, you can aggregate a range of networks in your routing table into a single prefix.

The following example command aggregates a range of addresses, such as 10.1.1.0/24, 10.1.2.0/24, 10.1.3.0/24 into the single prefix 10.1.0.0/16.

```
cumulus@switch:~$ net add bgp aggregate-address 10.1.0.0/16
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

The `summary-only` option ensures that longer-prefixes inside the aggregate address are suppressed before sending BGP updates:

```
cumulus@switch:~$ net add bgp aggregate-address 10.1.0.0/16 summary-only
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

## BGP Timers

BGP includes several timers that you can configure.

### BGP TTL Security Hop Count

You can use the TTL security hop count option to prevent attacks against eBGP, such as denial of service (DoS) attacks.
By default, BGP messages are sent to eBGP neighbors with an IP time-to-live (TTL) of 1, which requires the peer to be directly connected, otherwise, the packets expire along the way. (You can adjust the TTL with {{<link url="#ebgp-multihop" text="eBGP multihop">}} option.) An attacker can easily adjust the TTL of packets so that they appear to originating from a peer that is directly connected.

The BGP TTL security hops option inverts the direction in which the TTL is counted. Instead of accepting only packets with a TTL set to 1, only BGP messages with a TTL greater than or equal to 255 minus the specified hop count are accepted.

When TTL security is in use, eBGP multihop is no longer needed.

The following command example sets the TTL security hop count value to 200:

{{< tabs "44 ">}}
{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor swp51 ttl-security hops 200
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}
{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# neighbor swp51 ttl-security hops 200
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}
{{< /tabs >}}

The NCLU and `vtysh` commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
  ...
  neighbor swp51 ttl-security hops 200
...
```

{{%notice note%}}

- When you configure `ttl-security hops` on a peer group instead of a specific neighbor, FRR does not add it to either the running configuration or to the `/etc/frr/frr.conf` file. To work around this issue, add `ttl-security hops` to individual neighbors instead of the peer group.
- Enabling `ttl-security hops` does not program the hardware with relevant information. Frames are forwarded to the CPU and are dropped. Cumulus Networks recommends that you use the `net add acl` command to explicitly add the relevant entry to hardware. For example, configure a file, such as `/etc/cumulus/acl/policy.d/01control_plane_bgp.rules`, with a rule similar to the following:

   ```
   INGRESS_INTF = swp1
       INGRESS_CHAIN = INPUT, FORWARD

       [iptables]
       -A $INGRESS_CHAIN --in-interface $INGRESS_INTF -p tcp --dport bgp -m ttl --ttl 255 POLICE --set-mode pkt --set-rate 2000 --set-burst 1000
   -A $INGRESS_CHAIN --in-interface $INGRESS_INTF -p tcp --dport bgp DROP
   ```

   For more information about ACLs, see {{<link title="Netfilter - ACLs">}}.

{{%/notice%}}

### Keepalive Interval

If a link is up but the neighboring BGP process hangs or crashes, the FRRouting `watchfrr` daemon, which monitors the various FRRouting daemons, attempts to restart it. BGP itself has a keepalive interval that is exchanged between neighbors. By default, this keepalive interval is set to 3 seconds. You can increase this interval to a higher value, which decreases CPU load, especially in the presence of a lot of neighbors. The keepalive interval is the periodicity with which the keepalive message is sent. The hold time specifies how many keepalive messages can be lost before the connection is considered invalid, and is typically set to three times the keepalive time and defaults to 9 seconds.

The following example commands set the keepalive interval to 10 seconds and the hold time to 30 seconds.

{{< tabs "64 ">}}
{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor swp51 timers 10 30
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}
{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp
leaf01(config-router)# neighbor swp51 timers 10 30
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}
{{< /tabs >}}

The NCLU and `vtysh` commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
  ...
  neighbor swp51 timers 10 30
...
```

### Reconnect Interval

By default, the BGP process attempts to connect to a peer after a failure (or on startup) every 10 seconds.

To change this value, run the following commands on each neighbor.

The following example commands set the reconnect value to 30 seconds:

{{< tabs "204 ">}}
{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor swp51 timers connect 30
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}
{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp
leaf01(config-router)# neighbor swp51 timers connect 30
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}
{{< /tabs >}}

The NCLU and vtysh commands save the configuration in the `/etc/frr/frr.conf` file. For example:
```
...
router bgp 65101
  ...
  neighbor swp51 timers connect 30
...
```

### Advertisement Interval

By default, BGP chooses stability over fast convergence, which is very useful when routing for the Internet. For example, unlike link-state protocols, BGP typically waits a number of seconds before sending consecutive updates to a neighbor. This advertisement interval ensures that an unstable neighbor flapping routes are not propagated throughout the network. By default, this interval is set to 0 seconds for both eBGP and iBGP sessions, which allows for very fast convergence. For more information about the advertisement interval, see {{<exlink url="http://tools.ietf.org/html/draft-jakma-mrai-02" text="this IETF draft">}}.

The following example commands set the advertisement interval to 5 seconds:

{{< tabs "68 ">}}
{{< tab "NCLU Commands ">}}

```
cumulus@leaf01:~$ net add bgp neighbor swp51 advertisement-interval 5
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}
{{< tab "vtysh Commands ">}}

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp
leaf01(config-router)# neighbor swp51 advertisement-interval 5
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}
{{< /tabs >}}

The NCLU and `vtysh` commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
  ...
  neighbor swp51 advertisement-interval 5
...
```

## Route Reflectors

iBGP rules state that a route learned from an iBGP peer can not be sent to another iBGP peer. In a datacenter spine and leaf network using iBGP, this prevents a spine from sending a route learned from a leaf to any other leaf. As a workaround, BGP introduced the concept of a *route reflector* that selectively violates this rule; when an iBGP speaker is configured as a route reflector, it *can* send iBGP learned routes to other iBGP peers.

In the following example, spine01 is acting as a route reflector. The leaf switches, leaf01, leaf02 and leaf03 are *route reflector clients*. Any route that spine01 learns from a route reflector client is sent to other route reflector clients.

{{< img src = "/images/cumulus-linux/bgp-route-reflectors-example.png" >}}

To configure the BGP node as a route reflector, set the `route-reflector-client` option. The following example sets spine01 shown in the illustration above to be a route reflector for leaf01:

{{< tabs "344 ">}}
{{< tab "NCLU Commands ">}}

```
cumulus@spine01:~$ net add bgp neighbor swp51 route-reflector-client
cumulus@spine01:~$ net pending
cumulus@spine01:~$ net commit
```

{{< /tab >}}
{{< tab "vtysh Commands ">}}

```
cumulus@spine01:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65199
switch(config-router)# address-family ipv4
switch(config-router-af)# neighbor swp51 route-reflector-client
switch(config-router-af)# end
switch# write memory
switch# exit
cumulus@spine01:~$
```

{{< /tab >}}
{{< /tabs >}}

The NCLU and vtysh commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65199
 bgp router-id 10.10.10.101
 neighbor swp51 remote-as external
 !
 address-family ipv4 unicast
  network 10.10.10.101/32
  neighbor swp51 route-reflector-client
 exit-address-family
...
```

{{%notice info%}}
For IPv6, you must run the `route-reflector-client` command **after** the `activate` command; otherwise, the `route-reflector-client` command is ignored.
{{%/notice%}}

## BGP add-path

Cumulus Linux supports both BGP add-path RX and BGP add-path TX.

### BGP add-path RX

*BGP add-path RX* allows BGP to receive multiple paths for the same prefix. A path identifier is used so that additional paths do not override previously advertised paths. BGP add-path RX is enabled by default; no additional configuration is required.

To view the existing capabilities, run the NCLU `net show bgp neighbor` command or the vtysh `show ip bgp neighbors` command. The existing capabilities are listed in the subsection *Add Path*, below *Neighbor capabilities.*

```
cumulus@leaf01:~$ net show bgp neighbor
BGP neighbor on swp51: fe80::7c41:fff:fe93:b711, remote AS 65199, local AS 65101, external link
Hostname: spine01
 Member of peer-group underlay for session parameters
  BGP version 4, remote router ID 10.10.10.101, local router ID 10.10.10.1
  BGP state = Established, up for 1d12h39m
  Last read 00:00:03, Last write 00:00:01
  Hold time is 9, keepalive interval is 3 seconds
  Neighbor capabilities:
    4 Byte AS: advertised and received
    AddPath:
      IPv4 Unicast: RX advertised IPv4 Unicast and received
    Extended nexthop: advertised and received
      Address families by peer:
                   IPv4 Unicast
    Route refresh: advertised and received(old & new)
    Address Family IPv4 Unicast: advertised and received
    Hostname Capability: advertised (name: leaf01,domain name: n/a) received (name: spine01,domain name: n/a)
    Graceful Restart Capability: advertised and received
...
```

The example output above shows that additional BGP paths can be sent and received (TX and RX are advertised). It also shows that the BGP neighbor, fe80::7c41:fff:fe93:b711, supports both.

To view the current additional paths, run the NCLU `net show bgp <router-id>` command or the `vtysh show ip bgp <router-id>` command.

The example output shows an additional path that has been added by the TX node for receiving. Each path has a unique AddPath ID.

```
cumulus@leaf01:mgmt:~$ net show bgp 10.10.10.9
BGP routing table entry for 10.10.10.9/32
Paths: (2 available, best #1, table Default-IP-Routing-Table)
  Advertised to non peer-group peers:
  spine01(swp51) spine02(swp52)
  65020 65012
    fe80::4638:39ff:fe00:5c from spine01(swp51) (10.10.10.12)
    (fe80::4638:39ff:fe00:5c) (used)
      Origin incomplete, localpref 100, valid, external, multipath, bestpath-from-AS 65020, best (Older Path)
      AddPath ID: RX 0, TX 6
      Last update: Wed Nov 16 22:47:00 2016
  65020 65012
    fe80::4638:39ff:fe00:2b from spine02(swp52) (10.10.10.12)
    (fe80::4638:39ff:fe00:2b) (used)
      Origin incomplete, localpref 100, valid, external, multipath
      AddPath ID: RX 0, TX 3
      Last update: Fri Oct  2 03:56:33 2020
```

### BGP add-path TX

AddPath TX allows BGP to advertise more than just the bestpath for a prefix. Consider the following topology:

```
          r8
          |
          |
  r1 ----    ---- r6
  r2 ---- r7 ---- r5
          ||
          ||
        r3 r4
```

In this topology:

- r1 and r2 are in AS 100
- r3 and r4 are in AS 300
- r5 and r6 are in AS 500
- r7 is in AS 700
- r8 is in AS 800
- r7 learns 1.1.1.1/32 from r1, r2, r3, r4, r5, and r6; r7 picks the path from r1 as the bestpath for 1.1.1.1/32

{{< tabs "28 ">}}

{{< tab "NCLU Commands ">}}

The example below configures the r7 session to advertise the bestpath learned from each AS. In this case, a path from AS 100, a path from AS 300, and a path from AS 500. The `net show bgp 1.1.1.1/32` from r7 has `bestpath-from-AS 100` so you can see what the bestpath is from each AS:

```
cumulus@r7:~$ net add bgp autonomous-system 700
cumulus@r7:~$ net add bgp neighbor 192.0.2.2 addpath-tx-bestpath-per-AS
cumulus@r7:~$ net pending
cumulus@r7:~$ net commit
```

The output below shows the result on r8:

```
cumulus@r8:~$ net show bgp 1.1.1.1/32
BGP routing table entry for 1.1.1.1/32
Paths: (3 available, best #3, table Default-IP-Routing-Table)
  Advertised to non peer-group peers:
  r7(10.7.8.1)
  700 100
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 1:1
      AddPath ID: RX 2, TX 4
      Last update: Thu Jun  2 00:57:14 2016

  700 300
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 3:3
      AddPath ID: RX 4, TX 3
      Last update: Thu Jun  2 00:57:14 2016

  700 500
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external, bestpath-from-AS 700, best (First path received)
      Community: 5:5
      AddPath ID: RX 6, TX 2
      Last update: Thu Jun  2 00:57:14 2016
```

The example below shows the results when r7 is configured to advertise all paths to r8:

```
cumulus@r7:~$ net add bgp autonomous-system 700
cumulus@r7:~$ net add bgp neighbor 192.0.2.2 addpath-tx-all-paths
cumulus@r7:~$ net pending
cumulus@r7:~$ net commit
```

The output below shows the result on r8:

```
cumulus@r8:~$ net show bgp 1.1.1.1/32
BGP routing table entry for 1.1.1.1/32
Paths: (3 available, best #3, table Default-IP-Routing-Table)
  Advertised to non peer-group peers:
  r7(10.7.8.1)
  700 100
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 1:1
      AddPath ID: RX 2, TX 4
      Last update: Thu Jun  2 00:57:14 2016

  700 300
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 3:3
      AddPath ID: RX 4, TX 3
      Last update: Thu Jun  2 00:57:14 2016

  700 500
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external, bestpath-from-AS 700, best (First path received)
      Community: 5:5
      AddPath ID: RX 6, TX 2
      Last update: Thu Jun  2 00:57:14 2016
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

The example below configures the r7 session to advertise the bestpath learned from each AS. In this case, a path from AS 100, a path from AS 300, and a path from AS 500. The `show ip bgp 1.1.1.1/32` from r7 has
`bestpath-from-AS 100` so you can see what the bestpath is from each AS:

```
cumulus@switch:~$ sudo vtysh

r7# configure terminal
r7(config)# router bgp 700
r7(config-router)# neighbor 192.0.2.2 addpath-tx-bestpath-per-AS
r7(config-router)#
```

The output below shows the result on r8:

```
cumulus@switch:~$ sudo vtysh

r8# show ip bgp 1.1.1.1/32
BGP routing table entry for 1.1.1.1/32
Paths: (3 available, best #3, table Default-IP-Routing-Table)
  Advertised to non peer-group peers:
  r7(10.7.8.1)
  700 100
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 1:1
      AddPath ID: RX 2, TX 4
      Last update: Thu Jun  2 00:57:14 2016

  700 300
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 3:3
      AddPath ID: RX 4, TX 3
      Last update: Thu Jun  2 00:57:14 2016

  700 500
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external, bestpath-from-AS 700, best
      Community: 5:5
      AddPath ID: RX 6, TX 2
      Last update: Thu Jun  2 00:57:14 2016

r8#
```

The example below shows the results when r7 is configured to advertise all paths to r8:

```
cumulus@switch:~$ sudo vtysh

r7# configure terminal
r7(config)# router bgp 700
r7(config-router)# neighbor 192.0.2.2 addpath-tx-all-paths
r7(config-router)#
```

The output below shows the result on r8:

```
cumulus@switch:~$ sudo vtysh

r8# show ip bgp 1.1.1.1/32
BGP routing table entry for 1.1.1.1/32
Paths: (3 available, best #3, table Default-IP-Routing-Table)
  Advertised to non peer-group peers:
  r7(10.7.8.1)
  700 100
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 1:1
      AddPath ID: RX 2, TX 4
      Last update: Thu Jun  2 00:57:14 2016

  700 300
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external
      Community: 3:3
      AddPath ID: RX 4, TX 3
      Last update: Thu Jun  2 00:57:14 2016

  700 500
    10.7.8.1 from r7(10.7.8.1) (10.0.0.7)
      Origin IGP, localpref 100, valid, external, bestpath-from-AS 700, best
      Community: 5:5
      AddPath ID: RX 6, TX 2
      Last update: Thu Jun  2 00:57:14 2016
```

{{< /tab >}}

{{< /tabs >}}

## Graceful BGP Shutdown

To reduce packet loss during planned maintenance of a router or link, you can configure graceful BGP shutdown, which forces traffic to route around the BGP node:

{{< tabs "Graceful BGP shutdown">}}

{{< tab "NCLU Commands ">}}

To enable graceful shutdown:

```
cumulus@leaf01:~$ net add bgp graceful-shutdown
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

To disable graceful shutdown:

```
cumulus@leaf01:~$ net del bgp graceful-shutdown
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

To enable graceful shutdown:

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# bgp graceful-shutdown
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@v:~$
```

To disable graceful shutdown:

```
cumulus@leaf01:~$ sudo vtysh

leaf01# configure terminal
leaf01(config)# router bgp 65101
leaf01(config-router)# no bgp graceful-shutdown
leaf01(config-router)# end
leaf01# write memory
leaf01# exit
cumulus@leaf01:~$
```

{{< /tab >}}

{{< /tabs >}}

When configured, the `graceful-shutdown` community is added to all paths from eBGP peers and the `local-pref` for that route is set to `0`. To see the configuration, run the NCLU `net show bgp <route>` command or the vtysh `show ip bgp <route>` command. For example:

```
cumulus@switch:~$ net show bgp 10.10.10.0/24
BGP routing table entry for 10.10.10.0/24
Paths: (2 available, best #1, table Default-IP-Routing-Table)
  Advertised to non peer-group peers:
  bottom0(10.10.10.2)
  30 20
    10.10.10.2 (metric 10) from top1(10.10.10.2) (10.10.10.2)
      Origin IGP, localpref 100, valid, internal, bestpath-from-AS 30, best
      Community: 99:1
      AddPath ID: RX 0, TX 52
      Last update: Mon Sep 18 17:01:18 2017

  20
    10.10.10.3 from bottom0(10.10.10.32) (10.10.10.10)
      Origin IGP, metric 0, localpref 0, valid, external, bestpath-from-AS 20
      Community: 99:1 graceful-shutdown
      AddPath ID: RX 0, TX 2
      Last update: Mon Sep 18 17:01:18 2017
```

## Enable Read-only Mode

As BGP peers are established and updates are received, prefixes might be installed in the RIB and advertised to BGP peers even though the information from all peers is not yet received and processed. Depending on the timing of the updates, prefixes might be installed and propagated through BGP, and then immediately withdrawn and replaced with new routing information. Read-only mode minimizes this BGP route churn in both the local RIB and with BGP peers.

Enable read-only mode to reduce CPU and network usage when restarting the BGP process. Because intermediate best paths are possible for the same prefix as peers get established and start receiving updates at different times, read-only mode is particularly useful in topologies where BGP learns a prefix from many peers and the network has a high number of prefixes.

The following example commands enable read-only mode, sets the `max-delay` timer to 300 seconds and the `establish-wait` timer to 90 seconds.

The default value for max-delay is 0, which disables read-only mode. The update delay and establish wait can be any value between 0 and 3600 seconds. The `establish-wait` setting is optional; however, if specified, it must be shorter than the `max-delay`.

{{< tabs "48 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp update-delay 300 90
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# update-delay 300 90
switch(config-router)# end
switch# write memory
switch# switch
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

Read-only mode begins as soon as the first peer reaches its established state and the `max-delay` timer starts, and continues until either of the following two conditions are met:

- All the configured peers (except the shutdown peers) have sent an explicit EOR (End-Of-RIB) or an implicit EOR. The first keep-alive after BGP reaches the established state is considered an implicit EOR.  If you specify the `establish-wait` option, BGP only considers peers that have reached the established state from the moment the `max-delay` timer starts until the `establish-wait` period ends.

  The minimum set of established peers for which EOR is expected are the peers that are established during the `establish-wait window,` not necessarily all the configured neighbors.

- The timer reaches the configured `max-delay`.

While in read-only mode, BGP does not run best-path or generate any updates to its peers.

To show information about the state of the update delay, run the NCLU `net show bgp summary` command or the vtysh `show ip bgp summary` command.

## Route Maps for Route Updates

You can apply {{<exlink url="http://docs.frrouting.org/en/latest/routemap.html" text="route maps">}} in BGP in one of two ways:

- Filter routes from BGP into Zebra
- Filter routes from Zebra into the Linux kernel

{{%notice info%}}
In NCLU, you can only set the community number in a route map. You cannot set other community options such as `no-export`, `no-advertise`, or `additive`. This is a known limitation in `network-docopt`, which NCLU uses to parse commands.
{{%/notice%}}

### Filter Routes from BGP into Zebra

You can apply a route map on route updates from BGP to Zebra. All the applicable match operations are allowed, such as match on prefix, next hop, communities, and so on. Set operations for this attach-point are limited to metric and next hop only. Any operation of this feature does not affect the BGP internal RIB.

Both IPv4 and IPv6 address families are supported. Route maps work on multi-paths; however, the metric setting is based on the best path only.

The following example command applies a route map called routemap1 to filter route updates from BGP into Zebra:

{{< tabs "50 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp table-map routemap1
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# table-map routemap1
switch(config-router)# end
switch# write memory
switch# switch
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

### Filter Routes from Zebra into the Linux Kernel

The following example commands apply a route map called routemap1 to filter route updates from Zebra into the Linux kernel:

{{< tabs "52 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add routing protocol bgp route-map routemap1
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# bgp route-map routemap1
switch(config-router)# end
switch# write memory
switch# switch
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

## BGP Community Lists

You can use *{{<exlink url="http://docs.frrouting.org/en/latest/bgp.html#community-lists" text="community lists">}}* to define a BGP community to tag one or more routes. You can then use the communities to apply a route policy on either egress or ingress.

The BGP community list can be either *standard* or *expanded.* The standard BGP community list is a pair of values (such as *100:100*) that can be tagged on a specific prefix and advertised to other neighbors or applied on route ingress. Or, it can be one of four BGP default communities:

- *internet*: a BGP community that matches all routes
- *local-AS*: a BGP community that restricts routes to your confederation's sub-AS
- *no-advertise*: a BGP community that is not advertised to anyone
- *no-export*: a BGP community that is not advertised to the eBGP peer

An expanded BGP community list takes a regular expression of communities and matches the listed communities.

When the neighbor receives the prefix, it examines the community value and takes action accordingly, such as permitting or denying the community member in the routing policy.

Here is an example of a standard community list filter:

{{< tabs "54 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add routing community-list standard COMMUNITY1 permit 100:100
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# bgp community-list standard COMMUNITY1 permit 100:100
switch(config)# exit
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

You can apply the community list to a route map to define the routing policy:

{{< tabs "1127">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp table-map ROUTE-MAP1
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65101
switch(config-router)# table-map ROUTE-MAP1
switch(config-router)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

## Related Information

- {{<exlink url="https://tools.ietf.org/html/rfc4360" text="RFC 4360, BGP Extended Communities Attribute">}}
- {{<exlink url="https://tools.ietf.org/html/rfc4456" text="RFC 4456, BGP Route Reflection - An Alternative to Full Mesh Internal BGP (iBGP)">}}
