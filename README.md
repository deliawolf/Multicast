# Multicast

Multicast

Multicast is a method of routing data on a network where data is sent from one or more points to a group of destination computers simultaneously. It is often used for streaming media and gaming where many users need to receive the same data at the same time.

Pros:

Efficient bandwidth usage: Multicast sends a single copy of data to multiple recipients, reducing the amount of network bandwidth used.
Lower server load: Since the server only needs to send out a single data stream, it reduces the load on the server.
Cons:

Not suitable for one-to-one communication: Multicast is not designed for scenarios where data needs to be sent to a single recipient.
Network configuration: Network devices (like routers and switches) must be correctly configured for multicast. Not all devices support multicast, and incorrect configurations can lead to data not reaching all intended recipients.

## IGMP (Internet Group Management Protocol)

IGMP is a communications protocol used by hosts and adjacent routers on IPv4 networks to establish multicast group memberships. Essentially, it allows a computer to report its multicast group membership to any neighboring routers.

### IGMP operation:

When a device joins a multicast group, it sends an IGMP membership report to its local router.
When it leaves the group, it sends an IGMP leave group message.
The routers periodically send IGMP queries to discover which hosts belong to which multicast groups.
Multicast Distribution Tree

A multicast distribution tree is a hierarchy of routers that spans a multicast group. It forms the path that multicast packets travel from the source to the receivers.

There are two types:

Source tree: The tree is rooted at the source, and has branches forming a shortest path tree to the receivers.
Shared tree: All sources use the same tree, which is rooted at a designated router known as the Rendezvous Point (RP).

## Multicast Routing and Rendezvous Point (RP)

Multicast routing protocols are used to construct the distribution trees. Some of the common protocols include Protocol Independent Multicast (PIM), Distance Vector Multicast Routing Protocol (DVMRP), and Multicast OSPF (MOSPF).

The Rendezvous Point (RP) is a key element in multicast routing. It acts as a shared root for multicast distribution trees and a meeting place for multicast receivers and senders. In a network, any device can be configured to be an RP.

Case Study: Consider a live sporting event being streamed over the internet. The broadcaster's server is the source, and the viewers are the receivers. Using multicast, the server only needs to send out one data stream, which is then delivered to all the viewers simultaneously, saving bandwidth. IGMP is used by the viewers' devices to report their membership in the multicast group to their local routers. The routers use a multicast routing protocol to construct a distribution tree, rooted at the RP, which determines the path the data packets take from the server to the viewers.

### Protocol Independent Multicast (PIM)

PIM is a family of multicast routing protocols that can use any unicast routing protocol's existing unicast routing table to perform Reverse Path Forwarding (RPF) checks and create multicast distribution trees. PIM operates in two modes: Dense Mode (PIM-DM) and Sparse Mode (PIM-SM).

#### PIM Dense Mode (PIM-DM)

PIM-DM assumes that all hosts want to receive multicast traffic unless they explicitly state otherwise. When a source starts sending multicast traffic, PIM-DM routers begin by flooding this traffic out of all interfaces except the one from which the traffic was received (this process is called "pruning"). If a router has no downstream neighbors interested in this traffic, it will send a prune message upstream, effectively cutting itself off from the multicast tree.

Pros: Simple to configure and good for small networks. Cons: It can waste bandwidth initially by flooding traffic out to all interfaces.

#### PIM Sparse Mode (PIM-SM)

PIM-SM is more efficient for larger networks. It assumes that no hosts want to receive multicast traffic unless they explicitly state so. Multicast traffic is initially sent to a Rendezvous Point (RP). Hosts that want to receive this traffic send join messages towards the RP. Once the multicast traffic flow is established, the routers can switch to a source-specific multicast tree, bypassing the RP, if it is more efficient.

Pros: It's more bandwidth-efficient and scales better for larger networks. Cons: It's more complex to configure because you need to configure Rendezvous Points.

#### PIM Sparse-Dense Mode

This mode allows an interface to operate in both sparse and dense modes simultaneously. If a group has an RP (configured or learned through Auto-RP or BSR), it will operate in sparse mode; if not, it operates in dense mode.

A fundamental part of PIM operation, in any mode, is the creation of multicast distribution trees, which dictate the path that multicast traffic takes from the source to the receivers.

In both modes, the multicast traffic routes are dynamically determined based on the network's state and the locations of the receivers.

