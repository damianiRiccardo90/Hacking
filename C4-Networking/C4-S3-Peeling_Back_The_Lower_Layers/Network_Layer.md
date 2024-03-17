# *__Network Layer__*

The network layer is like a worldwide postal service providing an addressing and delivery method used to send things everywhere. The protocol used at this layer for Internet addressing and delivery is, appropriately, called _Internet Protocol_ (__IP__); the majority of the Internet uses _IP version 4_.

Every system on the Internet has an _IP address_, consisting of a familiar _four-byte_ arrangement in the form of _xx.xx.xx.xx_. The _IP header_ for packets in this layer is _20 bytes_ in size and consists of various fields and bit-flags as defined in _RFC 791_.

__From RFC 791__

<pre style="color: white;">
[Page 10]

September 1981

                                                    Internet Protocol

                          3. SPECIFICATION

3.1. Internet Header Format

  A summary of the contents of the internet header follows:

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|         Total Length          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                    
                    Example Internet Datagram Header

                               Figure 4.
Note that each tick mark represents one bit position.
</pre>

This surprisingly descriptive ASCII diagram shows these fields and their positions in the header. Standard protocols have awesome documentation. Similar to the Ethernet header, the _IP header_ also has a protocol field to describe the type of data in the packet and the _source and destination addresses_ for routing. In addition, the header carries a _checksum_, to help detect transmission errors, and fields to deal with _packet fragmentation_.

The Internet Protocol is mostly used to transmit packets wrapped in higher layers. However, _Internet Control Message Protocol_ (__ICMP__) packets also exist on this layer. _ICMP packets_ are used for messaging and diagnostics. IP is less reliable than the post office—there’s no guarantee that an IP packet will actually reach its final destination. If there’s a problem, an ICMP packet is sent back to notify the sender of the problem.

_ICMP_ is also commonly used to test for connectivity. _ICMP Echo Request_ and _Echo Reply messages_ are used by a utility called `ping`. If one host wants to test whether it can route traffic to another host, it pings the remote host by sending an _ICMP Echo Request_. Upon receipt of the _ICMP Echo Request_, the remote host sends back an _ICMP Echo Reply_. These messages can be used to determine the connection latency between the two hosts. However, it is important to remember that ICMP and IP are both connection-less; all this protocol layer really cares about is getting the packet to its destination address.

Sometimes a network link will have a limitation on packet size, disallowing the transfer of large packets. IP can deal with this situation by fragmenting packets, as shown here.

<div align="left" width="100%">
<img src="IP_Packet_Fragmentation.png?raw=true" alt="IP Packet Fragmentation" width="70%">
</div>

The packet is broken up into smaller packet fragments that can pass through the network link, IP headers are put on each fragment, and they’re sent off. Each fragment has a different fragment offset value, which is stored in the header. When the destination receives these fragments, the offset values are used to reassemble the original IP packet.

Provisions such as fragmentation aid in the delivery of IP packets, but this does nothing to maintain connections or ensure delivery. This is the job of the protocols at the transport layer.