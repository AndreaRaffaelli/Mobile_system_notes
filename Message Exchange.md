Decoupling communication in space and in time.
First of all must define:
- Principle and architecture
	- **Loose coupling**
	- Extensibility:
		- Version ID
		- Formats with extension points
		- Forward compatibility: (**robustness principle**)
	- Not a transparent middleware but a **see-through** middleware highly configurable
- Message syntax
	- Translating form a format to another (**marshalling/unmarshalling**):
		- Interface Definition Language with automated stub skeleton generation
	- Messages can specify the type of the values in the message to avoid marshalling/unmarshalling (more expensive)
	- Agreement of format
		- Common Standard (need marshalling/unmarshalling)
		- Negotiation (overhead)
		- **Receiver makes-right**: sender uses native type, receiver has to translate. 
			- Only one translation per communication 
			- Difficult to deploy new format translations.
	- Existing: binary ASN.1, human readable: XML, Json.
- Protocol for message exchange:
	- Session Level vs Transport level. 
		- Session level can hide multiple transport connection of a mobile device
		- Session level can be also reversed, clients connects to server with tcp, but server sends packets in push.
	- End to End vs Store-and-forward architecture
	- One-way, Two-way, request response, sub-notify, conversation. 
	- Blocking / Non blocking, polling/callback
- Locator: 
	- Transparent, like URI as in Web Services (WSDL) interface+physical endpoint or UPnP
	- Opaques, need a middleware to translate them\
	Sometimes mobility can be handled also at session level.

### Mobility Handling:
- Proxy roles: Splitting the connection in half, breaks the end to end principle
- Mobile nodes can't offer services since there are technologies masking them (NAT and Firewalls).
- How to complete message delivery if a node moves? **Rebinding problem**
- Heterogeneous devices, easy syntax, reduced content, use compression for smaller bandwidth.
- Security handling: **Session level with SSL** o at **message level** with encryption to **only some parts of the message**.
### Reliability
Acknowledgement and transmissions, also at application level.
ACK:
- Standard
- Cumulative
- Negative (only if one missing)
- piggy-backing (added in the response)
In fixed networks usually are lost more packets together, instead in mobile networks just single packets.
In order delivery can be scarified for the sake of performance.
## Java Message Service
It is a specification to give an abstraction layer from the messaging service provider actually used and the protocols applied underneath. It defines a series of specification for providers to implement and make homogeneous message exchange architecture for all the Java applications. Since each provider implements the API its own way, all the system components must agree on the same provider. 
#### Semantics
- Exactly once -> Transactional delivery. Not really achieved in all conditions: If the broker fails during reception of ack starts re-transmissions
- At least once -> Persistent messages and durable subscription
- At most once -> May loose packets.

- Pub/sub -> Topics
- Point to Point -> Queues
#### Persistence
`setDeliveryMode(DeliveryMode.NON_PERSISTENT)`
- Persistent messages: A message sent to a queue are stored persistently until any consumers receive it.
- Durable Subscription: Messages in a Topic are stored until the subscriber receives them. The subscription can be also passed to other consumers. The consumer can also disconnect.
#### Acknowledgement
`connection.createSession(false, Session.AUTO_ACKNOWLEDGE);`
- `auto_acknowledge`: the driver support takes care of acknowledging the receiving of each message
- `client_acknowledge`: the developer must take care of cumulative acks, can increase performances.
- `dups_acknowledge`: the driver takes care care of acknowledging the receiving of each message **lazily**, this means that the application level must take care of duplicates **idempotent messages**
#### Priority and time to live
To achieve further quality JMS supports priority and time to live, useful for persistent messages.
`producer.setTimeToLive(60000);`
`producer.setPriority(7);`
#### Ordering
Inside the same session the messages are usually sent to the broker in order, but from the receiver PoV only the messages inside the same topic/queue are received ordered.
To achieve more reliability a **transactional session** must be created. In this way all the receiving are atomic even from more topics and queues. It can also have some support from `JTA`.
`connection.createSession(true, Session.AUTO_ACKNOWLEDGE);`
#### Asynchronous Message Receiving
Allows a consumer to receive messages without blocking the application while waiting for messages to arrive.
Consumer registers a `MessageListener` with the JMS provider.
The JMS provider typically manages the threading for the listener. When a message arrives, the provider creates a new thread (or uses a thread from a pool) to invoke the listener's `onMessage` method.
```java
consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                try {
                    if (message instanceof TextMessage) {
                        TextMessage textMessage = (TextMessage) message;
                        System.out.println("Received: " + textMessage.getText());
                    }
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
```
## Corba Messaging
Corba Messaging includes the **Asynchronous messaging Interface**, can support either polling and callback. Decouples clients and servants. Since Corba was born as a distributed RPC model and it has a strictly synchronous dependence. To achieve an asynchronous communication it is necessary to add a third CORBA object "Handler" that the servant can synchronously call to deliver results.
- Polling: the main CORBA client can ask periodically the holder fro results
- Callback: the Holder executes the callback code instead of the original object. This is transparent for the original object. The server sends a **fire and forget** message, actually is a remote function call.
The Handler is automatically generated form the `IDL` compiler.
The developer has to write only the signature with input and output parameters, the IDL compiler will split the method in half: the servants will call a return method to pass the results as input.
#### Time independent invocationt
Support is not mandatory in ORBs, they have to explicitly support it. 
Routing Object:
- third Object in the middle between the client and the servants.
- Works as a Store and Forward Broker.
- Standard Corba Object with buffering functionalities
The routing object has a configurable persistent queue:
- FIFO
- Priority
- Timeout
It can add **routing policies**: when, where, how to route messages. The clients can specify the routing policies.
Results can directly go from the servants to the Handler obj according to AMI, some routers decouples also answers. 
## XMPP
### Extensible Messaging and Presence Protocol
Instant messaging. Includes publish/subscribe, service discovery, state updates. **Client Server model**, the clients after negotiating parameters sends a XMPP flow. **Server coordinates in a peer to peer** model to deliver messages (**federation**).
This protocol introduces the **stanzas** concept:
- **Message stanzas**: one to one, email like
- **Presence stanzas**: pub/sub mechanism
- **Info/query stanzas**: request response mechanism 
#### Technicalities
- The payload of a XMPP message is formatted in XML. 
- XMPP works on top of TCP.  
For each TCP reconnection (the mobile device changed position) a new XMPP session must be negotiated. 
XML + New session every disconnection cause this protocol to be very heavy for mobile devices. 
Android introduced its own version with a different payload format and with a persistent TCP connection **(bidirectional stream, push model)** .
#### Integration
The infrastructure allows **XMPP gateways**, with this entities it is possible to add services like SMS, Emails... 
Each device has an **JID** (Jabber ID formatted like emails) that abstract the real position of the users.
## Web Service
The Locator is a unique URI HTTP. In the WSDL description are then identified several available endpoint, physical implementation. The implementation can be SOAP, with XML-like messages. 
There are several binding protocol available, but HTTP is the most spread, since firewalls policies. Other protocols are also SMTP (email) and XMPP.
### REST
Web services are heavy since they are state-full. The Rest architecture wants stateless interactions, caching oriented. 
Every resource has an unique persistent URI and the content of the messages are their representations. The messages are mostly formatted in JSON (rarely in XML) and the main protocol is HTTP.
In the **HTTP headers** are contained **metadatas**:
- **Resources Metadata**: Timestamp on last modification ecc..
- **Representation metadata**: format and media type
- **Control metadata**: message length and caching options
Well known implementations are the RESTful web services. Service operation supported by http methods invocation.
