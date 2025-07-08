## Intro
Devices in the routing layer are linked using an address that most of the time binds them to their location. For them, moving means asking for/getting a new IP address, which means that all active connections must be shut down and set up again: a service can't know the new address until the two of them communicate again.
### Location strategies
- **Location Update**: the device updates the communicating peer and the infrastructure about the new IP it received.
	- Statically: This can happen under static conditions like moving from one area to another
	- Dynamically: This can happen under various conditions
		- Periodically
		- When the signal gets too weak
		- When the device moves
- **Location search**: The infrastructure locates the device in the space when a handoff occurs. The infrastructure is also responsible for giving a new address to the device
	- **Blanket polling**: The infrastructure locates the device in the space by polling the APs under its control. This increases the latency of the communication.
	- **Expanding ring search**: Starting from the old position, the infrastructure searches for the device in an expanding pattern
	- **Sequential paging**: Using some heuristics, first the infrastructure searches for the device in the highest probability location, then moves on to the second highest and so on.
____________
## MIP
Mobile IP is an infrastructure-based way to overcome the mobility problem without changing the IP protocol while still being backwards compatible. The infrastructure adds two nodes to keep track of movements while keeping the correspondents unaware of the mobility:

- **Home Agent**: It's a local node in the home network. It acts as a gateway: it receives packets on behalf of the node at the known IP address, then forwards the messages to the Mobile Host at the new location using *IP-within-IP*.
- **Foreign Agent**: It's the gateway closest to the mobile device and leases it a new address ***(COA - care of address)*** to receive packets from the HA. The FA can give its own IP (*FA COA*) or use a DHCP server. When using a *FA COA*, the FA keeps a table with incoming IP and MAC addresses to forward packets directly to the MH, without needing to use routing at layer 3.
- **Mobile Host**: when it moves, it looks for a foreign agent to get a new IP. The IP can usually be the same as the FA or a valid local network IP. The MH signals the new IP to the HA, which can now forward messages for the MH to the FA. The MH can communicate with the CH without going through the HA by applying its former IP directly to be consistent with the connection. While moving, some packets can possibly be missed and lost; this can be hidden using some kind of retransmission at upper layers.

This architecture can be defined as **triangular routing**.
### Problems: Firewalls
- Exiting Filter: A firewall can block packets with source addresses not in the local network range.
- Entering Filter: If the correspondent node is in the same local network as the MH, the firewall can block local IPs (MH former IP) coming from outside.

Since the HA → FA → MH path can already create latency overhead, adding MH → HA → CH could completely kill performance.
### MIPv6
To overcome these problems and eliminate the overheads, IPv6 introduces a new design.

The MH, once it obtains a new address (COA) from the FA, updates the HA (as in the previous protocol). When the correspondent node contacts the HA with a packet for the MH, the HA can send back a *binding update* with the current IP address of the MH. The correspondent node and the MH can then communicate directly using the tunneling technique, also used in IPsec.

This fixes the problems and overhead of **triangular routing**, but it makes all nodes aware of MIPv6 and requires all nodes to implement it properly.
#### Hierarchical MIPv6
With this extension, the infrastructure can set up a hierarchy of FAs to decrease the number of binding updates and route packets locally for the MH without involving the HA in case of handoff between several local FAs ***(Local subnet handoff)***. The upper node in the FA hierarchy is also known as Mobile Anchor Point **MAP**. The binding update is still needed if the node moves to a different MAP region ***(MAP Domain handoff)***.

Using statistics, such as the Service and Mobility Rate (**SMR**), the FAs can autonomously reconfigure themselves to achieve the best configuration:
- Hierarchical MIPv6 if the movements are small and the traffic is steady
- MIPv6 if the movements in the network are frequent and generalized
#### Proxy MIP
In this solution, the MH nodes can be unaware of MIP; instead, they use a gateway that implements it for them. The infrastructure includes a *Local Mobility Anchor* (**LMA**) that handles the association between node IPs and their locations, i.e., which proxy they are connected to. The proxies are properly called *Mobile Access Gateways* (**MAG**); they can also be access points for WLANs. The LMA is the access point for the network and routes packets to the right gateway.
_____________
## Host Identity Protocol
This alternative solution wants to split location and identification from the IP address. The IDs are public keys and the locators are the IP addresses. To bind ID and position, a special discovery-authentication protocol has been introduced.

To introduce the HIP protocol without modifying the TCP/IP stack, a new layer is needed between them. The HIP layer includes a bridge (from IPv4 to IPv6), multi-homing (multiple network interfaces have the same public key) and mobility algorithms.

Since this protocol has to be injected into the kernel of each device, it is not actually used in the wild.
____________________________________
## I-TCP

The TCP standard doesn't take into account mobile nodes and their problems. As we have seen in the physical layer, most packet loss in mobile communication is due to fast and sudden physical effects and not from congestion like in fixed communication. TCP handles these cases all in the same way: fast decrease of the bit-rate and slow start.

I-TCP wants to solve these problems by providing an alternative that is backward compatible with TCP:

- Splits stream management and congestion management, giving each node their proper treatment
- Adds a Gateway that proxies the communications: on the inside I-TCP is used, on the outside standard TCP is used.

Why I-TCP is more suitable for mobile nodes?
- **Connection Management**: I-TCP can manage the state of the TCP connection more effectively. If the *mobile node loses connectivity, the proxy can detect* this and take appropriate actions, such as suspending the connection or attempting to re-establish it, while buffering TCP packets coming from the peers.
- **Error Recovery**: I-TCP can implement *error recovery mechanisms* that are more suited to mobile environments. For instance, it can use techniques like **selective acknowledgment (SACK)** to recover lost packets without requiring a full retransmission of the data stream.
- **Reduced Latency**: By maintaining a separate connection for the mobile node, I-TCP can reduce the latency associated with handoffs and reconnections. The proxy can quickly respond to changes in connectivity without needing to renegotiate the entire TCP connection

Peers don't need to know I-TCP, only mobile nodes in the LAN. Since it's the proxy that communicates directly with the peer using TCP, the End-To-End semantic is broken.

I-TCP is deeply integrated with MIP, so it also handles handoff and mobility across the WLAN: it's possible to manage a handoff with packet buffering without triggering retransmissions, but if one of the proxies crashes, the whole session is lost without retransmissions occurring (the End-to-End semantic is broken).

To avoid this kind of scenario, application layer support is needed, but this would make the I-TCP protocol no longer transparent.
____________
## Recap:
We have seen some common pattern of resolution:
- Fixed Anchor point
- Proxies
- Hierarchies
- Approximated solutions and optimistic algorithms
- Locality concept, trying fixing locally without global heavyweight coordination

