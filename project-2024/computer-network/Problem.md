
# CH3
---
R1. Suppose the network layer provides the following service. The network layer in the source host accepts a segment of maximum size 1,200 bytes and a destination host address from the transport layer. The network layer then guarantees to deliver the segment to the transport layer at the destination host. Suppose many network application processes can be running at the destination host.

 1. Design the simplest possible transport-layer protocol that will get application data to the desired process at the destination host. Assume the operating system in the destination host has assigned a 4-byte port number to each running application process. 
    Protocol name : SIMP
    SIMP provides a socket interface for both sender and receiver
    ==Sender side==
	- Except for the size of port number, the segment size should not exceed 1196bytes
	- The segment accepts destination port and address information.
	- SIMP packs application payload and segment header altogether and relay it to network layer.
	  ==Receiver side==
	  - a communication socket is blocked state until it get messages.
	  - If messages arrived at dest's network layer, the layer give the segment
	  - SIMP interpret the data from network by its segment layout and extract dest ip address and port number.
	  - Then it relay the segment to the destination.

2.   Modify this protocol so that it provides a “return address” to the destination process.
	At the receiver side protocol, Change return type of SIMP API as return address of combined version string of destination ip address and port number.

4. In your protocols, does the transport layer “have to do anything” in the core of the computer network?
   SIMP is designed following 'end to end principle.' so it doesn't need to do anything.

---

R2. Consider a planet where everyone belongs to a family of six, every family lives in its own house, each house has a unique address, and each person in a given house has a unique name. Suppose this planet has a mail service that delivers letters from source house to destination house. The mail service requires that (1) the letter be in an envelope, and that (2) the address of the destination house (and nothing more) be clearly written on the envelope. Suppose each family has a delegate family member who collects and distributes letters for the other family members. The letters do not necessarily provide any indication of the recipients of the letters. 
1.  Using the solution to Problem R1 above as inspiration, describe a protocol that the delegates can use to deliver letters from a sending family member to a receiving family member.
2. In your protocol, does the mail service ever have to open the envelope and examine the letter in order to provide its service?
