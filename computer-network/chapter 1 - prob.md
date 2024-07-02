
# Review question

### section 1,2 
1. What is the difference between a host and an end system? List several different types of end systems. Is a Web server an end system?
	- host == end system . it is false statement
	- mobile device, Internet of Things, desktop, laptops and so on
	- web server is an end systemt that provide a service for requests from its client.


2. Describe the protocol that might be used by two people having a telephonic conversation to initiate and end the conversation, i.e., the way that they talk.
	- caller try to make a phone call
	- callee got the call request and response it by tapping to accept button
	- caller got the response from callee and a connection between caller and callee established.
	- they start to talk to each other

3. Why are standards important for protocols?
	To achieve Interoperability between different systems

4. List four access technologies. Classify each one as home access, enterprise access, or wide-area wireless access.
	- DSL / cable / satellite access technologies / FTTH
	- DSL , cable : home access 
	- satellite access technologies : Wide-area wireless access
	- FTTH : modern home access

5. Is HFC transmission rate dedicated or shared among users? Are collisions possible in a downstream HFC channel? Why or why not?
	- HFC is shared among users
	- All packets coming from head-end which is a single source.
	  By demultiplexing it into several links, there is no collision in a downstream

6. What access network technologies would be most suitable for providing internet access in rural areas?
	Satellite access network technology

7. Dial-up modems and DSL both use the telephone line (a twisted-pair copper cable) as their transmission medium. Why then is DSL much faster than dialup access?
	- different bandwidth
	- always on connection
	- asymmetric speeds for downstream and upstream

8. What are some of the physical media that Ethernet can run over?
	- twisted pair copper wire 
	- fiber optic
	- air medium for wireless

9. HFC, DSL, and FTTH are all used for residential access. For each of these access technologies, provide a range of transmission rates and comment on whether the transmission rate is shared or dedicated.
	- ADSL : up to 24Mbsp for down stream and 2.5Mbps for upstream . dedicated
	- HFC : up to 42.8Mbps downstream, 30.7Mbps upstream, shared
	- FTTH : 10Gbps


### section 1.3

- R11. Suppose there is exactly one packet switch between a sending host and a receiving host. The transmission rates between the sending host and the switch and between the switch and the receiving host are R1 and R2 , respectively. Assuming that the switch uses store-and-forward packet switching, what is the total end-to-end delay to send a packet of length L? (Ignore queuing, propagation delay, and processing delay.)
	- ==L/R1 + L/R2==

- R12. What advantage does a circuit-switched network have over a packet-switched network? What advantages does TDM have over FDM in a circuit-switched network? 
	- better service quality thanks to exclusive circuit reservation
	- If we use TDM, we can share the channel in terms of time domain.
	  It means that a single user can utilize maximum bandwidth when it is his/her allocated time slice.

- R13. Suppose users share a 2 Mbps link. Also suppose each user transmits continuously at 1 Mbps when transmitting, but each user transmits only 20 percent of the time. (See the discussion of statistical multiplexing in Section 1.3 .) 
	a. When circuit switching is used, how many users can be supported? 
		two users
	b. For the remainder of this problem, suppose packet switching is used. Why will there be essentially no queuing delay before the link if two or fewer users transmit at the same time? Why will there be a queuing delay if three users transmit at the same time? 
		If there are only two users, the average arrival rate of packets doesn't exceed bandwidth of link, it doesn't requires multiplexing
	c. Find the probability that a given user is transmitting.
		P(active|user) = 0.2
	d. Suppose now there are three users. Find the probability that at any given time, all three users are transmitting simultaneously. Find the fraction of time during which the queue grows. 
		$_3C_{3}(0.2)^{3}= 0.008$

- R14. Why will two ISPs at the same level of the hierarchy often peer with each other? How does an IXP earn money?
	==Ans)==
	- The cost that lower ISP need to pay for the service from Upper level ISP is depending on the traffics of the network. To reduce this cost, different peers on the same layer do peering.
	- IXP provides a place for multiple peering and the ISPs use the IXP pay for it.
- R15. Some content providers have created their own networks. Describe Google’s network. What motivates content providers to create these networks?
	==Ans)== To provide better quality of their service to their users by separating  traffics from public network.
	Their data centers are located in lower tier ISPs

### section 1.4
- R16. Consider sending a packet from a source host to a destination host over a fixed route. List the delay components in the end-to-end delay. Which of these delays are constant and which are variable? 
	- processing delay : nearly constant, depending on computing power
	- propagation delay : constant , depending on physical medium
	- transmission delay : constant
	- queueing delay : variable, depending on traffic.
- R17. Visit the Transmission Versus Propagation Delay applet at the companion Web site. Among the rates, propagation delay, and packet sizes available, find a combination for which the sender finishes transmitting before the first bit of the packet reaches the receiver. Find another combination for which the first bit of the packet reaches the receiver before the sender finishes transmitting. 

