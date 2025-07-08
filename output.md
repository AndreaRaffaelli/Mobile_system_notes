### Intro

Level-based architecture:

- **Kernel Based**: Based on Linux, it introduces a **Hardware Abstraction Layer (HAL)** that allows apps to run on many different devices.
- **Libraries**: Native C/C++ code
- **Android Runtime (ART)**: Application execution environment written in Java. Before it was the Dalvik VM.
- **Application framework**: Support through Java objects
- **Application**: Native applications

|DVM|ART|
|---|---|
|Application lagging due to garbage collector pauses and JIT|Reduced application lagging and better user experience|
|App installation time is comparatively lower as the compilation is performed later|App installation time is longer as compilation is done during installation|
|DVM converts bytecode every time you launch a specific app.|ART converts it just once at the time of app installation. That makes CPU execution easier.|
|It is slower than ART.|It is faster.|
|It does not provide optimized battery life as it consumes more power.|It provides optimized battery performance as it consumes less power.|

##### Note on the versions:

- Android 4.4: Introduced ART, then in 5.0 made it the default and only option.
- Android 6.0: Introduced request for user permission after installation/execution
- Android 7.0: Java on OpenJDK
- ...

Over time, the direction is to limit the freedom of the user to enhance the security and stability of the system.

### Kernel Level

The kernel level is based on Linux 3.x merged with the **Hardware Abstraction Layer (HAL)**. Actually, the Linux kernel lacks a native window manager, GNU C support and the standard Linux utilities, and introduces some extensions:

- **Shared memory manager (Ashmem)**: It has reference counting and automatic deallocation.
- **Binder IPC**: Inter-process communication. Allows communication between distinct processes and apps. Apps use this service also to access system services (such as the NotificationManager).
- **WakeLocks**: Battery policy manager.

HAL provides all the support for custom hardware:

- Memory management
- Process management
- Network stack

#### WakeLocks

They are locks to access the functions of the Power Manager. If an app has permission to access them, it can choose the policy for energy consumption. For example: the screen brightness or the CPU usage.

This is the perfect **example of the Cross-layering pattern**: the application level can directly work on the bottom hardware layers to control battery usage.

### Native Libraries and Dalvik VM

Native libraries are designed for heavy tasks like graphics and multimedia services and are accessible through APIs. They mask the lower layer of system services.

The **Dalvik VM** is a JVM machine optimized for mobile devices; for this reason it's register-based instead of the standard stack-based. Due to this particularity, the Dalvik VM requires that the source codes are compiled JIT for the device. The Dalvik VM also comes with its own bytecode (_dex_), but it's actually used only as a universal language between Android devices; it needs to be compiled. The compiled code doesn't get cached, so at each execution the device pays the energy cost to use a JIT compiler.

The new Android Runtime introduced ahead-of-time compiling at installation time, making the stored source code heavier, but saving the energy cost of using the JIT compiler.

### Application Framework

#### Components

Application level:

- **Activity**: Every action a user can make through a GUI. Usually it's a single screen.
- **Service**: It runs in the background and has no interaction with a user. Can work with multiple activities (even when the user is not directly interacting with the application) and can have a dedicated thread or work in the main thread.
- **Intent**: Application interaction enabler. To receive an Intent, a component should have a compatible Intent Filter.
- **Broadcast Receiver**: Receives and responds to compatible Intents. Its life cycle is bound to a single Intent. (An intent can activate several Broadcast Receivers.)

Support Level:

- **Package**: An APK contains the _dex_ executable files, the manifest (an _XML_ file that describes the application), static resources (png, style sheets, etc). All in a well-defined structure.
- **Activity Manager**: Manages the life cycle of activities.
- **Window Manager and View system**: Advanced graphic functions. The view system provides GUI components to interact with users and events. Developer defines a hierarchy of views describing the GUI, then the View system draws them on the screen. It also handles events and touches. The window manager handles the application in windows when not in full screen.
- **Resource manager** and **content provider**: The first handles all the resources (non-code) in the `./res` folder of the project. The latter allows access to structured data through SQLite DB; the data can also be shared with others (like media files).
- **Telephony, Notification and Location Manager**

##### Base Applications

