# *__Transport Layer__*

The transport layer can be thought of as the first line of office receptionists, picking up the mail from the network layer. If a customer wants to return a effective piece of merchandise, they send a message requesting a _Return Material Authorization_ (__RMA__) number. Then the receptionist would follow the return protocol by asking for a receipt and eventually issuing an RMA number so the customer can mail the product in. The post office is only concerned with sending these messages (and packages) back and forth, not with what’s in them.

The two major protocols at this layer are the _Transmission Control Protocol_ (__TCP__) and _User Datagram Protocol_ (__UDP__). _TCP_ is the most commonly used protocol for services on the Internet: _telnet_, _HTTP_ (web traffic), _SMTP_ (email traffic), and _FTP_ (file transfers) all use _TCP_. One of the reasons for TCP’s popularity is that it provides a transparent, yet reliable and bidirectional, connection between two IP addresses. Stream sockets use _TCP/IP_ connections. A bidirectional connection with TCP is similar to using a telephone—after dialing a number, a connection is made through which both parties can communicate. Reliability simply means that TCP will ensure that all the data will reach its destination in the proper order. If the packets of a connection get jumbled up and arrive out of order, TCP will make sure they’re put back in order before handing the data up to the next layer. If some packets in the middle of a connection are lost, the destination will hold on to the packets it has while the source retransmits the missing packets.

All of this functionality is made possible by a set of flags, called _TCP flags_, and by tracking values called sequence numbers. The TCP flags are as follows:

<div align="left" width="100%">
<img src="TCP_Flags.png?raw=true" alt="TCP Flags" width="100%">
</div>

These flags are stored in the _TCP header_ along with the _source and destination ports_. The TCP header is specified in _RFC 793_.

__From RFC 793__

<pre style="color: white;">
[Page 14]

September 1981

                                          Transmission Control Protocol

                      3. FUNCTIONAL SPECIFICATION

3.1. Header Format

  TCP segments are sent as internet datagrams. The Internet Protocol
  header carries several information fields, including the source and
  destination host addresses [2]. A TCP header follows the internet
  header, supplying information specific to the TCP protocol. This
  division allows for the existence of host level protocols other than
  TCP.

  TCP Header Format

   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |          Source Port          |       Destination Port        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                        Sequence Number                        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                    Acknowledgment Number                      |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |  Data |           |U|A|P|R|S|F|                               |
  | Offset|  Reserved |R|C|S|S|Y|I|            Window             |
  |       |           |G|K|H|T|N|N|                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |           Checksum            |         Urgent Pointer        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                    Options                    |    Padding    |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                             data                              |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                           TCP Header Format

         Note that one tick mark represents one bit position.

                              Figure 3.
</pre>

The _sequence number_ and _acknowledgment number_ are used to maintain state. The __SYN__ and __ACK__ flags are used together to open connections in a _three-step handshaking_ process. When a client wants to open a connection with a server, a packet with the __SYN__ flag on, but the __ACK__ flag off, is sent to the server. The server then responds with a packet that has both the __SYN__ and __ACK__ flags turned on. To complete the connection, the client sends back a packet with the __SYN__ flag off but the __ACK__ flag on. After that, every packet in the connection will have the __ACK__ flag turned on and the __SYN__ flag turned off. Only the first two packets of the connection have the __SYN__ flag on, since those packets are used to synchronize sequence numbers.

<div align="left" width="100%">
<img src="Three_Way_Handshake.png?raw=true" alt="Three Way Handshake" width="70%">
</div>

Sequence numbers allow _TCP_ to put unordered packets back into order, to determine whether packets are missing, and to prevent mixing up packets from other connections.

When a connection is initiated, each side generates an _initial sequence number_. This number is communicated to the other side in the first two __SYN__ packets of the connection handshake. Then, with each packet that is sent, the sequence number is incremented by the number of bytes found in the data portion of the packet. This sequence number is included in the _TCP packet header_. In addition, each TCP header has an _acknowledgment number_, which is simply the other side’s sequence number plus one.

_TCP_ is great for applications where reliability and bidirectional communication are needed. However, the cost of this functionality is paid in communication overhead.

_UDP_ has much less overhead and built-in functionality than _TCP_. This lack of functionality makes it behave much like the _IP protocol_: It is connection-less and unreliable. Without built-in functionality to create connections and maintain reliability, _UDP_ is an alternative that expects the application to deal with these issues. Sometimes connections aren’t needed, and the lightweight _UDP_ is a much better protocol for these situations. The _UDP header_, defined in _RFC 768_, is relatively tiny. It only contains four _16-bit_ values in this order: _source port_, _destination port_, _length_, and _checksum_.