- R18. A user can directly connect to a server through either long-range wireless or a twisted-pair cable for transmitting a 1500-bytes file. The transmission rates of the wireless and wired media are 2 and 100 Mbps, respectively. Assume that the propagation speed in air is 3 * 108 m/s, while the speed in the twisted pair is 2 * 108 m/s. If the user is located 1 km away from the server, what is the nodal delay when using each of the two technologies?
	- 17ms for wired
	
- Suppose Host A wants to send a large file to Host B. The path from Host A to Host B has three links, of rates R1 = 500 kbps, R2 = 2 Mbps, and R3 = 1 Mbps.
	  a. Assuming no other traffic in the network, what is the throughput for the file transfer? 
	  $$min(500kbps, 2Mbps, 1Mbps) = 500kbps$$
	  b. Suppose the file is 4 million bytes. Dividing the file size by the 
	  throughput, roughly how long will it take to transfer the file to Host B? 
		  $\frac{32MB}{500kbps}= 64s$ 
	  c. Repeat (a) and (b), but now with R2 reduced to 100 kbps.
- R20. Suppose end system A wants to send a large file to end system B. At a very high level, describe how end system A creates packets from the file. When one of these packets arrives to a router, what information in the packet does the router use to determine the link onto which the packet is forwarded? Why is packet switching in the Internet analogous to driving from one city to another and asking directions along the way? 
	By following application, transport, link and physical layer protocol, from the top level protocol to lowest level protocolk
- R21. Visit the Queuing and Loss applet at the companion Web site. What is the maximum emission rate and the minimum transmission rate? With those rates, what is the traffic intensity? Run the applet with these rates and determine how long it takes for packet loss to occur. Then repeat the experiment a second time and determine again how long it takes for packet loss to occur. Are the values different? Why or why not?
### section 1.5

- If two end-systems are connected through multiple routers and the data-link level between them ensures reliable data delivery, is a transport protocol offering reliable data delivery between these two end-systems necessary? Why?
	- yes, because there is no guarantee that all data-link protocol offers reliable delivery, transport layer should support reliable data delivery by following end to end principle.
- R23. What are the five layers in the Internet protocol stack? What are the principal responsibilities of each of these layers? 
	- Application : define a protocol between two end-system application
	- transport : provide a service that sends segments from client application to server application
	- ip : a responsibility to route a datagram from one host to others
	- link : packet switching between source node and destination node
	- physical : move a bit from current node to adjacent network component
- R24. What is an application-layer message? A transport-layer segment? A network-layer datagram? A link-layer frame? 
	- each of the words is a packet format related to each of the layers.
	  from the application layer to link layer frame, each layer packet has their body and header.
- R25. Which layers in the Internet protocol stack does a router process? Which layers does a link-layer switch process? Which layers does a host process?
	- router process network, link and physical layers.
	- link layer switch process link
	- host process all five layers.

### section 1.6
- R26. What is the difference between a virus and a worm? 
- R27. Describe how a botnet can be created and how it can be used for a DDoS attack. 
- R28. Suppose Alice and Bob are sending packets to each other over a computer network. Suppose Trudy positions herself in the network so that she can capture all the packets sent by Alice and send whatever she wants to Bob; she can also capture all the packets sent by Bob and send whatever she wants to Alice. List some of the malicious things Trudy can do from this position.

# Problems

1. Equation 1.1 gives a formula for the end-to-end delay of sending one packet of length L over N links of transmission rate R. Generalize this formula for sending P such packets back-to-back over the N links.
	$N\frac{L}{R} + (P-1)\frac{L}{R}$  
	(p-1)L/R is tranmission delay

2. Consider an application that transmits data at a steady rate (for example, the sender generates an N-bit unit of data every k time units, where k is small and fixed). Also, when such an application starts, it will continue running for a relatively long period of time. Answer the following questions, briefly justifying your answer: 
	a. Would a packet-switched network or a circuit-switched network be more appropriate for this application? Why? 
		circuite switched network because it ensures that user always use the bandwidth.
	b. Suppose that a packet-switched network is used and the only traffic in this network comes from such applications as described above. Furthermore, assume that the sum of the application data rates is less than the capacities of each and every link. Is some form of congestion control needed? Why?
	No, because application rates will be always less than the capacities. The network never be congested.

3. P4. Consider the circuit-switched network in Figure 1.13 . Recall that there are 4 circuits on each link. Label the four switches A, B, C, and D, going in the clockwise direction. 
	a. What is the maximum number of simultaneous connections that can be in progress at any one time in this network?  
		16 connections for each pairs
	b. Suppose that all connections are between switches A and C. What is the maximum number of simultaneous connections that can be in progress? 
		8 connections
	c. Suppose we want to make four connections between switches A and C, and another four connections between switches B and D. Can we route these calls through the four links to accommodate all eight connections? 
		yes , by assigning two connections for each pair

4. P5. Review the car-caravan analogy in Section 1.4 . Assume a propagation speed of 100 km/hour. 
   a. Suppose the caravan travels 150 km, beginning in front of one tollbooth, passing through a second tollbooth, and finishing just after a third tollbooth. What is the end-to-end delay? 
   b. Repeat (a), now assuming that there are eight cars in the caravan instead of ten.