
CCNA - Routing and Switching

CCNA Routing and Switching 200-125 Certification: A Perfect Guide Summary.

One of the most relevant certification for holding the networking expertise up to date is the Cisco Certified Network Associate (CCNA) Routing and Switching. The CCNA Routing and Switching 200-125 Certification Guide includes analysis and practise questions as well as subjects included in the most recent CCNA exam. This guide walks you through the structure of IPv4 and IPv6 addresses, as well as how to build IP networks and sub-networks and allocate addresses to them.

### Models of Internetworking

- Routers
- Bridges
- Hubs
- Topologies of networks
- The topology of the bus
- The topology of the star
- The topology of the Ring
- The OSI model
- The layer of the application
- The layer Session
- The layer of presentation
- The lower layers
- The Transport layer
- The Network layer
- The Data Link layer
- Logical Link Control (LLC)
- The Physical layer

At the conclusion of this article, you’ll have a better understanding of networking best practises and the protocols that a network administrator may use to control the network.

Models of Internetworking Let’s have a look back at how it all started before we start our journey into the field of networking. The Galactic Network definition, which involves social networking, was coined by J.C.R. Licklider and W. Clark in 1962. The first edition of ARPANET (internet) did not go online until 1969. It linked four devices from four universities:

the University of Utah, Stanford Research Institute, UCLA, and the University of California, Santa Barbara. So, in our computer technology culture, we’ve been attempting to connect with each other using various forms of networks since the 1960s. The Defense Department was one of the first organisations to create a technology that would enable us to connect around the globe in the event of a major disaster. Well, we had the phone system and the post office, and we could transmit details around the globe by air and sea, but we were dissatisfied with the scalability, interoperability, and reliability of these networks, and you needed a PhD to even get near to one of those monstrosities known as machines.

Computer networks did not become more prevalent in small and large enterprises until the mid-1980s. This was thanks to the powers that be; they developed the TCP/IP network suite, which enabled the majority of the world to relay information within their industry, allowing them to be more effective in bringing information into the hands of those that needed it in a timely and effort-free manner. In the following sections, the following topics will be discussed: Internet-connected computers Topologies of networks - The OSI model is a diagram that depicts the hierarchy of services and Internet-connected computers. With that in mind, let’s begin by discussing internetworking devices and their place in the network.

### Routers

Routers are devices that enable you to send and receive is planning to move the processing workloads to dataThe most knowledgeable computer on the network is this one. It manages the network’s traffic and routes it to the appropriate destination. Routers have an Internetworking Operating System ( IOS) that helps you to customise it with the network parameters

you need to bring the is planning to move the processing workloads to data across:

Switches come with a variety of flavours, which means that depending on the IOS and the needs of the network, they can have various functionalities. Layer-two switches would be the subject of our research for qualification purposes, but we’ll go through several layer-three switching features quickly as well: On a network, the primary aim of a switch is to provide functionality. The switch is where all of your computers would be wired in order for them to interact with one another, but it also has a number of functionality that we can use to make our network more effective. Any of such characteristics are discussed in the following bullet points: VLANs Switchport security Spanning Tree Protocol EtherChannel

And, depending on the IOS, there’s a lot more. The switch has the same components as the router, but it still keeps a VLAN storage file that you may know about. All of these features, as well as their specifics, will be revealed later in the book. Bridges

Bridges are structures that connect two points.Bridges are similar to switches, but they have smaller connectors, are software-based rather than hardware-based, and have fewer features: Bridges run at layer two and have the primary role of segmenting the network. They also make a number of collision and transmitted territories.

Hubs are places where people get together.In today’s IT nation, hubs are no longer seen on networks. Hubs are non-smart machines. They’re layer one devices with the primary purpose of acting as a multiport repeater. It would generate one collision domain and one transmitted domain, which is extremely undesirable, especially in an Only keep in mind that using hubs in your network can slow things down. cabling for the network

I understand that cabling is not an internetworking system, but the form of network cabling utilised while installing, restoring, or upgrading a network is critical. The standard CAT5e cabling used to link end devices to internetworking devices to enable them to communicate as seen in the illustration below. We’ll go into cabling in more detail later, but for now, bear the following in mind: Topologies of networks Let’s speak about topologies now that you’ve been exposed to the internetworking computers. Let’s start by defining what a topology is. There are two kinds of topologies: actual topology and logical topology. Physical topology refers to how the network is physically linked. The logical topology, on the other hand, describes how rel="bookmark">How to: EIGRP Routing Protocol Implementation & Tutorial.