#### Explanantion about static or dynamic
The part that can be either static or dynamic is the selection of the Rendezvous Point (RP) in PIM-SM. The RP can be manually assigned (static) or elected dynamically using mechanisms like Auto-RP or Bootstrap Router (BSR).

## Config Example

let's assume we have three routers in a row: Router1 (source), Router2 (RP), and Router3 (receiver). They're connected through their GigabitEthernet0/0 and GigabitEthernet0/1 interfaces.

Static RP Configuration

Router1 (Source):
```
Router1> enable
Router1# configure terminal
Router1(config)# interface GigabitEthernet0/0
Router1(config-if)# ip pim sparse-mode
Router1(config-if)# exit
Router1(config)# end
```
Router2 (RP):
```
Router2> enable
Router2# configure terminal
Router2(config)# ip multicast-routing
Router2(config)# interface GigabitEthernet0/0
Router2(config-if)# ip pim sparse-mode
Router2(config-if)# exit
Router2(config)# interface GigabitEthernet0/1
Router2(config-if)# ip pim sparse-mode
Router2(config-if)# exit
Router2(config)# ip pim rp-address 10.1.1.2
Router2(config)# end
```
Router3 (Receiver):
```
Router3> enable
Router3# configure terminal
Router3(config)# interface GigabitEthernet0/0
Router3(config-if)# ip pim sparse-mode
Router3(config-if)# ip igmp join-group 239.0.0.1
Router3(config-if)# exit
Router3(config)# end
```
Dynamic RP Configuration using BSR

Router1 (Source):
```
Router1> enable
Router1# configure terminal
Router1(config)# interface GigabitEthernet0/0
Router1(config-if)# ip pim sparse-mode
Router1(config-if)# exit
Router1(config)# end
```
Router2 (RP and BSR):
```
Router2> enable
Router2# configure terminal
Router2(config)# ip multicast-routing
Router2(config)# interface GigabitEthernet0/0
Router2(config-if)# ip pim sparse-mode
Router2(config-if)# ip pim rp-candidate
Router2(config-if)# ip pim bsr-candidate
Router2(config-if)# exit
Router2(config)# interface GigabitEthernet0/1
Router2(config-if)# ip pim sparse-mode
Router2(config-if)# ip pim rp-candidate
Router2(config-if)# ip pim bsr-candidate
Router2(config-if)# exit
Router2(config)# end
```
Router3 (Receiver):
```
Router3> enable
Router3# configure terminal
Router3(config)# interface GigabitEthernet0/0
Router3(config-if)# ip pim sparse-mode
Router3(config-if)# ip igmp join-group 239.0.0.1
Router3(config-if)# exit
Router3(config)# end
```
In this configuration, Router2 is the RP and BSR. It announces itself as a candidate RP and BSR on both interfaces. If your network setup is different, you would need to adjust the configuration accordingly.

Auto-RP Configuration

Router1 (Source):
```
Router1> enable
Router1# configure terminal
Router1(config)# interface GigabitEthernet0/0
Router1(config-if)# ip pim sparse-mode
Router1(config-if)# exit
Router1(config)# end
```
Router2 (RP and Auto-RP Mapping Agent):
```
Router2> enable
Router2# configure terminal
Router2(config)# ip multicast-routing
Router2(config)# access-list 1 permit 224.0.0.0 15.255.255.255
Router2(config)# ip pim rp-announcement-filter rp-list 1
Router2(config)# interface GigabitEthernet0/0
Router2(config-if)# ip pim sparse-mode
Router2(config-if)# exit
Router2(config)# interface GigabitEthernet0/1
Router2(config-if)# ip pim sparse-mode
Router2(config-if)# exit
Router2(config)# ip pim send-rp-announce GigabitEthernet0/0 scope 16
Router2(config)# ip pim send-rp-discovery GigabitEthernet0/0 scope 16
Router2(config)# end
```
Router3 (Receiver):
```
Router3> enable
Router3# configure terminal
Router3(config)# interface GigabitEthernet0/0
Router3(config-if)# ip pim sparse-mode
Router3(config-if)# ip igmp join-group 239.0.0.1
Router3(config-if)# exit
Router3(config)# end
```
In this configuration, Router2 is both the RP and the Auto-RP mapping agent. It announces itself as the RP for all multicast groups in the range 224.0.0.0 to 239.255.255.255. The mapping agent then sends these RP announcements to all other routers. If your network setup is different, you would need to adjust the configuration accordingly.
