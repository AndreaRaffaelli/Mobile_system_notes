Naming systems and discovery systems for mobile systems must help the network to:
- **Self-configure** to use available services or even get a proper IP
- Service Discovery: what services are available in the area
- Resource Delivery with authentication and authorization, provides drivers for the services and manages synchronization
## Jini / Apache River
A discovery system that provides full Service Proxy objects to be used by clients. The clients don't need to have any information beforehand.
- Services sign in to the Discovery server, providing their **Java Service Proxy object** and the **service attributes**.
- Clients do a lookup on the discovery service and get back a fully configured Proxy object that doesn't need any stub or skeleton.
The Discovery service can form a distributed environment with other service brokers, for example by signing in as a service in each other's registry. Adding more Service brokers can improve reliability and redundancy, which can benefit the entire network; however, the coordination can be harder.
The coordination between the clients is done by the Service broker through **lease time**. In the time window, a device can **use mutually exclusively the Proxy** object to reach the service. Another way would have been for the broker to update each server proxy to inform about the status of a service, but in this way the proxy can't handle the sudden disconnection of the service.

## Service Location Protocol
Proposed by the **IETF** didn't have a wide diffusion.
The clients communicate with the **User Agent** that takes care of every other communication. The **service Agent** registers the service available and the **Directory agent** caches the advertisement messages and answer faster to the queries of the user agents. It manages also the service agents, this makes it a local repository for all the services. 
The services sign up with URL that defines them uniquely.
Rich and flexible purpose, but hard to configure.

## UPnP universal plug in protocol
Standard from Microsoft to autonomously configurate a network without the need of an infrastructure. When a device enters a local network first of all asks for an IP, if a DHCP server in not available gets randomly an IP from the dedicated range `169.254.0.0/16` (**Auto IP**). Since the IP is random, of course the device asks via ARP request is anyone has already that address. When the IP is correctly assigned the the protocol must link service providers and clients. This is done using the **Simple Service Discovery Protocol SSDP**: this decentralized protocol specify a broadcast IP address and a port to send advertisement and discovery messages, `UDP <239.255.255.250, 1900>`. The simple discovery protocol use 2 messages `ssdp:alive` and `ssdp:discovery`. The requests run on top HTTP and UDP: multicast for requests, unicast for answers. 
When a **control point** (a client) asks for a service broadcasts a request the service answers with a `HTTP NOTIFY` message, it contains the URL where it is possible to find the `DDF` device description file, a XML file with the description of the device and the provided service. More details on the service and interfaces are written in the `SDF`, service description file. It contains all the methods of a service and values contained in the **state service table**. 
The communication between the device and client is based on `SOAP` protocol. A **control point** can subscribe to a device and it notifies it whenever a state value changes (uses the GENA protocol on HTTP) and receives back a `SID` subscriber id.
Two UPnP networks can be bridged, but in this case if there are two devices with same IP it is important to keep a forwarding table with the location of each device.
#### Message Flow:
- `ssdp:discovery` Multicast Search **M-SEARCH** from control point. Control $\rightarrow$ device  
- `HTTP Notify` URL of `DDF`  Control $\leftarrow$ Device
- `HTTP GET` the `DDF` Control $\rightarrow$ device
- `HTTP Response` Control $\leftarrow$ Device
- `HTTP subscribe` Control $\rightarrow$ device
- `HTTP Response` with `SID` Control $\leftarrow$ Device
- `HTTP POST` soap, sending commands Control $\rightarrow$ device
- `HTTP` SOAP response Control $\leftarrow$ Device
- `Notify GENA` with status changes Control $\leftarrow$ Device
#### Summarizing:
- Service discovery: peer to peer environments without infrastructure
- Adaptability: 
	- IP are dynamically allocated
	- Event based interaction
	- No support for routing
- Transparent support to communication
	- Internet protocols
	- No support for multi-hop ad-hoc communications
- Data Transformations
	- from standard XML to proprietary control point specific