The topology of the bus Both end devices are attached to a primary cable in a bus topology. The data flows through this wire, which is why it’s called a Bus. The concern is that we were using coaxial cabling at speeds of 10 Mbps at the time this form of topology occurred, which is deemed sluggish by today’s standards. Since the bandwidth was split depending on how many machines were linked, it was considered a common network. The fundamental form of Bus topology is depicted in the diagram below:

Ethernet technology was used in this topology, which employs the Carrier Sense Multiple Access Collision Detection (CSMA/CD) access process. End devices in Ethernet use

the CSMA/CD system to relay data. As previously stated, if a system detects any noise on the wire, it will not transmit; instead, it will wait until all noise has vanished before sending its results. It’s possible that one node or computer isn’t hearing the other, and all end nodes are sending at the same time. This will result in a collision; at that moment, a jamming signal will be transmitted, packets will be lost, and a countdown will begin to see who will transmit first; the person whose countdown ends first will transmit first.

## CCNA Routing and Switching Explained

The topology of the star You may be saying, “Hey, that doesn’t look like a star,” and you’d be right. Only because it’s called a Star doesn’t mean you’ll build the physical network in the same way. It basically implies that you’re linking all of your gadgets to a single hub from which they can communicate:

The preceding diagram depicts the reality of a typical network architecture. Your cable will be run from your workplace, cubicle, or classroom to the communications closet, where it will be terminated at the patch panel. This is then connected to the switch, which is then connected to the router using patch cables. With all of that said and explained, I hope the Star topology concept is now simple. How to Download WhatsApp Status Videos: WhatsApp Status Images and Videos from Internet

The topology of the Ring A token ring network is defined as a circle or ring in the illustration below, but token ring networks are more than that. The aim of a token ring network’s central system, known as a Multi-Station Access Unit (MAU), is to link all end devices to it: Token passing is a form of access mechanism that is deterministic, unlike Ethernet, which is dependent on contention. This means that a token ring has a null, free-flowing token that travels across the network waiting to be seized and used to transfer info. Only the owner of that token has the ability to transmit, and if that token is seized, no more tokens are created. As a result, no one else will transmit before the destination end device releases the token back into the network. There were no accidents with the token loop, and it was dependable, but the pace was just too slow. The popularity of planning, installing, and utilising a token ring network for use on LANs clearly did not catch

on. We had the Fiber Distributed Data Interface (FDDI) on WANs, which used token ring technologies and could run at gigabit speeds. However, the token ring will not be discussed at all in this book; it is an older invention that is not used on LANs. You would not need to know this detail for your qualification. Consider it information to have in your back pocket for interviews and dinner parties. The cool thing is that you can shift staff inside a department and have little effect on the results of the company’s efforts whether they are qualified or at least experienced about their respective fields. The same is true for networks: each layer of the OSI model has a job to do, and if one layer is changed, the other layer will continue to function normally. Let’s have a peek at the OSI model’s seven layers: Layer number Layer name Brief description

 - Application Works closest to the user, data

 - Presentation Deals with the format of the data

 - Session Keeps different applications’ data separate

 - Transport Provides reliable or unreliable delivery of information, segmentation

 - Network Provides logical addressing, which routers use to route traffic, packets

 - Data Link Deals with frames, error correction, and uses the MAC address to access media

 - Physical Deals with bits, voltage, cabling

Now that we’ve seen the OSI model, you’ll need to know each layer’s number and name for certification purposes, as well as be able to recognise or identify the job that each layer is responsible for. Let’s divide the OSI model into two sections: upper layers and lower layers. The upper layers We will see that the following three upper layers are more concerned about user engagement and how it can connect with other end devices from looking at them. So, beginning at the top and working our way down, let’s define each layer: Layer number Layer name Brief description

 - Application Works closest to the user
 - Presentation Deals with the format of the data
 - Session Keeps different applications’ data separate

### Application Layer

Since it is the gui between an individual programme and the next layer down, this layer is the most accessible to the consumer. Because of its name, this layer is sometimes misunderstood. It does not imply that a programme, such as Internet Explorer or Microsoft Word, resides at that layer; however, it is the gui that enables the user to communicate with it. Download Cisco Webex and how to use it As a result, we must remember the protocols that operate on this layer for certification purposes: HTTP, HTTPS, FTP, TFTP, SNMP, DNS, POP, IMAP, TELNET, and any network infrastructure that requires connectivity over a wide network.


### Presentation Layer

