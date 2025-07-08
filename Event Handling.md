Usually are [[Internet of Things#MQTT| pub/sub systems]] with a store-and-forward model. Filtering properties on headers and content are appreciated. The primitives are the usual: **subscribe, publish, unsubscribe**. Usually are implemented with logically Centralized systems.
## Event Router 
Event routers are the broker for event driven architectures. They decouple pubs and subs. The must keep a routing table if in the case of distributed brokers.
They can be:
- **Centralized**, (with redundancy) 
- **Hierarchical**: the distributed brokers are managed in tree structure, every event is forwarded up to the root and then back down to all the other routers.
- **Cyclic/Acyclic**: peer to peer, allows redundancy but needs a [[algoritmi#Shortest Spanning Tree|spanning tree]] to avoid cycles. It is possible to use a DHT to auto configure the network.
- **Rendezvous point:** special routers that works as meeting points. 
### Routing
- **Basic**: **"subscription flooding"** the router knows every subscription in the system. If the brokers are distributed do not share the same sub twice.
- **Covering based**: The subscription-cover takes advantage of the fact that some subscriptions can "cover" other more specific subscriptions. A subscription A "covers" a B subscription if all messages that meet B also meet A. If a sub unsubscribes to the general topic the more specific gets trashed too.
- **Merging-based routing**: Merge two related topics in the same filter to have just one routing rule. Too large merges can lead to false positives, too strict reduces the benefits. 
The events usually follows the reverse path of subscriptions.
The routing can be:
- **Channel/Topic based**: there is an agreement between pub and sub and can also have a multicast address. 
- **Subject based**: like in [[managing#Observation|observability]] patterns
- **Header based**: filtering on the headers (like in SOAP and JMS)
- **Content based**: Greatest expressiveness but highest costs 
## Java model for distributed events
Integrated model in [[Discovery System#Jini / Apache River|Apache River]] for event distribution based on RMI. Consumers must subscribe using a `RemoteEventListener` and implement a `notify(RemoteEvent)` method. 
The `RemoteEvent` holds the source object, the unique id of the event and the `handback object`: when you receive a `RemoteEvent`, you often want to know **why** you registered for it or **what it’s about**, especially if your listener is handling multiple registrations. The `handback object` lets you **carry your own context or metadata** with the registration, and retrieve it again when processing the event.
The `handback object` is useful to implement event filtering. 

The subscription works with the **Jini lease mechanism**. 
It is possible:
- to define `DistributedEventAdaptors` that implements filters and QoS policies. 
- to define other objects to delegate to them the notifications.
	- It's important to have `handback` to handle the event 

Jini is a community of distributed objects working together through proxy: there is a **dependency inversion**, a client subscribes giving his proxy stub. The pub calls the methods on the proxy and it knows the location of the client, it's like a server receives the proxy for the client.

## OMG Distributed Data Service DDS
The OMG specific is pub/sub system **peer to peer, no brokers**. Designed for real time systems and data driven communication between publishers and subscribers. The DDS middleware gives an abstraction of the global data space, accessible to the interested applications.   
Instances in a stream flow are uniquely identified thanks the combo `<topic, key, object>` . It offers the support to content filtering an QoS negotiation. 

> **Corba Event Service**: not data driven and no QoS support.
> **Corba Notification Service**: filters, QoS, mandatory usage of CDR and IIOP

### DDS partitions
Partitions are name spaces that allow logical division of a subdomain: pubs and subs can choose at runtime to write and read from which specific partition. To communicate a `DataWriter` and a `DataReader` must write on the same topic of the same partition. For each partition it is possible to negotiate a different QoS.
### QoS
There are six possible QoS levels, which are always negotiable. A subscriber can receive a message only if the publisher will grant that level.
DDS gives two policy for reliability:
- **Best Effort**: **no grants on ordering and receiving**
- **Reliable**: **grants receiving and ordering** using ACK and retransmissions. All the data are kept in a queue waiting to be confirmed and elaborated.
The dimension of the queue can be set through the **HISTORY policy** and **RESOURCE LIMITS policy**. Through the **DURABILITY policy** can choose for how long keep the acked data for future requests:
- **Volatile**: no persistence
- **Transient**: RAM persistence
- **Persistent**: saved on the disks
Also support ordering, priority, temporal constraints, fault detection,....
## GENA
[[Discovery System#UPnP universal plug in protocol|UPnP]] Control point is listener of modifications of device state.
1. get the device address
2. get the device descriptor file
3. get the URL for eventing
4. subscribes
Control point must subscribe, the device sends the full current state value registry, then sends notifications only for changes.
## Web Services
[[The Ultimate Bullet Point List for SIST_DIST25#Web Service|Web Services]]
### WS eventing
Web services can accept registrations for event notifications.
- Expiration time and allow renewal
- Support to filters
### WS notification
Data dissemination to other web services.
- Interest based filtering
- Distributed topologie for notification brokers

Both allows direct subscription or third parties routing (with broker) for decoupling.
