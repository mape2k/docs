---
title: Best Practices
author: Cumulus Networks
weight: 816
toc: 3
---
Cumulus Networks recommends you follow these best practices when configuring MLAG on your switches.

## Converge Quickly On Link Failures

In the Clos topology, Cumulus Networks recommends that you only use interface addresses to set up peering sessions. This means that when the link fails, the BGP session is torn down immediately, triggering route updates to propagate through the network quickly. This requires the `link-detect` and `ttl-security hops` commands to be enabled for all links. The `ttl-security hops` command specifies how many hops away the neighbor is. For example, in a Clos topology, every peer is at most 1 hop away.

To set the `ttl-security hops`:

{{< tabs "62 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor swp51 ttl-security hops 1
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# neighbor swp51 ttl-security hops 1
switch(config-router)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and `vtysh` commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65101
  ...
  neighbor swp51 ttl-security hops 1
...
```

## Converge Quickly On Soft Failures

It is possible that the link is up but the neighboring BGP process is hung or has crashed. In this case, the FRRouting `watchfrr` daemon, which monitors the various FRRouting daemons, attempts to restart it. BGP itself has a keepalive interval that is exchanged between neighbors. By default, this keepalive interval is set to 3 seconds. You can increase this interval to a higher value, which decreases CPU load, especially in the presence of a lot of neighbors. The keepalive interval is the periodicity with which the keepalive message is sent. The hold time specifies how many keepalive messages can be lost before the connection is considered invalid. It is typically set to three times the keepalive time and defaults to 9 seconds. The following examples commands change the keepalive interval to 10 seconds and the hold time to 30 seconds.

{{< tabs "64 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor swp51 timers 10 30
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# neighbor swp51 timers 10 30
switch(config-router)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and `vtysh` commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65000
  ...
  neighbor swp51 timers 10 30
...
```

## Reconnect Quickly After a Failure

By default, the BGP process attempts to connect to a peer after a failure (or on startup) every 10 seconds. To change this value, run the following commands on each neighbor.

{{< tabs "204 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor swp51 timers connect 30
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# neighbor swp51 timers connect 30
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
  neighbor swp51 timers connect 30
...
```

## Peer Groups

Instead of specifying properties of each individual peer, you can define one or more peer groups and associate all the attributes common to that peer session to a peer group. A peer needs to be attached to a peer group only once, when it then inherits all address families activated for that peer group.

After you attach a peer to a peer group, you need to associate an IP address with the peer group. The following example shows how to define and use peer groups:

{{< tabs "34 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor tier-2 peer-group
cumulus@switch:~$ net add bgp neighbor tier-2 next-hop-self
cumulus@switch:~$ net add bgp neighbor swp51 peer-group tier-2
cumulus@switch:~$ net add bgp neighbor swp51 peer-group tier-2
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp 65101
switch(config-router)# neighbor tier-2 peer-group
switch(config-router)# address-family ipv4 unicast
switch(config-router-af)# neighbor tier-2 next-hop-self
switch(config-router-af)# exit
switch(config-router)# neighbor swp51 peer-group tier-2
switch(config-router)# neighbor swp51 peer-group tier-2
```

{{< /tab >}}

{{< /tabs >}}

{{%notice note%}}

BGP peer-group restrictions have been replaced with update-groups, which dynamically examine all peers and group them if they have the same outbound policy.

{{%/notice%}}

For an unnumbered configuration, you can use a single command to configure a neighbor and attach it to a peer group.

{{< tabs "16 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor swp51 interface peer-group tier-2
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
switch(config-router)# neighbor swp51 interface peer-group tier-2
```

{{< /tab >}}

{{< /tabs >}}

## BGP Advertisement

As a best practice, limit the exchange of routing information at various parts in the network. The following image illustrates one way you can do so in a typical Clos architecture:

{{< img src = "/images/cumulus-linux/bgp-advertisement-best-practices.png" >}}

## Multiple Routing Tables and Forwarding

You can run multiple routing tables (one for in-band/data plane traffic and one for out-of-band management plane traffic) on the same switch using {{<link url="Management-VRF" text="management VRF">}} (multiple routing tables and forwarding).

{{%notice note%}}

BGP and static routing (IPv4 and IPv6) are supported within a VRF context. For more information, refer to {{<link title="Virtual Routing and Forwarding - VRF">}}.

{{%/notice%}}

## Advertisement Interval

By default, BGP chooses stability over fast convergence, which is very useful when routing for the Internet. For example, unlike link-state protocols, BGP typically waits a number of seconds before sending consecutive updates to a neighbor. This advertisement interval ensures that an unstable neighbor flapping routes are not propagated throughout the network. By default, this interval is set to 0 seconds for both eBGP and iBGP sessions, which allows for very fast convergence. For more information about the advertisement interval, see {{<exlink url="http://tools.ietf.org/html/draft-jakma-mrai-02" text="this IETF draft">}}.

To modify the advertisement interval, run the following commands:

{{< tabs "68 ">}}

{{< tab "NCLU Commands ">}}

```
cumulus@switch:~$ net add bgp neighbor swp51 advertisement-interval 5
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

{{< /tab >}}

{{< tab "vtysh Commands ">}}

```
cumulus@switch:~$ sudo vtysh

switch# configure terminal
switch(config)# router bgp
switch(config-router)# neighbor swp51 advertisement-interval 5
switch(config-router)# end
switch# write memory
switch# exit
cumulus@switch:~$
```

{{< /tab >}}

{{< /tabs >}}

The NCLU and `vtysh` commands save the configuration in the `/etc/frr/frr.conf` file. For example:

```
...
router bgp 65000
  ...
  neighbor swp51 advertisement-interval 5
...
```

To show the keepalive interval, hold time, and advertisement interval, you can run the NCLU `net show bgp neighbor <peer>` command or the vtysh `show ip bgp neighbor <peer>` command. For example:

```
cumulus@switch:~$ net show bgp neighbor swp51
BGP neighbor on swp51: fe80::4638:39ff:fe00:5c, remote AS 65020, local AS 65011, external link
Hostname: spine01
  Member of peer-group fabric for session parameters
  BGP version 4, remote router ID 0.0.0.0
  BGP state = Connect
  Last read 00:04:37, Last write 00:44:07
  Hold time is 30, keepalive interval is 10 seconds
  Configured hold time is 30, keepalive interval is 10 seconds
  Message statistics:
    Inq depth is 0
    Outq depth is 0
                          Sent       Rcvd
    Opens:                  1          1
    Notifications:          1          0
    Updates:                7          6
    Keepalives:          2374       2373
    Route Refresh:          0          0
    Capability:             0          0
    Total:               2383       2380
  Minimum time between advertisement runs is 5 seconds
...
```