The layer of presentation The aim of this layer is straightforward: it is in charge of data translation and code formatting. When applications send data, it is encoded in a certain format, such as ASCII. When the data arrives at its destination, it must recognise this format, be able to decipher the ASCII, and display it to the Application layer so that the consumer may access it. Full-duplex contact is two-way communication, similar to a chat you might hold with a person or on the internet. Half-duplex, on the other hand, is similar to a walkie-talkie in that you speak first, then listen. As a result, you can submit or receive at any time. The lower layers The following layers describe how data can be transferred from the source to the destination, in simple terms:

 - Transport Provides reliable or unreliable delivery of information, segmentation
 - Network Provides logical addressing, which routers use to route traffic, packets
 - Data Link Deals with frames, error correction, and uses a MAC address to access media
 - Physical Deals with bits, voltage, cabling

The OSI paradigm has now been well understood. We can see the general image of what they are attempting to do by splitting them up into two pieces. However, we must delve further and dissect the OSI into its various layers.

### Transport layer

The Transport layer is responsible for transporting data.This is the layer on which data is segmented and reassembled. All data from the Application layer is combined into a concise data source by services that live on this layer. The Transmission Control Protocol (TCP) and the User Datagram Protocol (UDP). As opposed to UDP is regarded as the connection-oriented protocol, which implies it can provide secure communication.

Let’s start by defining Connection-Oriented Communication. We have something called a three-way handshake in secure transmission. The source sends a SYN packet to the receiver as part of the procedure. If the recipient is ready, it will respond with a SYN/ACK, and if the sender responds with an ACK, contact and data transmission will take place.

Keep in mind that the topologies and internetworking devices play a big role in this. Everyone is bound to a central system in a Star topology. When you connect to a hub, you’re in a mutual collision domain that runs at half-duplex and uses the CSMA/CD access process. The following are characteristics of services that are called relationship oriented:

- Three-way handshake Uses sequencing
- Uses ACKs Uses flow control

### Windowing

Windowing is the method of determining how many data the recipient may carry in a single line. As a result, when configuring routers or some other layer-three unit, we must be extremely cautious when entering IP addresses and subnet masks on the interface. Make sure you enter the network addresses you’re specifically linked to before configuring some routing protocol. Routers will often select the shortest route to a destination depending on the metric; this will decide the packet’s path to the destination. Let’s describe a few of the words we’ll be using:

- Routed protocols: These are the protocols that sit on an interface, such as IPv4 and IPv6. These protocols will have a subnetted scheme, so data can be routed by a routing protocol that chooses the appropriate network.
- Routing protocols: These are the components that create the routing table based on their algorithm, which will use the routed protocol’s IP information to obtain the network address, and then route protocols to the correct destination.
- Metric: This is a measurement of how far the destination is from the source; depending on the routing protocol in use, it will use the shortest metric to get to the destination.

### The Data Link layer

The Data Link layer is a sublayer of the Data Link layer.This layer performs flow management, physical network topology, and error notification, as well as providing physical information transmission. Each message is converted into a data frame at the Data Link layer, which contains personalised details such as the source and destination hardware addresses. At layer two, the Data Link layer does not do any routing. To get from source to destination inside the same segment, it merely uses the physical addresses of the end devices. Routers are more concerned with layer-three addressing than with layer-two addressing. Be cautious with that comment, because if you’re using Ethernet, layer-two addressing becomes extremely critical to the router at this stage. There are two sublayers in the Data Link layer.

### Media Access Control layer

Depending on the technologies used, such as Contention-Based or Token-Passing, packets are put on the media in this sub layer. Physical addressing, or the MAC address or burned-in address of a NIC card, is used in both the physical and logical topologies, as you already recognize. FragAttacks, a new form of Wi-Fi vulnerability, puts all handheld devices at risk. Logical Link Control (LLC) The focus shifts to discovering network protocols and then sending them along to be encapsulated. As a frame is sent, the LLC header would still inform the Data-Link layer what to do about it.

### The Physical layer

The physical coating is made up ofThis layer is in charge of transmitting bits from the source to the target, regardless of the medium. And if we tell 0’s and 1’s in principle, what we’re actually talking about are electrical impulses that are produced and transmitted through the air as a Carrier Wave; or through cabling that may require complex encoding and decoding, such as serial cables. Devices such as routers, repeaters, amplifiers, cabling, and even a modem on the network side, known as a channel service unit/data processing unit, are included in this sheet. When it comes to your certification, what you need to know about the OSI is IEEE basic knowledge.

