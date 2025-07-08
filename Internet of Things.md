## Introduction

Networks of "things": the things are daily life objects that are now equipped with hardware that allows them to connect to the internet. IoT is applied to several use cases:

- Wireless sensor networks
- Near Field Communications (NFC)
- Body area networks (BAN / PAN)
- Machine to Machine communications

To use low energy sensors, it's necessary to extend Internet protocols to new wireless networks with different constraints.

#### SCADA

Old protocol to manage and control industrial objects remotely. Centralized protocol with little human intervention:

- Widely tested solutions
- Real-time and reliable
- Technical people friendly

but:

- Costly
- Expandable but within the same plant
- No standards
- No interoperability outside the production floor

#### IoT

IoT should introduce a new approach to overcome the drawbacks of the past using the Internet. The things can be:

- Physical: _cyber-physical system_, actuators
- Digital: Binary things, like applications, but not standardized.

IoT systems are heterogeneous and geographically distributed; they must be identified and managed uniquely no matter their place and state. Well-designed IoT systems provide a self-configuring network, where things have both physical and virtual representations.

Improvements:

- Reduction of costs and hardware dimensions
- Wireless communication
- Web-based, open standards
- Standards bring interoperability
- Horizontal solutions for general purpose
- ML to deduce knowledge

Horizontal layer to manage heterogeneity and vertical layer to build domain-specific applications. The horizontal layer can be recycled in several different projects. Usually these projects come with a domain-specific GUI.

## Cloud-based IoT

Cloud-based IoT architectures are made of:

- IoT sensors and actuators
- Gateways close to the IoT
- Remote applications on the cloud
- Analytics: ML software to deduce new knowledge
- Communication protocols: to exchange raw bytes from IoT to Gateway
- Data exchange protocols: usually web-based from Gateway to cloud.

### Gateway:

Protocol translation from IoT chunks to Cloud. It can do pre-processing:

- Buffering
- Efficiency
- Aggregation
- Filtering
- Security

Without a gateway, IoT devices would have to communicate directly to the cloud. They usually don't have the capability to speak such protocols and don't have enough CPU power to support cryptography.

### Cloud Computing

Computers managed to provide reliable access to resources "as a service":

- Fast answers to requests
- Reliable services
- Virtual infrastructures
- Scalable growth

There are various models available:

- IaaS
- PaaS
- SaaS

But also emerging: FaaS, BaaS, MaaS. Cloud providers also give APIs to manage resources autonomously.

Cloud can be built locally or in remote locations:

- On-premise
- Communal
- Public
- Hybrid

### Edge Computing

In the first generation of IoT computing, the gateways were used only to access the cloud; the devices always had to communicate with the cloud to get information. This introduced latency and wasn't scalable since the cloud received a lot of requests.

Now the gateways are Edge nodes, which means they have business logic loaded and can answer directly to the IoT devices. The Edge updates and gets updated from the cloud less frequently.

The Edges that operate on the data are also called _Fog data services_, and the cloud runs ML models to fetch information or trends. The model can be trained on the cloud.

## IoT Devices

IoT devices are usually constrained devices; they have little resources available. IoT networks are made of nodes that communicate periodically or when strictly needed. For these reasons, we talk about _constrained networks_:

- Asymmetric links
- Low throughput
- Devices not reachable
- Packet loss
- Penalization for heavy packets

#### Class 0 Devices

Very constrained devices that don't support standard Internet protocols and need a gateway to talk with the cloud. They usually come preconfigured and can't be configured again. They are constrained in RAM, ROM, and battery.

#### Class 1 Devices

They support some lightweight Internet protocols designed for IoT devices, such as CoAP over UDP. They can benefit from a Gateway but can also talk directly to the Cloud, since they are equipped with IP. They are still constrained on RAM and battery but can support security functions.

#### Class 2 Devices

They can mostly support every standard Internet protocol and can communicate directly with the cloud. They are still constrained, so they can benefit from lightweight protocols.

### Battery Limitations

|Name|Type of Limitation|Example Power Source|
|---|---|---|
|E0|Event energy limited|Event-based harvesting|
|E1|Period energy limited|Battery periodically changed/replaced|
|E2|Lifetime energy limited|Non-replaceable primary battery|
|E9|No energy limitations|Mains-powered|