Applications preinstalled on the device. The most common are:

- Home: single thread application to launch other apps
- Browser: web-based on WebKit

### Activities Life Cycle

States of an activity:

- `RUNNING` state: only one at a time and it is the one the user is interacting with.
- `PAUSED`: not active but still visible on screen.
- `STOPPED`: not active and not visible
- `KILLED`: not enough resources available

All the resources are usually dedicated to the `RUNNING` activity. The apps that have been deallocated are in the **Bundle** information holder that keeps the state to recover.

Life cycle management through callback methods:

- `OnCreate`
- `OnStart`
- `OnResume`
- `OnDestroy`

### Task

An application can be made of multiple activities. A task models the interaction between Activities and keeps a **LIFO** stack of them. The uppermost is the running one. When we press back we remove the activity from the stack. When we press home we move between different tasks.

### Intent and Intent Filter

Allows communication and exchange of messages between components, like activities and tasks.

- **Explicit**: the class to call is known at compile time `new Intent("ID", MyActivity.class)`
- **Implicit**: the class to call is to be discovered; the user selects the app to call from a menu. The intent has to specify:
    - Action and Category
    - URI of the data to pass
    - MIME Type: specifies the type of the data. 
    The activities are selected with an algorithm of Intent resolution based on the intent filters declared in the manifest.
```Java
<activity android:name="IntentActivity">
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
		<category android:name="it.mypackage.intent.category.CAT_NAME" />
		<data android:mimeType="vnd.android.cursor.item/vnd.mytype" />
	</intent-filter>
</activity>

protected void onCreate(Bundle savedInstanceState) {
...
	Intent intent = new Intent();
	intent.setAction(MY_ACTION);
	intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
	intent.addCategory(Intent.CATEGORY_BROWSABLE);
	Uri uri = Uri.parse("content://it.mypackage/items/");
	intent.setData(uri);
	intent.setType("vnd.android.cursor.item/vnd.mytype");
	startActivity(intent); //Intent Resolution Algorithm 
...
}
```
### Threading

Every application is associated with a single thread that runs each different activity (**single thread model**). Each thread has an endless loop to handle the message queue: external events (system or user inputs) or internal from the activities.

Options:

- Different Application in a single Dalvik VM (one heavy process)
- Dedicated Dalvik for each application: default option even if started by an Intent with `startActivity(Intent)` or `startService(Intent)`.

> If the caller wants a result it should use `startActivityForResult(Intent,int)`. The int serves as an ID for the asynchronous callback.

If two apps want to use the same VM it must be configured at compile time by setting the _`android:sharedUserId`_ in the manifest.

#### Scheduling

The developer has no direct control over scheduling; it is completely delegated to the Runtime. The priorities are:

- Running Activity
- Visible Activity
- Services
- Background

An app's priority is the same as the highest priority of one of its activities. Typically the CPU is assigned to the running activity for 90% of the time.

## Async Techniques

For the execution of application logic:

- Thread
- Executor
- Handler
- AsyncTask
- Service
- IntentService

### Service

Executes long-term operations in the background. It does not provide any user interface and can't modify the UI. It gets activated by an Intent.

Even though services are split from the UI, they still run on the main thread by default.

Examples of services: Network transactions, Audio streaming, I/O on files, DB interactions.

- **Unbound/Started Service**: Executes indefinitely until it reaches completion. Life cycle managed by the developer. `onStartCommand()`
- **Bound Service**: Client/Server interface. Clients send requests and receive results. Started with `bindService` and terminates when all clients disconnect. 
	- `onBind()`
	- `onUnbind()`
- **Foreground Service**: Needs user acknowledgement, like a music player. Higher priority than background; the user would notice their absence. They have a notification that can't be deleted while the service is working (battery policy).

If a service stops responding it blocks the main thread and also the UI, so it is useful to split into a different thread.

With a Broadcast receiver, services and activities can share information.

If an app goes to the background it also blocks the services; a background app can't start a service and gets back an `IllegalStateException`.

#### IntentService

Easier life cycle. Uses a worker thread to handle requests and stops when it is done. Ideal for a long job on a single thread in the background.

Just one request at a time and it can't be stopped.

