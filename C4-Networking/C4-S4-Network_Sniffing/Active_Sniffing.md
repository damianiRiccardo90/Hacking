# *__Active Sniffing__*

In a _switched network environment_, packets are only sent to the port they are destined for, according to their _destination MAC addresses_. This requires more intelligent hardware that can create and maintain a _table_ associating MAC addresses with certain ports, depending on which _device_ is connected to each _port_, as illustrated here.

The advantage of a switched environment is that devices are only sent packets that are meant for them, so that promiscuous devices aren’t able to sniff any additional packets. But even in a switched environment, there are clever ways to sniff other devices’ packets; they just tend to be a bit more complex. In order to find hacks like these, the details of the protocols must be examined and then combined.

One important aspect of network communications that can be manipulated for interesting effects is the _source address_. There’s no provision in these protocols to ensure that the source address in a packet really is the address of the source machine. The act of forging a source address in a packet is known as __spoofing__. The addition of _spoofing_ to your bag of tricks greatly increases the number of possible hacks, since most systems expect the source address to be valid.

<div align="left" width="100%">
<img src="Switched_Network.png?raw=true" alt="Switched Network" width="100%">
</div>

Spoofing is the first step in sniffing packets on a switched network. The other two interesting details are found in __ARP__. First, when an _ARP reply_ comes in with an IP address that already exists in the _ARP cache_, the receiving system will overwrite the prior MAC address information with the new information found in the reply (unless that entry in the _ARP cache_ was explicitly marked as permanent). Second, no state information about the ARP traffic is kept, since this would require additional memory and would complicate a protocol that is meant to be simple. This means systems will accept an _ARP reply_ even if they didn’t send out an _ARP request_.

These three details, when exploited properly, allow an attacker to sniff network traffic on a switched network using a technique known as __ARP redirection__. The attacker sends _spoofed ARP replies_ to certain devices that cause the _ARP cache_ entries to be overwritten with the attacker’s data. This technique is called __ARP cache poisoning__. In order to sniff network traffic between two points, _A_ and _B_, the attacker needs to poison the ARP cache of A to cause A to believe that B’s IP address is at the attacker’s MAC address, and also poison the ARP cache of B to cause B to believe that A’s IP address is also at the attacker’s MAC address. Then the attacker’s machine simply needs to forward these packets to their appropriate final destinations. After that, all of the traffic between A and B still gets delivered, but it all flows through the attacker’s machine, as shown here.

<div align="left" width="100%">
<img src="ARP_Redirection.png?raw=true" alt="ARP Redirection" width="100%">
</div>

Since _A_ and _B_ are wrapping their own Ethernet headers on their packets based on their respective _ARP caches_, A’s IP traffic meant for B is actually sent to the attacker’s MAC address, and vice versa. The switch only filters traffic based on MAC address, so the switch will work as it’s designed to, sending A’s and B’s IP traffic, destined for the attacker’s MAC address, to the attacker’s port. Then the attacker rewraps the IP packets with the proper Ethernet headers and sends them back to the switch, where they are finally routed to their proper destination. The switch works properly; it’s the victim machines that are tricked into redirecting their traffic through the attacker’s machine.

Due to _timeout_ values, the victim machines will periodically send out real _ARP requests_ and receive real _ARP replies_ in response. In order to maintain the redirection attack, the attacker must keep the victim machine’s ARP caches poisoned. A simple way to accomplish this is to send _spoofed ARP replies_ to both A and B at a constant interval—for example, every _10 seconds_.

A __gateway__ is a system that routes all the traffic from a local network out to the Internet. ARP redirection is particularly interesting when one of the victim machines is the __default gateway__, since the traffic between the _default gateway_ and another system is that system’s _Internet traffic_. For example, if a machine at _192.168.0.118_ is communicating with the gateway at _192.168.0.1_ over a switch, the traffic will be restricted by MAC address. This means that this traffic cannot normally be sniffed, even in _promiscuous mode_. In order to sniff this traffic, it must be redirected.

To redirect the traffic, first the MAC addresses of _192.168.0.118_ and _192.168.0.1_ need to be determined. This can be done by _pinging_ these hosts, since any IP connection attempt will use ARP. If you run a _sniffer_, you can see the ARP communications, but the OS will cache the resulting IP/MAC address associations.

<pre style="color: white;">
reader@hacking:~/booksrc $ ping -c 1 -w 1 192.168.0.1
PING 192.168.0.1 (192.168.0.1): 56 octets data
64 octets from 192.168.0.1: icmp_seq=0 ttl=64 time=0.4 ms
--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.4/0.4/0.4 ms
reader@hacking:~/booksrc $ ping -c 1 -w 1 192.168.0.118
PING 192.168.0.118 (192.168.0.118): 56 octets data
64 octets from 192.168.0.118: icmp_seq=0 ttl=128 time=0.4 ms
--- 192.168.0.118 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.4/0.4/0.4 ms
reader@hacking:~/booksrc $ arp -na
? (192.168.0.1) at 00:50:18:00:0F:01 [ether] on eth0
? (192.168.0.118) at 00:C0:F0:79:3D:30 [ether] on eth0
reader@hacking:~/booksrc $ ifconfig eth0
eth0    Link encap:Ethernet HWaddr 00:00:AD:D1:C7:ED
        inet addr:192.168.0.193 Bcast:192.168.0.255 Mask:255.255.255.0
        UP BROADCAST NOTRAILERS RUNNING MTU:1500 Metric:1
        RX packets:4153 errors:0 dropped:0 overruns:0 frame:0
        TX packets:3875 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:100
        RX bytes:601686 (587.5 Kb) TX bytes:288567 (281.8 Kb)
        Interrupt:9 Base address:0xc000
reader@hacking:~/booksrc $
</pre>