#### Some IoT Devices

- Telos B: Sensor Networks. **Class 1**. CPU 8 MHz, 10KB ROM, 1 MB EEPROM, IEEE 802.15.4 radio antenna.
- Zolertia Re-Mote: Extendible hardware, **Class 2**, double radio interface for long-distance communication, 32 MHz Cortex
- Single Board Computer: Beyond Class 2, can host complete OS and can work like Edge Node.

Three-phase evolution:

- Phase 1: Smart labels to identify things uniquely, QR codes, RFID.
- Phase 2: Autonomous discovery to transmit collected data, periodically or on request. Read-only, battery-powered.
- Phase 3: Smart devices, modern and complex IoT applications. Sensors and actuators, gateway and edge nodes.

## Wireless Communication Protocols

Differs in area covered, battery consumption, bandwidth, security, and costs.

- **Bluetooth** and **BLE**: Radio protocols for PAN
- **WiFi**: Covers a small area (100m) but has a large bandwidth, is secure, uses more energy than Bluetooth.
- **LTE**: Scalable, 2km, consumes energy. Supports video streaming.
- **LoRaWAN**: WAN at low speed, long-distance covering, open standard. Typical range is 5-10km, small bandwidth, 290 bps to 52 Kbps. Supports symmetric security AES. Use case: isolated and private areas with sensors that rarely communicate.
- **NB-IoT** and **LTE Cat-M**: Cellular channel to communicate long range with bandwidth.
- **SigFox**: Proprietary protocol with small energy consumption, wide range (24km).
- **Satellite**: Extra wide coverage costs.

#### Data Exchange Protocol

Protocol to exchange data between gateway and servers. Types of communication:

- Push: Gateway decides autonomously to send messages to the server.
- Pull: Servers ask the gateway to send data.
- Request/Response: REST/HTTP, CoAP
- Publish/Subscribe: MQTT and AMQP

The choice depends on the domain.

### Request/Response

The client initiates the communication to the server:

- **Push**: Sensors send values to the gateway
- **Pull**: Actuators ask for configurations from the server

#### REST

Not a protocol but an architecture, request/response and stateless since it's based on HTTP. The clients must initiate a new connection from scratch at each request. For each resource, a URI is assigned. With the HTTP methods, the representation of the resources is transferred. To each HTTP method, a CRUD method is assigned.

- GET: Read a resource through the ID
- PUT: Create a new resource
- POST: Update a resource through the ID
- DELETE: Delete a resource through the ID

The responses are serialized in JSON that has taken the place of XML.

#### Constrained Application Protocol

Request/Response protocol M2M/IoT for constrained devices. It is REST compliant but without the overhead of HTTP. Supports the same methods as HTTP and same fields:

- URI
- Content Type
- Security by Datagram TLS, DTLS

But:

- Runs on UDP
- Can be asynchronous
- Can be Publish/Subscribe

Since the protocol is really simple, it can run on top of **6LoWPAN**, a version of IPv6 for constrained devices (works well with ZigBee). CoAP can have different levels of reliability:

