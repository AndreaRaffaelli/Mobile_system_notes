In traditional software architecture, the layers of a stack, like ISO/OSI, are carefully separated and independent from each other, each using standard interfaces to communicate up-down. In the mobile world this is usually not true, since communications are challenged by the scarce nature of the medium.

In fixed and infrastructured networks, the middleware are designed for devices with no constraints on energy and bandwidth. In mobile networks the working context changes dynamically and time and space decoupling is a must. See-through layers allow making wiser choices in terms of communication, battery savings and better QoS.
## Principles
Some principles true also for the mobile world:
### Internet Principles:
- End To End
- Robustness
### Web Principles:

- **Least Power Principle** ("Tim Berners-Lee"): keep the internet free and accessible by using easy technologies and widely interoperable standards.
- **Stateless Interactions**
- **REST architecture**
### SOA principles:
Use well-defined interfaces, message-oriented, loosely coupled services running on remote nodes discovered at runtime.
## Cross-layering: Principles for mobile

Strategy for the mobile world used to overcome the unstable reality of the media.
- **Upward flow**: management information is visible from the upper layers.
- **Downward flow**: management information is visible from the bottom layers.
- **Back and forth flow**
- **Merging adjacent layers**, examples:
    - I-TCP needs MIP to handle handoffs.
    - HTTPS is not a merged layer: HTTP and TLS are still completely separated.
- **Coupling**: a layer is designed to work only with a specific version of another layer.
- **Vertical calibration**: some parameters are used to calibrate communication at upper layers, like if the RSSI is low the transport protocol could decrease the bit rate. If the bit rate is too low the application could switch to a different stream.
## Architectural Patterns
- ***Level Based***: Multi-layer software with each layer having a different responsibility, rigidly separated
- ***Client-Server***
- ***Peer to Peer***: a device can become alternately server and client
- ***Pipeline***: chain of processing elements to reach the final result
- ***Multi-tier***: each tier has its own responsibility and is executed by different software agents
- ***Blackboard***: Common shared memory between agents that can also be used as a synchronization mechanism
- ***Publish/Subscribe***
## Mobile Computing Patterns

- Distribution patterns
- Resource management
- Synchronization
- Communication patterns
### Distribution Patterns
#### Facade pattern
It's a coarse-grained interface through fine-grained independent services.
It's an entry point that hides the multiple concurrent servers used in processing the response. The facade is designed for mobile clients to make communication easier: usually it takes in an asynchronous request and deals with multiple synchronous requests.
#### Data Transfer Object
Since communications are unstable, it's better to work with a single data structure that holds all the information needed by clients than using several data objects and multiple remote invocations. The DTO is a serializable object and can be populated with _setters and getters_.
#### Remote Proxy
It's a proxy placed between the devices and the network. It mediates all requests and can do heavy computation for the clients. The proxy can implement special protocols and solutions while leaving the mobile nodes unaware. Usually a discovery phase is needed to find the proxies.
#### Observer
The observer can "look" for changes in the _subjects_. Actually, the subjects notify the observer through a callback method. This decouples the two elements and supports group communication, but eventually is not scalable without middleware.
### Resource management
#### Session Tokens
For each client, a unique key is generated that represents its identity. With the session token it can present itself to the server and access state as it was left. The session key is valid throughout the session, but it should not be persistent.
#### Caching
Clients can keep a copy of the data locally to be able to work also in conditions of disconnection, save bandwidth and battery. When a request is made, first the cache is checked and if not present, the result of a remote request is cached.
#### Eager Acquisition
The client fetches more data than it needs preemptively, so it can access information locally when needed and the general responsiveness will increase. The client saves bandwidth and needs fewer remote invocations.
#### Lazy Acquisition
The data is accessed only when needed. This keeps the application lightweight and saves bandwidth if data is not accessed. The data, once fetched, is cached and kept.
### Synchronization
The synchronization can happen at the process level or at the data level. At the process level, access is sequential and the current user blocks the resources; they are locked and the operations on them are atomic. It's a pessimistic protocol. At the data level, more processes can access the same data concurrently but some management is needed for reconciliation of the various versions available.
#### How to Reconcile
- **When:**
    - Automatically
    - Manually
- **How:**
    - **Pessimistic**: Each change must be propagated to every copy available. This may not always be feasible because it can severely affect the user experience.
    - **Optimistic**: Each version can be modified independently and a synchronization engine will take care of merging the several copies. Sometimes in this phase inconsistencies can emerge and some changes must be invalidated in a fair way.
#### Techniques
- **Versioning**: Each data item has a version number that reflects the causality. If change B comes after A, it must have a number greater than A. If B and A are changes not related to each other, they should not be comparable.
- **Change Detection**: Clean and dirty copy, chunks, etc.
- Reconciliation should also take account of semantics and syntax.
    - **Logs with changes**: From the common state, replays the actions in the logs.
    - **State comparison**: Operates directly on changed data.
#### Architecture
Eventually, the reconciliation is done on two copies at a time. Some available architectures are:
- **Centralized**: For each data item there is a master node that manages the reconciliation
- **Binary Tree**: For each data item there is a master root and the copies are nodes. The users are the leaves. The reconciliation goes from the bottom to the top.
- **Cyclic Graph**: General and complex, not easy to understand which version to use for reconciliation.
The reconciliation algorithms work better if they are aware of the structure of the data, the syntax and the semantics.
#### Distributed File Systems
These applications need to reconcile unstructured data as a fundamental functionality.
- In DFS, a copy can be local or remote. If the local copy is modified, remote copies must be updated; sometimes the user is needed for the reconciliation.
- In NFS, there is no need for reconciliation; there are no copies, all the files are stored on the remote server and all the changes act on it. Every change is instantly visible on each node. This approach is favorable for total consistency but may not scale if large volumes of changes are made on the same file. Even large volumes of reads can stress the server.
- **Concurrent Version System**: Initially just put a lock on a file making it editable by only one process. Since it was not scalable, now it uses an optimistic **3-way merge**, conflict detection and occasionally human action.
In the mobile world, a known protocol is the Synchronization Markup Language used for emails and databases.
#### State Transfer
It's a pattern useful to synchronize activities on two different nodes. It is implemented with a rendezvous point, logically centralized, always available and stateful. A node moving from one domain to another, before the handoff, could save its state on a rendezvous point to transfer its state and keep the session alive after the handoff.
### Communication pattern
#### Connection Factory
Using a connection factory can enhance code reusability, decrease code complexity and reduce time to market. It can also decouple application code from the management part.
#### Multiplexed Connection
Using the same connection can improve performance and speed things up since each connection carries overhead on the bootstrap, negotiation and bandwidth. A connection factory can easily achieve this pattern and also support different priorities for each virtual connection (example: Stream Control Transfer Protocol).
#### Client Initiated Connection for Push Model
Since mobile devices are usually not reachable from the open internet due to firewalls and NAT in managed networks, clients initiate the connection towards publicly reachable servers (Edge Proxy or also known as **Push Servers**) and then keep alive a long-term connection. This connection can later be used by the Push Servers to deliver notifications. Usually this kind of connection is supported by the WebSocket protocol or something more specific like MS Direct Push (note: never heard of it). Generally browser and mobile middleware use the **Web Push protocol**.
