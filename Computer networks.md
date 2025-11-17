# Overview of the protocol layers

Devices in a computer network interact via protocols defined in the following five protocol layers:

	5. Application
	4. Transport
	3. Network
	2. Data link
	1. Physical

The above layers are described in the book *Computer Networking: A Top Down Approach* by Jim Kurose and Keith Ross. In the *Open Systems Interconnection* (OSI) model of the internet, there are seven layers, since the application layer is expanded into the application, presentation, and session layers.

When a message is sent from an application on one computer to an application on another computer:

	5. The application-level message is broken down into segments.
	4. Transport layer protocols are invoked to send each segment from the appropriate OS process on one host to the appropriate OS process on the other host.
	3. Transport-layer protocols themselves invoke network-layer protocols to send a segment from the one host to the other.
	2. Network-layer protocols invoke link-layer protocols to send the message between physically connected hosts.
	1. Finally, link-layer protocols invoke physical-layer protocols that use physical principles to solve the basic problem of digitally representing and transmitting data.

A higher-level protocol passing a message to a lower-level protocol more specifically entails that a header associated with the higher-level is sent along with message content by the lower-level protocol. The message content is called a *protocol data unit*, or PDU.

The following table includes the names the OSI model gives to PDUs at different protocol layers.

| Layer | Layer name | PDU name          | Types of sender and receiver |
| ----- | ---------- | ----------------- | ---------------------------- |
| 4     | Transport  | Segment, datagram | OS processes on hosts        |
| 3     | Network    | Packet            | Hosts                        |
| 2     | Data link  | Frame             | Physically connected hosts   |
| 1     | Physical   | Bit, symbol       | Physically connected hosts   |

The sender and receiver are computers that support protocols in all five layers. Once the message has been represented as a layer-3 PDU, it is passed in-between intermediary devices (*hosts* and *routers*) that support only the first three protocol layers.

<img src="https://book.systemsapproach.org/_images/f01-13-9780123850591.png" alt="../_images/f01-13-9780123850591.png" style="zoom: 25%;" />

# Summary of protocol layers

## Basic definitions

- (Host). A *host* is a device capable of sending and receiving PDUs to and from physically-connected neighbors. 
- (Switch). A *switch* is a device that accepts PDUs from multiple hosts, determines destinations for these PDUs from the PDUs, and sends the PDUs to their destinations.
- (Physical connection). Hosts are considered *physically connected* if they are connected via a wire or wireless connection.
- A computer network can be thought of as a graph in which the nodes are hosts and switches and the edges are physical connections between nodes.

## Application: *message* sent from application to application

## Transport: *datagram* sent from process to process

- TCP
- UDP
	- Is called a "connectionless" protocol because recipients of UDP messages do not need to consent to in order for messages to be sent to them.
	- Is a very simple, "bare bones" protocol that simply sends messages and hopes for the best.
	- UDP datagrams may be lost or delivered out-of-order. Comparison of checksum metadata is used to implement error detection, but no means of error correction is provided.
	- Is used for transmitting media like audio or video, or online action-based videogames. Error correction is not as important for this use-case because a couple of corrupted 

## Network: *packet* sent from host to host

### Forwarding and routing