```java
public class HelloIntentService extends IntentService{....
@Override 
protected void onHandleIntent(Intent intent){...} ...}
```


### Threads

Simple Java threads, but they can't operate directly on the GUI, since **Android GUI libraries are not thread-safe.**

They cannot be stopped with `destroy()` or `stop()`, but you must use:
- `interrupt()`
- `join()`

To use it:
- Write a class that extends Thread and overrides `run()`
- Give a thread instance an Object `Runnable` (implements the interface)
Then just call the method `start()`.
### Handler
New way to communicate between the main thread and new threads. It is a channel to send messages or executable code (an anonymous class that implements `Runnable`) to a thread.
- `post()`
- `postDelayed()`
- `sendMessage()`
- `sendMessageDelayed()`
- `removeCallbacks()`
- `removeMessages()`
```java
    @Override
    public void run() {
        // Prepare the Looper for this thread
        Looper.prepare();
        // Create a Handler associated with this thread's Looper
        this.handler = new Handler(Looper.myLooper()) {
            @Override
            public void handleMessage(Message msg) {
                // Handle the message received from another thread
                String data = (String) msg.obj;
                Log.d("TargetThread", "Received message: " + data);
            }
        };
        // Start the Looper
        Looper.loop();
    }
	public Handler getHandler() {
        return this.handler;
    }
```
### AsyncTask

Services can't update the UI. AsyncTasks run in a background thread and when they end they can update the UI to show results.

- `Params`: parameters
- `Progress`: unit size of the progress bar
- `Result`: result type expected

There are 4 callbacks to handle the life cycle:

- `onPreExecute()`: called from the main thread to show a progress bar
- `doInBackground(Params)`: application logic, works alongside the main thread
- `onProgressUpdate(Progress)`: the background worker called `publishProgress`
- `onPostExecute(Result)`: called from the main thread to show the results on the UI

Since UI handling can become messy and nested code is painful, this solution can ease the work and solve concurrency problems.

### Broadcast

Messages sent from the Android system or other applications. The message is wrapped in an Intent.

- `android.intent.action.HEADSET_PLUG` when headphones are inserted
- `ACTION_BOOT_COMPLETED` when the device is booted
- `ACTION_POWER_CONNECTED` when the device is charging

Useful to be aware of the context of the application, like when WiFi is connected or similar events.

The developer can code custom broadcasts for the application:

- `Ordered Broadcast`: The receivers have a priority number and receive the message sequentially. A receiver can propagate or abort the broadcast.
	- `sendOrderedBroacast(Intent, Permissions)`
- `Normal broadcast`: The receivers don't have a priority; they can't share results with each other. 
	- `sendBroacast(Intent, Permissions)`
- `Local broadcast`: Message sent and received inside the application, created with a `LocalBroadcastManager.getInstance(this).sendBroadcast(Intent)`.

The custom broadcast must have a common name between the Activity and the Broadcast Receiver to narrow the visibility of the intent inside the app. It's important to limit the reception of the broadcast for security reasons; the field `setPackage()` is also available to limit the broadcast to the selected application.

### Broadcast Receiver

They subscribe to receive broadcasts (**of any kind**):

- **Statically**: through the manifest
- **Dynamically**: Using a context or an Activity

The reception of the broadcast has a timeout; if it expires, the application is considered blocked.

To write a broadcast receiver:

- Register it:
	- **Dynamic**: `this.registerReceiver(mReceiver, filter);`
	- **Static:**`<receiver android:name=".MyBroadcastReceiver"> ` 
- Override the `onReceive()` method

In some cases it should specify a permission (just nominal) to allow the receiver to subscribe to the selected broadcast.

### Alarm

Software component that starts the execution of operations at selected times by sending Intents. It is based on real-time clock and also supports elapsed time. The application can be inactive or the device can be sleeping to receive the alarms.

Alarms are usually used in tandem with Broadcast Receivers that capture the Intents.

1. An activity creates a notification and schedules an Alarm
2. An Alarm triggers and sends the Intent
3. The broadcast receiver captures the Intent
4. The broadcast receiver shows the notification on screen

