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