The "data plane" aspect of the network layer concerns the *forwarding* of packets, while the "control plane" aspect of the network layer concerns the *routing* of packets. *[Computer Networks: A Systems Approach](https://book.systemsapproach.org/internetworking/routing.html)* gives the following definitions:

* *Forwarding* consists of [a network-level switch] receiving a packet, looking up its destination address in a table, and sending the packet in a direction determined by that table.
* *Routing* is the process by which forwarding tables are built.

## Network - data plane

When the "data plane" aspect of the network layer is mentioned, usually the speaker is referring to conventions, like "IP addresses", that are involved in the implementation of packet *forwarding*.

- (Router). A *router* is a network-layer switch.
- (Host and router interfaces). Hosts can have multiple *interfaces*; for example, a host might have a wired interface (e.g. Ethernet) and a wireless interface (WiFi). Routers have multiple interfaces; each one is associated with a host interface.
- (IP addresses). Each host and router interface is identified by *internet protocol (IP) address*, which is a 32-bit identifier of the form x1.x2.x3.x4, where each x_i is an 8-digit binary number, or *octet*. IP addresses are most commonly written in their *decimal form*, which is obtained by converting each octet to a decimal. for example, 11011111.00000001.00000001.00000100 [binary representation] = 223.1.1.4 [decimal representation].
  - IPv4 addresses are 32-bit. IPv6 addresses are 64-bit.
- A *subnet* is a collection of hosts that are immediate "children" of the same router. In a subnet, there exists an IP address prefix shared by all hosts in that subnet. This prefix is called the *subnet mask*.
- In *classless interdomain routing* (CIDR), up to the first 8 * 3 = 24 bits of an IP address can be used as the subnet mask. In *CIDR notation*, an IP address whose subnet mask is specified by the first k bits of its binary representation is denoted as y1.y2.y3.y4/k, where y1.y2.y3.y4 is the decimal representation of the IP address.
  - (Example). Consider the IP address 223.1.1.4 [decimal representation] = 11011111.00000001.00000001.00000100 [binary representation], and suppose the subnet mask is 11011111.0000000, i.e., the subnet mask is specified by the first 15 bits of the binary representation. Then the CIDR notation for this IP address is "223.1.1.4/15".
- (Network address translation). In a local network (a subnet plus its router), each host has a local IP address, while the router has both a local IP address as well as a global IP address. The router's *network address translation table*, or *NAT table*, uniquely assigns local IP address to an octet that is called a *port*. Thus, the NAT table of a router `router` allows for the mapping of each `host.localIPAddress` address to the pair (`router.globalIPAddress`, `router.getPort(host.localIPAddress)`) and for the mapping of the pair (`router.globalIPAddress`, `port`) to `router.getLocalIPAddress(port)`. These mappings are used to allow the router to send and receive packets on behalf of its hosts.
- (Assignment of IP addresses).
  - (To ISPs). Each ISP is allocated a subnet mask by Internet Corporation for Assigned Names and Numbers (ICANN).
  - (To networks). Networks that are subnetworks of an ISP's network are allocated a "subsubnet mask" of the ISP's subnet mask.
  - (To hosts). A host's IP address is either
  	- Set in a configuration file.
  	- Obtained dynamically from a server via Dynamic Host Configuration Protocol (DHCP). In DHCP, messages are sent back and forth between the host and the *DHCP server* associated with the host's network's router to facilitate the requesting and allocating of an IP address.  
  		- Q: How are messages sent and recieved if IP addresses haven't been established yet? A: Broadcasting is used. Q: But then how does the DHCP server ensure the client accepting an address is the one requesting it? A: Each client is required to send their MAC addresses when sending a request to the DHCP server and when accepting an offer from the DHCP server. The DHCP server can therefore compare the MAC address recieved in the request to the one receieved along with the acceptance of the offer.

## Network - control plane

The "control plane" aspect of the network layer concerns *routing* packets *within a network*, which is determining which of the multiple paths between a sender and receiver a packet takes.
- There are two ways to implement routing. The traditional and most common way is to configure the forwarding table of each router one router at a time. In the more modern *software-defined networking*, a single software program is used to configure the forwarding tables for all routers.
	- E.g. OpenFlow is a software-defined networking protocol
- The Internet is divided into different subnetworks called "autonomous systems" or "domains". All routers in the same domain use the same routing protocol. *Gateway routers*, or *gateways*, perform routing between domains.

### Classical routing algorithms

- In a *link-state* routing algorithm, each router knows the (global) weighted-graph of the entire domain. (In practice, all routers in a network come to know this information because every router sends a message containing this information to every other router.) 
  - E.g. Dijkstra's algorithm
- In a *distance-vector* routing algorithm, each router only knows the (local) weighted-graph that contains it and its neighbors.
  - E.g. Bellman-Ford algorithm

### Logically centralized control

In the more modern approach of logically centralized control, a remote computer installs forwarding tables in routers.

## Link: *frame* sent between physically-connected hosts

- Frames can be sent by different link protocols over different links (e.g. WiFi on first link, Ethernet on second).
- A *local area network*, or *LAN*, is a set of physically connected hosts that all share the same network-layer router. (A set of physically connected hosts that have more than one network-layer router, i.e., a set of physically connected hosts that include hosts from more than one LAN, is called a *wide area network*, or *WAN*.)
  - A *wired LAN* is a LAN in which all physical connections are wired connections. Historically, the first LANs were wired LANs.
  - (MAC addresses). Every host has its own *MAC address*, which is an identifier in the format x1.x2.x3.x4.x5.x6, where each x_i is a two-digit hexadecimal number. Since each hexadecimal number requires four bits to specify, and as there are 6 * 2 hexadecimal numbers in a MAC address, then a MAC address is a 6 * 2 * 4 = 48 bit identifier. MAC addresses are also referred to as "LAN addresses", "Ethernet addresses", and "physical addresses".
  - (MAC address vs. IP address). MAC addresses are to social security numbers as IP addresses are to postal addresses. Every host has a MAC address that never changes. A host's IP address changes when the network it is connected to changes.
  - Every network-level node (i.e. every host and router) has an *address resolution protocol* (ARP) table, which is used to determine a network layer node's MAC address from its IP address. When a node's ARP table is missing an entry it needs, it sends out a broadcast to its LAN to initiate the process of getting the appropriate information.
  - "Ethernet" is the dominant wired LAN technology. There are two type of Ethernet:
    - Bussed Ethernet, in which multiple hosts share a single wire (called the *bus*), was popular through the mid '90's.
    - Switched Ethernet, in which hosts are only physically connected to switches, prevails today.
  - Ethernet is a connectionless protocol.

## Physical: *symbol* sent between physically-connected hosts

* Byte-oriented and bit-oriented protocols
* Multiple signals can be sent over the same link via time-division multiplexing or frequency-division multiplexing

# [Network technology in the real world](https://book.systemsapproach.org/direct/perspective.html)

| Portion of network | Technology                                                   |
| ------------------ | ------------------------------------------------------------ |
| Backbone           | Fiber-optic cables and refrigerator-sized routers            |
| Last-mile          | Copper phone lines - Digital Subscriber Line<br />Copper lines - G.Fast<br />fiber-optic cables |

| LAN                | Ethernet, Wi-Fi                                              |