- Best Effort: Messages CON (asks for an ack) and ACK
- May be: Messages NON (don't ask for ack)

### Publish Subscribe

The communication is temporally and spatially decoupled by adding a third node in the communication. The broker abstracts the sender and the receiver. This behavior can have different semantics, but in general the broker routes the messages on:

- Topic of the message
- Type of the message
- Header
- Flags and filters
- Priority
- Content

This system is centralized and allows basic primitives such as subscribe, unsubscribe, and publish.

#### MQTT

From IBM to reach EAI, MQTT is an open standard for constrained devices. Runs on **TCP**.

- Topic-based
- Low header overhead (2 bytes)
- Easy primitives: `connect`, `publish`, `subscribe`, `disconnect`
- **Retained Messages**: messages stored for new devices
- **Will**: messages if a client disconnects suddenly.
- 3 Semantics: **at least once**, **at most once**, **exactly once**

> To achieve exactly once, MQTT uses an exchange of 4 messages, a unique ID for every packet, and a DUP flag: `PUBLISH`, `PUBREC`, `PUBREL`, `PUBCOMP`; If a packet is lost, it is retransmitted with the same IP and DUP flag set to 1.

#### AMQP

Heavier and more complex than MQTT, it is used for EBS and integration. Since it is complex, it is usually not suitable for Class 1 and below, but it works well with gateways and edge nodes. Extra features:

- Clustering
- Federation
- Security

#### Data Distribution Service - DDS

Distributed Pub/Sub, brokerless.

- Scalable
- Real-time
- Interoperable
- Dependable

For mission-critical environments and it is resource-consuming.

## Data Exchange Framework

They make data transfer easier. They can be seen as proxies that translate protocols to others.

### RabbitMQ

Infrastructure for message exchange.

- Distributed
- Cluster support
- Multi-protocols: MQTT, AMQP, HTTP
- Control plane by remote HTTP API invocations

It has:

- Topics
- Durable queue (persistent messages)

ACK:

- Automatic
- Explicit
- Cumulative ACK

### MTConnect

M2M message exchange between machines and application software. It is used mainly to share data and not to send commands or manipulate machines; for this reason, it is said to be **read-only**.
- Data formatted in XML
- HTTP RESTful interface
- LDAP for discovery services
### OPC
Connects Windows PC to SCADA devices. Architecture manager workers, Windows PC is an OPC client meanwhile machines are OPC servers. The interface between OPC servers and PLC is usually proprietary and provided by the PLC manufacturer.
### IoT Platforms
Should:
- **Connect** devices to gather information
- Secure communication with devices and applications
- **Manage devices** to control their behavior
- **Analyze** data
- **Build applications**

#### Amazon Web Services

- AWS IoT Device SDK connects devices to AWS IoT
- **Device Gateway: software proxy** that supports the publish/subscribe model.
- Authentication and authorization, AWS provides its own certificate and pipeline
- **Registry** with unique identity of devices, with **metadata** and capability
- **Device shadows**: virtual version of the devices, makes them always accessible. Set **desired future state** with rules and API.
- **Rules engine**: applies rules to data to transform and deliver them

#### Azure IoT
Flexible architecture to solve industrial IoT. Can combine many different services and modules. 3 Abstraction layers:

- **Things**: devices that can host edge applications with some connectivity
- **Insights**: aggregates and analyzes data through stream analytics and ML tools.
- **Actions**: visualized into graphs and dashboards

Main components:

- **IoT Hub** (on the cloud): manages the communications between services and devices, telemetry data device to cloud 
- **IoT Edge**: deploy and manage applications on devices. Avoids sending terabytes to the cloud.
    - **Modules**: containers. Unit of deployment, has credentials and permissions, has twin (JSON document with status and configuration)
    - **Runtime**: runs on each device, manages deployed modules
        - **Edge agent**: deploys modules
        - **Edge Hub**: responsible for communication
    - **Cloud interface**: remote monitoring of the device
- Stream Analytics: analytics tools, real-time elaboration
- ML: manage, deploy and train ml models on collected data
- Blob storage: non-structured data database

IoT Connectivity with Azure IoT SDK: C, Java, Python, HTTPS, AMQP, and MQTT.
#### Siemens MindSphere Platform

IoT platform based on Siemens PLC:

- Application system: CloudFoundry to deploy, scale, and monitor
- Services:
- MindConnect Elements: connect machines and plants
    - Provides connectivity from the machine to the cloud
    - OPC, MQTT, S7 protocols
- MindSphere Gateway: Routing between webapp, agents, edge nodes, backend. All defined in the registry.
- Services:
    - File service: CRUD, every resource is a file
    - Time series service: same as file service but for time series, periodically generated data from IoT devices.
    - Aggregator: creates synthesis from series and interfaces to read them.

Connectivity:
- Agent management: agent permissions on data, to each agent an access token.
- API: splits the OT (operational technology, shop floor machines) and IT (software)
- OPC UA pub/sub: API based on MQTT to give agent OPC UA protocol
Siemens is strong on the hardware.

#### EdgeX Foundry
Uniform communication protocols for industrial IoT. Works on edge computing level to give real-time and cloud-native architecture. Can be programmed like a data orchestrator, can interpret events (messages) and route them to endpoints, using metadata. Support for containers, seen as software plug and play. Data services are the software abstraction from physical devices, can add metadata.