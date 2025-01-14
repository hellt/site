<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/hellt/drawio-js@main/embed2.js" async></script>

As always, this tutorial will be backed up by a lab that readers can effortlessly deploy on their own machine and follow along. Oper-group lab is contained within [srl-labs/oper-group-lab](https://github.com/srl-labs/opergroup-lab) repository and features:

1. A Clos based fabric with 4 leaves and 2 spines, forming the fabric
2. Two dual-homed clients emulated with linux containers and running `iperf` software to generate traffic
3. L2 EVPN service[^1] configured across the leaves of the fabric
4. A telemetry stack to demonstrate oper-group operations in-action.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:0,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

## Physical topology
On a physical layer topology interconnections are layed down as follows:
<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:5,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

Each client is dual homed to corresponding leaves; To achieve that, interfaces `eth1` and `eth2` are formed into a `bond0` interface.  
On the leaves side, access interface `Ethernet-1/1` is part of a LAG interface which is "stretched" between a pair of leaves, forming a logical construct similar to MC-LAG.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:6,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

## Fabric underlay
In the underlay of a fabric leaves and spines run eBGP protocol to enable leaves to exchange reachability information for their `system0` interfaces.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:7,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

eBGP peerings are formed between each leaf and spine pair.

## Fabric overlay
To support BGP EVPN service, in the overlay iBGP peerings with EVPN address family are established from each leaf to each spine, with spines acting as route reflectors.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:8,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

From the EVPN service standpoint, the mac-vrf instance named `vrf-1` is created on leaves and `ES-1` ethernet segment is formed from a LAG interface.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:9,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

Ethernet segments are configured to be in an all-active mode to make sure that every access link is utilized in the fabric.

## Telemetry stack
We have enhanced the lab with a telemetry stack featuring [gnmic](https://gnmic.kmrd.dev), prometheus, and grafana - our famous GPG stack. Nothing beats real-time visualization, especially when we want to correlate events happening in the network.

| Element    | Address                |
| ---------- | ---------------------- |
| Grafana    | https://localhost:3000 |
| Prometheus | https://localhost:9090 |


## Lab deployment
Start with cloning lab's repository

```
git clone https://github.com/srl-labs/opergroup-lab.git && cd opergroup-lab
```

Lab repository contains startup configuration files for the fabric nodes, as well as necessary files for the telemetry stack to come up online operational. To deploy the lab:

```
containerlab deploy -t opergroup.clab.yml
```

This will stand up a lab with an already pre-configured fabric using startup configs contained within [`configs`](https://github.com/srl-labs/opergroup-lab/tree/main/configs) directory.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;margin:0 auto; display:block;" data-mxgraph="{&quot;page&quot;:10,&quot;zoom&quot;:3,&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;check-visible-state&quot;:true,&quot;resize&quot;:true,&quot;url&quot;:&quot;https://raw.githubusercontent.com/srl-labs/learn-srlinux/diagrams/opergroup.drawio&quot;}"></div>

The deployed lab starts up in a pre-provisioned step, where underlay/overlay configuration has already been done. We proceed with oper-group use case exploration in the next chapter of this tutorial.

[^1]: Check [L2 EVPN tutorial](../../../l2evpn/intro.md) to get the basics of L2 EVPN service configuration.