| |Elapsed Real Time (ERT) - since boot|Real Time Clock (RTC)|
|---|---|---|
|do not wake up|`ELAPSED_REALTIME`|`RTC`|
|wake up|`ELAPSED_REALTIME_WAKEUP`|`RTC_WAKEUP`|

Waking up the CPU with the screen turned off should only be for critical operations; it drains the battery. If the alarm is programmed to not wake up, it rings when the screen is turned on.

Using RTC can also cause a burst of requests to remote servers, so using ERT without waking up can add some randomness that can help the servers.

The Alarm also provides `setInexactRepeating()` that helps to merge a series of upcoming notifications together. This helps because it activates the CPU less frequently.

Exact time notifications are also avoided for real-time clock; now the notifications arrive grouped together and the developer can specify time windows.

### Internet Connections

To use the internet, the app must specify the permissions:

- `<uses-permission android:name="android.permission.INTERNET">`
- `<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE">`

The `ConnectivityManager` answers questions about network connection and its changes through the data structure `NetworkInfo`.

- `AsyncTask`: small downloads
- `AsyncTaskLoader`: larger downloads
- `HttpURLConnection`: to establish a new connection on a new Thread

Usually the results are parsed in `onPostExecute()` using helper classes.

#### Efficient Data Transfers

The radio interface can work at 3 levels:

- **Full power**
- **Low power**: 50% of full power consumption
- **Standby**: no connections available

The Android middleware merges multiple requests in the same transmission to leave the radio interface on standby for as long as possible.

The Battery Manager sends updates on power and battery using intents, so a developer with a Broadcast Receiver can choose the best strategy.

### Job Scheduler

Smart scheduling of heavy background tasks. Condition-based, not time-based. Much more efficient than `AlarmManager`; the scheduler groups tasks together to save battery.

- `JobService`: It starts the task, runs on the main thread
    - `onStartJob(JobParams)`: Called when conditions are reached. It uses child threads to run heavy jobs. Returns `FALSE` if the job ended, `TRUE` if it is running in another thread (it calls `jobFinished()` when done)
    - `onStopJob()`: If the Android system decides to stop the job because the conditions are not met anymore, it plans the job for the future. Calls `jobFinished(Params, boolean)`
- `JobInfo`: Sets the execution task's conditions. Takes the job ID, the service that runs the `JobService` and the `JobService` to run.
- `JobScheduler`: Does the task scheduling. Get one from the system and call `schedule(JobInfo)`

## Security

Each Dalvik VM has its own UID and GID. App permissions come from the UID, GID and from the manifest.

## iOS

It is based on a version of XNU, has Apple's API and full integration with Apple's environment. The SDK provided has all the common services: location manager, events, SQLite DB integrated, threading, etc.

Apple also provides an iPhone simulator for x86 devices.

iOS has many more rules and has a tighter policy for developers to increase the security of the system. The developer can only use Apple's API. The applications can be written in: Objective-C, C, C++ or JavaScript (run on iOS WebKit). The software must be compiled and can't run any interpreted code, plug-ins, downloaded code, etc. Each executable can be downloaded only from the Apple Store and it is signed to run only on the specific device.

A workaround for developers is developing web apps to run on Safari, but this limits the use of sensors and hardware.

### Unlocking and Jailbreaking

- **Unlocking**: Opening the device to work with every telecom provider
- **Jailbreaking**: Breaking the producer policies by gaining root access on the device. This allows running arbitrary code on the system, bypassing the security constraints.

## Cross Platform

_Write once, run everywhere_. Typically it's done by using HTML to develop the UI.

### Xamarin

It allows developing applications for Android, iOS, Windows with native UI using the same application logic. can a device know when the control point leaves, and stop sending event messages?













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
Uniform communication protocols for industrial IoT. Works on edge computing level to give real-time and cloud-native architecture. Can be programmed like a data orchestrator, can interpret events (messages) and route them to endpoints, using metadata. Support for containers, seen as software plug and play. Data services are the software abstraction from physical devices, can add metadata.Decoupling communication in space and in time.
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

## Intro
- **Physical Systems**: We find some coordinates for the device. Precise and consistent positioning in space, computationally intensive to process.
- **Logical Systems**: We find some labels that represent the location of a device in space. Fast to process, provide more privacy, hide the real position.
We can also distinguish:
- **Centralized Systems**
- **Decentralized Systems**
- **Absolute Systems**: they find the position in coordinates widely understandable without any further reference
- **Relative Systems**: they use a reference to which they relate.

