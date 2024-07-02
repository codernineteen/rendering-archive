
# Review
## introduction to application layer
1. Nonproprietary Internet applications
	web - HTTP
	file transfer - FTP
	remote login - Telnet
	email - SMTP
	BitTorrent - bit torrent protocol
2. Network Architectre : refers to organization of communication process into layers.
3. Application arhitecture : designed by app developer.
4. a client is a process who initialize communication session and a server is a process who are listening to new incoming session.
5. In p2p architecture , one peer do a role of client who request service and the other peer do a role of server who provide requested service.
6. In internetworking, IP address is used to identify a process running on another host.
7. It is  a componet  of network application as networking protocol between two applications related to each other.

## Transport service
1. An example of requirig both no data loss and highly time-sensitive
	live coding functionality in visual studio ide
9. Four broad services of transport protocol can provide
	- reliable data transfer 
		- TCP
	- a gurantee that a data will be arrived at a specified amout of time.
		- no support in TCP and UDP
	- a guarantee that a certain value for **throughput to be maintained**
		- no support in TCP and UDP
	- security
		- no support in TCP and UDP (by the way TLS on TCP is available option.)
10. SSL operates on application layer. it encrpyt a message from networked application and pass through it into transport layer by socket. If an application programmer wants to use the SSL protocol, he/she need to implement it in the code.

## HTTP
1. With handshake protocol, two entities exchange control packet before they starting to communicate with each other.
2. HTTP is stateless protocol which doesn't store any states from communications
3. a web site can keep track of user by cookie. When a user enter the site at the very first time, the user's identifier is stored in backend-entry. A server response agains client request with a 'Set-Cookie' header line. Then, the browser as a client store the cookie in the web storage. 
4. Yes. the body content depends on the section of 'Content-Type' specified in HTTP header.
5. In HTTP/1.1, If there is a huge size of data on the head position of the traffic, the data with small sizes need to wait for long time. HTTP2 introduces multiplexing allowing multiple requests and responses interleaved and sent over a single TCP connection.

## dns 
1. recursive and iterative DNS queries
	1. recursive DNS query relay the query to the next DNS server if the current DNS server can't resolve the query.
	2. a query sent to its local DNS server is sent to from root DNS server to authoritative DNS server.
		If a DNS server has cached information or it is authoratative server that has the right information, it answers.
		If not, it response with a useful information about the next candidates.


## socket programming
- TCP server requires two sockets because the protocol is connection-oriented by its nature, so it need one socket for listening incomming connection request and the other socket for communicating with each client
- If there are n-simultaneoous connections , TCP server needs 1 socket for listening and n sockets for communications.
- In the TCP client , it attempts to try handshaking with target server as soon as the program starts. That's why the server need to be executed before client start. On the other hand, there is no handshaking in UDP protocol and the protocol is connection-less. Because there is no connection, the client doesn't care about whether the message reaches designate server or not.

# T/F

a. A user requests a Web page that consists of some text and three images. For this page, the client will send one request message and receive four response messages.
	False, client will send 4 requests and get 4 response
b. Two distinct Web pages (for example, www.mit.edu/research .html and www.mit.edu/students.html) can be sent over the same persistent connection.
	True, persistent connection allows multiple request and response on a single connection.
c. With nonpersistent connections between browser and origin server, it is possible for a single TCP segment to carry two distinct HTTP request messages.
	False, Once a nonpersistent HTTP client and server exchange a message , they close the TCP connection.
d. The Date: header in the HTTP response message indicates when the object in the response was last modified.
	False, Data field is related to the generation time of response message itself.
	The mentioned information is specified in 'Last-Modified' section.
e. HTTP response messages never have an empty message body.
	False, GET method has empty message body.

# Problems
1. Assume you open a browser and enter http://yourbusiness.com/ about.html in the address bar. What happens until the webpage is displayed? Provide details about the protocol(s) used and a high-level description of the messages exchanged.
	UDP for DNS, TCP for HTTP, HTTP and DNS
	- DNS resolving
	A browser relay the DNS resolve query to the DNS client and the DNS client query DNS information about 'http://yourbusiness.com/ ' with iterative approach or recursive approach (root -> TLD -> authoritative)
	- TCP connection
	- HTTP request
	A HTTP message is generated with HTTP protocol the browser using with GET method and relative path to 'about.html'.
	With some of header lines and empty HTTP body, it send the generated message to server.
	- HTTP response