Some key factors to evaluate when using physical systems are **accuracy (error-range)** and **precision (percentage confidence)**, scalability of the model and costs of the system.
## Physical based strategies
### Lateration
To identify a point on a 2D surface, you need 3 reference points. Once you determine the distance from each of these points, you can draw 3 circles (centered on the reference points) and the position will be at the intersection of the three.

To estimate the distance from a point, there are several methods available:
- **Time of Arrival**: It's based on the latency between the clock timestamp in the signal and the clock of the receiver, using the formula $d=v_s * ToA$ to calculate the distance between two points. This method assumes that both devices have synchronized clocks, which can be difficult to achieve on cheap mobile devices. If the clocks of the two points suffer from clock skew, the calculation will suffer from errors.
- **Received Signal Strength Indicator**: Based on the expected power of the signal, the perceived power can be used to estimate the distance. This method can suffer from known propagation effects in the real world: obstacles, reflections, etc.
- **Time Difference of Arrival**: This method avoids introducing a synchronized clock on the mobile device; instead it uses the received clock signals to estimate the position between two reference nodes. Getting the preliminary position from two pairs of reference nodes gives back the position on the surface. Using this method assumes that the clocks on the reference points are synchronized and their positions are fixed.
- **Angulation**: Place two reference points in fixed positions at a known distance. When a node emits a signal, the reference points receive it at a certain angle. Using the Carnot theorem, it's possible to compute the position of the third vertex of the triangle (the mobile device). This requires that the reference points have directional antennas.
- **Proximity**: Like in cellular networks, the localization is based on the proximity of a base station or service provider, like an NFC reader or POS terminal. The accuracy depends on the width of the covered area.
- **Scene Analysis**: Knowing some features of an environment, like the temperature or WiFi signals, the node can recognize its location based on the data collected from its sensors. This technique needs a preliminary phase to collect data on the environment to check afterwards with the data collected from the nodes.
### Known problems:

- **Non Line of Sight** (NLOS): The source of the signals is covered by obstacles so the perceived distance is affected by reflections and seems longer than it should be.
- **Fading**
- **Clock skew**: non-synchronized clocks can lead to false latency in the time of arrival
________
## Position in MANETs

How to calculate position in MANETs without using an external infrastructure.
### Assumption Based Coordinates (ABC)
Each node keeps a map of all the nodes reachable with a single hop.
A node decides to trace an axis using a close node as reference. It estimates the distance from it and gives it coordinates $(x_1,0,0)$. Then it takes a second close node as reference, calculates the distance and gives its coordinates $(x_2, y_2, 0)$. After this action it can trace the third axis and give coordinates to all the other close nodes $(x,y,z)$.

The least mobile node could also be elected as "**Anchor node**" and answer the requests for localization of the other nodes.
#### Terrain
It's an extension for **ABC** to monitor the position of the nodes outside the single-hop range.
Basically each node shares it's local map with the close nodes. Eventually the area covered can be $numhop * avgdistance$. 

Eventually positioning systems for manets are **not widely adopted**.
______
## Positioning systems with dedicated hardware
### Global Positioning System
Based on satellites orbiting at low heights (18,000 km) on well-known trajectories. The satellites send periodic signals to earth containing a clock timestamp that is used to calculate the latency and thus the distance. The intersections between the resulting spheres built on the calculated distances are usually two, and only one of those is on the ground, while the other one can be ignored.

This system suffers from some known problems:
- **The clocks must be synchronized**. An error of $1\mu s$ can cause a distance error of $300 m$. The satellites use atomic clocks, but the devices on the ground usually have cheap internal clocks. To solve this, another satellite is taken as reference to fix the clock skew.
- **Unstable satellite orbits** can cause errors up to $1-5m$.
- **Weather conditions** can cause errors up to $30m$.
- **Atomic clocks not perfectly synchronized** can cause errors up to $1.5m$
- **Multi-path fading** and reflections can cause errors up to $1m$