2. Given HTTP message
	GET /cs453/index.html HTTP/1.1
	Host: gai a.cs.umass.edu
	User-Agent: Mozilla/5.0 ( Windows;U; Windows NT 5.1; en-US; rv:1.7.2) Gec ko/20040804 Netscape/7.2 (ax) 
	Accept:ex t/xml, application/xml, application/xhtml+xml, text /html;q=0.9, text/plain;q=0.8,image/png,*/*;q=0.5
	Accept-Language: en-us,en;q=0.5AcceptEncoding: zip,deflateAccept-Charset: ISO -8859-1,utf-8;q=0.7,*;q=0.7Keep-Alive: 300 
	Connection:keep-alive

	a. What is the URL of the document requested by the browser?
		gai a.cs.umass.edu/cs453/index.html
	b. What version of HTTP is the browser running? HTTP/1.1
	c. Does the browser request a non-persistent or a persistent connection? 
		Persistent connection (Connection : keep-alive)
	d. What is the IP address of the host on which the browser is running? 
		IP address of the host running was not specified in the message
	e. What type of browser initiates this message? Why is the browser type needed in an HTTP request message?
		Mozila/5.0, It is need to be specified because different objects for different browser.

3. Given HTTP message
	HTTP/1.1 200 OK
	Date: Tue, 07 Mar 2008 12:39:45GMT
	Server: Apache/2.0.52 (Fedora) 
	Last-Modified: Sat, 10 Dec2005 18:27:46 GMTE
	Tag: ”526c3-f22-a88a4c80”
	AcceptRanges: bytes
	Content-Length: 3874 
	Keep-Alive: timeout=max=100
	Connection: Keep-Alive
	Content-Type: text/html; charset= ISO-8859-1
	
	a. Was the server able to successfully find the document or not? What time was the document reply provided? 
	- successfully found the document with status code of 200 and phrase Ok
	- Tue, 07 Mar 2008 12:39:45GMT
	b. When was the document last modified?  
	-   Sat, 10 Dec200
	c. How many bytes are there in the document being returned? 
	- 3874bytes
	d. What are the first 5 bytes of the document being returned? Did the server agree to a persistent connection?
	- "<!doc". The server agreed to a persistent connection, as indicated by the presence of both the "Keep-Alive" and "Connection" headers with values indicating a persistent connection ("timeout=max=100" and "Keep-Alive" respectively).

4. Suppose within your Web browser, you click on a link to obtain a Web page. The IP address for the associated URL is not cached in your local host, so a DNS lookup is necessary to obtain the IP address. Suppose that n DNS servers are visited before your host receives the IP address from DNS; the successive visits incur an RTT of RTT1, . . . , RTTn. Further suppose that the Web page associated with the link contains exactly one object, consisting of a large amount of HTML text. Let RTT0 denote the RTT between the local host and the server containing the object. Assuming transmission duration of 0.002 * RTT0 of the object, how much time elapses from when the client clicks on the link until the client receives the object?
	transmission duration : 0.002 * RTT0 
	n DNS server visits : RTT1,..., RTTn
	Web page has only an object with HTML text
	
	time elapses : $\sum_{n}^{i=1}RTT_{i} + 2RTT_{0} + 0.002 \times RTT_0$  

5.  Consider Problem P7 again and assume RTT0 = RTT1 = RTT2 = . . . RTTn = RTT, Furthermore, assume a new HTML file, small enough to have negligible transmission time, which references nine equally small objects on the same server. How much time elapses with 
	a. non-persistent HTTP with no parallel TCP connections? 
		$n\times RTT + 2RTT \times 9 = (n+18)RTT$
	b. non-persistent HTTP with the browser configured for 6 parallel connections? 
		$n\times RTT + 2RTT \times 2$
	c. persistent HTTP?
		$n \times RTT + RTT + 9RTT$

9. Consider Figure 2.12, for which there is an institutional network connected to the Internet. Moreover, assume the access link has been upgraded to 54 Mbps, and the institutional LAN is upgraded to 10 Gbps. Suppose that the average object size is 1,600,000 bits and that the average request rate from the institution’s browsers to the origin servers is 24 requests per second. Also suppose that the amount of time it takes from when the router on the Internet side of the access link forwards an HTTP request until it receives the response is three seconds on average (see Section 2.2.5). Model the total average response time as the sum of the average access delay (that is, the delay from Internet router to institution router) and the average Internet delay. For the average access delay, use ∆/(1 - ∆b), where ∆ is the average time required to send an object over the access link and b is the arrival rate of objects to the access link.
10. ![](../../images/Pasted%20image%2020240422214525.png)
	a. Find the total average response time. 
		$$\frac{16M/54Mbps}{1-24*(16Mb/10000Mbps)}$$
	b. Now suppose a cache is installed in the institutional LAN. Suppose the miss rate is 0.3. Find the total response time.