An easy fix is introduced by **D-GPS**. It uses some fixed base stations that know perfectly their own position; this is used to **calculate the error range in the GPS signals** and this value is sent to devices close to the stations. This fixes the errors due to the infrastructure, but multi-path fading and reflections still persist. However, the accuracy increases from a range of $15m$ to $30-70 cm$.

The US defense used to add a random error (**selective accuracy error**) on purpose to the GPS signals to prevent them from being used by other states. Nowadays many countries have developed their own GPS systems and the US moved from GPS to GPSv2, making it no longer useful to add an extra error to the GPS signals.
### Active Badge
Developed by Olivetti Research lab, it uses active badges to localize employees inside buildings. The badges use infrared signals to send information to receivers installed on the ceilings. This solution had room-level accuracy and didn't work well with sunlight and fluorescent light bulbs.
### Active BAT
This is essentially an upgrade of the Active Badge system. In this solution the badges can use both radio frequencies and ultrasounds to send information. The latter are very effective, but battery-draining, so they are used only to send signals when requested by the server. The server can send messages using radio frequencies and uses a matrix of receivers on the ceiling to estimate the position of the badges through the Time of Arrival of the signals. This solution had centimeter-level accuracy and worked well in every condition, but raised some ethical and privacy concerns.
___
## Positioning without dedicated hardware
This aims to localize nodes using already present technologies.
### GSM positioning
It uses signals from the cellular network to localize mobile devices, this is done through the **Cell Global ID**. The antennas can localize devices within their coverage area between $100m-30km$. Some improvements can be made using directional antennas and **Time of Arrival or RSSI** to determine the distance from the antenna.
#### E-OTD
This is a more sophisticated method that aims to achieve trilateration using 3 adjacent GSM antennas. The telecom provider forces a handoff between the 3 cells, so it is not very efficient and not widely adopted.
### Bluetooth positioning
Since Bluetooth has a small coverage area, the idea is to place several "_Points of Interest_" that are reference points in fixed positions that broadcast their MAC addresses periodically. A Bluetooth device can connect to them to estimate the RSSI. Since the Bluetooth protocol specifies 2 phases in the connection, the **inquiry scan and paging scan**, a device can be identified only by some POIs, at its own choice.
### WiFi fingerprinting
WiFi fingerprinting is a technique that needs a preparation phase in which a vehicle, with high GPS signal, drives around the city scanning the SSIDs of WiFi APs. Each SSID is associated with their RSSI and the GPS location. Then in the production phase, a mobile node can detect its location by scanning for available WiFi signals.
This technique needs frequent updates to the database of known APs, because the APs are not managed and can be moved or become faulty.
The database is usually managed by a centralized server that answers queries. This model may not be privacy-tolerant.
An implementation of this technique is PlaceLab, an academic project.
#### RADAR
It's a version of WiFi fingerprinting adapted for indoor use. A node uses the perceived RSSI to ask a central server to detect its position; it uses a UDP protocol.
#### Ekahau
It's a version of RADAR that uses ML to predict the position of nodes. The main principle used is "Rail tracking": a user will probably be near the last detected location and can't walk through walls. The detection of walls is done by the ML engine that tries to detect "hidden Markov models".

---
## Mixing location systems
### Sensor Flow
It combines different sources of positioning to obtain a better estimate. It could combine GPS, WiFi and Bluetooth signals.
### VTT
It's a middleware that selects the best source of positioning among those available. It uses **pure selection**. The value to optimize is $k_n = \frac{1}{(t_a \cdot v + \epsilon)}$, where:
- $t_a$ is the age of the position
- $v$ is the speed of the node
- $\epsilon$ is the accuracy
VTT is battery efficient since it turns off the non-selected positioning sources.
### Universal Location Framework
A real middleware that merges all sources. It returns a weighted mean value of the position taking into account all available values.
### JSR-179 and PoSIM
Standard Java API to get the position of the device independently of the system used. It is an abstraction layer with standard APIs to make the invocation transparent. It uses a middleware that does **pure selection**.
It's not widely used since Android introduced its own Location Manager.

PoSIM stands for "Positioning System Integration and Management", Unibo's proposal for a middleware. It is a middleware that allows specifying policies for selecting and merging positioning information.
