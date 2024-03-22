# *__Decoding the Layers__*

In our packet captures, the outermost layer is _Ethernet_, which is also the lowest visible layer. This layer is used to send data between Ethernet endpoints with _MAC addresses_. The header for this layer contains the source MAC address, the destination MAC address, and a _16-bit_ value that describes the type of Ethernet packet. On Linux, the structure for this header is defined in _/usr/include/linux/if_ethernet.h_ and the structures for the IP header and TCP header are located in _/usr/include/netinet/ip.h_ and _/usr/include/netinet/tcp.h_, respectively. The source code for `tcpdump` also has structures for these headers, or we could just create our own header structures based on the RFCs. A better understanding can be gained from writing our own structures, so let’s use the structure definitions as guidance to create our own packet header structures to include in _hacking-network.h_.

First, let’s look at the existing definition of the Ethernet header.

__From /usr/include/if_ether.h__

```c
#define ETH_ALEN 6 /* Octets in one ethernet addr */
#define ETH_HLEN 14 /* Total octets in header */

/*
 * This is an Ethernet frame header.
 */
struct ethhdr 
{
    unsigned char h_dest[ETH_ALEN]; /* Destination eth addr */
    unsigned char h_source[ETH_ALEN]; /* Source ether addr */
    __be16 h_proto; /* Packet type ID field */
} __attribute__((packed));
```

This structure contains the three elements of an _Ethernet header_. The variable declaration of ____be16__ turns out to be a type definition for a _16-bit unsigned short integer_. This can be determined by recursively grepping for the type definition in the include files.

<pre style="color: white;">
reader@hacking:~/booksrc $
$ grep -R "typedef.*__be16" /usr/include
<strong><em>/usr/include/linux/types.h:typedef __u16 __bitwise __be16;</em></strong>

$ grep -R "typedef.*__u16" /usr/include | grep short
/usr/include/linux/i2o-dev.h:typedef unsigned short __u16;
<strong><em>/usr/include/linux/cramfs_fs.h:typedef unsigned short __u16;</em></strong>
/usr/include/asm/types.h:typedef unsigned short __u16;
$
</pre>

The include file also defines the Ethernet header length in __ETH_HLEN__ as _14 bytes_. This adds up, since the source and destination MAC addresses use _6 bytes_ each, and the packet type field is a _16-bit_ short integer that takes up _2 bytes_. However, many compilers will pad structures along _4-byte_ boundaries for alignment, which means that __sizeof(struct ethhdr)__ would return an incorrect size. To avoid this, __ETH_HLEN__ or a fixed value of _14 bytes_ should be used for the Ethernet header length.

By including _<linux/if_ether.h>_, these other include files containing the required ____be16__ type definition are also included. Since we want to make our own structures for _hacking-network.h_, we should strip out references to unknown type definitions. While we’re at it, let’s give these fields better names.

__Added to hacking-network.h__

```c
#define ETHER_ADDR_LEN 6
#define ETHER_HDR_LEN 14

struct ether_hdr 
{
    unsigned char ether_dest_addr[ETHER_ADDR_LEN]; // Destination MAC address
    unsigned char ether_src_addr[ETHER_ADDR_LEN]; // Source MAC address
    unsigned short ether_type; // Type of Ethernet packet
};
```

We can do the same thing with the _IP_ and _TCP_ structures, using the corresponding structures and _RFC_ diagrams as a reference.

__From /usr/include/netinet/ip.h__

```c
struct iphdr
{
#if __BYTE_ORDER == __LITTLE_ENDIAN
    unsigned int ihl:4;
    unsigned int version:4;
#elif __BYTE_ORDER == __BIG_ENDIAN
    unsigned int version:4;
    unsigned int ihl:4;
#else
# error "Please fix <bits/endian.h>"
#endif
    u_int8_t tos;
    u_int16_t tot_len;
    u_int16_t id;
    u_int16_t frag_off;
    u_int8_t ttl;
    u_int8_t protocol;
    u_int16_t check;
    u_int32_t saddr;
    u_int32_t daddr;
    /*The options start here. */
};
```

__From RFC 791__

<pre style="color: white;">
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
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
</pre>

Each element in the structure corresponds to the fields shown in the _RFC header diagram_. Since the first two fields, _Version_ and _IHL_ (_Internet Header Length_) are only _four bits_ in size and there aren’t any 4-bit variable types in C, the Linux header definition splits the byte differently depending on the byte order of the host. These fields are in the network byte order, so, if the host is little-endian, the _IHL_ should come before _Version_ since the byte order is reversed. For our purposes, we won’t really be using either of these fields, so we don’t even need to split up the byte.

__Added to hacking-network.h__

```c
struct ip_hdr 
{
    unsigned char ip_version_and_header_length; // Version and header length
    unsigned char ip_tos;          // Type of service
    unsigned short ip_len;         // Total length
    unsigned short ip_id;          // Identification number
    unsigned short ip_frag_offset; // Fragment offset and flags
    unsigned char ip_ttl;          // Time to live
    unsigned char ip_type;         // Protocol type
    unsigned short ip_checksum;    // Checksum
    unsigned int ip_src_addr;      // Source IP address
    unsigned int ip_dest_addr;     // Destination IP address
};
```

The compiler padding, as mentioned earlier, will align this structure on a _4-byte_ boundary by padding the rest of the structure. IP headers are always _20 bytes_.

For the _TCP packet header_, we reference _/usr/include/netinet/tcp.h_ for the structure and _RFC 793_ for the header diagram.

__From /usr/include/netinet/tcp.h__

```c
typedef u_int32_t tcp_seq;

/*
 * TCP header.
 * Per RFC 793, September, 1981.
 */
struct tcphdr
{
    u_int16_t th_sport; /* source port */
    u_int16_t th_dport; /* destination port */
    tcp_seq th_seq; /* sequence number */
    tcp_seq th_ack; /* acknowledgment number */
# if __BYTE_ORDER == __LITTLE_ENDIAN
    u_int8_t th_x2:4; /* (unused) */
    u_int8_t th_off:4; /* data offset */
# endif
# if __BYTE_ORDER == __BIG_ENDIAN
    u_int8_t th_off:4; /* data offset */
    u_int8_t th_x2:4; /* (unused) */
# endif
    u_int8_t th_flags;
# define TH_FIN 0x01
# define TH_SYN 0x02
# define TH_RST 0x04
# define TH_PUSH 0x08
# define TH_ACK 0x10
# define TH_URG 0x20
    u_int16_t th_win; /* window */
    u_int16_t th_sum; /* checksum */
    u_int16_t th_urp; /* urgent pointer */
};
```

__From RFC 793__

<pre style="color: white;">
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
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Data Offset: 4 bits
    The number of 32 bit words in the TCP Header. This indicates where
    the data begins. The TCP header (even one including options) is an
    integral number of 32 bits long.
Reserved: 6 bits
    Reserved for future use. Must be zero.
Options: variable
</pre>

Linux’s __tcphdr__ structure also switches the ordering of the 4-bit _data offset field_ and the 4-bit section of the reserved field depending on the host’s byte order. The data offset field is important, since it tells the size of the variable length TCP header. You might have noticed that Linux’s __tcphdr__ structure doesn’t save any space for _TCP options_. This is because the _RFC_ defines this field as optional. The size of the TCP header will always be _32-bit-aligned_, and the data offset tells us how many _32-bit words_ are in the header. So the TCP header size in bytes equals the data offset field from the header times _four_. Since the data offset field is required to calculate the header size, we’ll split the byte containing it, assuming _little-endian_ host byte ordering.

The __th_flags__ field of Linux’s __tcphdr__ structure is defined as an _8-bit unsigned character_. The values defined below this field are the _bitmasks_ that correspond to the six possible flags.

__Added to hacking-network.h__

```c
struct tcp_hdr 
{
    unsigned short tcp_src_port;  // Source TCP port
    unsigned short tcp_dest_port; // Destination TCP port
    unsigned int tcp_seq;         // TCP sequence number
    unsigned int tcp_ack;         // TCP acknowledgment number
    unsigned char reserved:4;     // 4 bits from the 6 bits of reserved space
    unsigned char tcp_offset:4;   // TCP data offset for little-endian host
    unsigned char tcp_flags;      // TCP flags (and 2 bits from reserved space)
#define TCP_FIN 0x01
#define TCP_SYN 0x02
#define TCP_RST 0x04
#define TCP_PUSH 0x08
#define TCP_ACK 0x10
#define TCP_URG 0x20
    unsigned short tcp_window;    // TCP window size
    unsigned short tcp_checksum;  // TCP checksum
    unsigned short tcp_urgent;    // TCP urgent pointer
};
```

Now that the headers are defined as structures, we can write a program to decode the layered headers of each packet. But before we do, let’s talk about __libpcap__ for a moment. This library has a function called _pcap_loop()_, which is a better way to capture packets than just looping on a _pcap_next()_ call. Very few programs actually use _pcap_next()_, because it’s clumsy and inefficient. The _pcap_loop()_ function uses a _callback function_. This means the _pcap_loop()_ function is passed a function pointer, which is called every time a packet is captured. The prototype for _pcap_loop()_ is as follows:

```c
int pcap_loop(pcap_t* handle, int count, pcap_handler callback, u_char* args);
```

The first argument is the _pcap_’s handle, the next one is a count of how many packets to capture, and the third is a function pointer to the callback function. If the count argument is set to -1, it will loop until the program breaks out of it. The final argument is an optional pointer that will get passed to the callback function. Naturally, the callback function needs to follow a certain prototype, since _pcap_loop()_ must call this function. The callback function can be named whatever you like, but the arguments must be as follows:

```c
void callback(u_char* args, const struct pcap_pkthdr* cap_header, const u_char* packet);
```

The first argument is just the optional argument pointer from the last argument to _pcap_loop()_. It can be used to pass additional information to the callback function, but we aren’t going to be using this. The next two arguments should be familiar from _pcap_next()_: a pointer to the capture header and a pointer to the packet itself.

The following example code uses _pcap_loop()_ with a callback function to capture packets and our header structures to decode them. This program will be explained as the code is listed.

__decode_sniff.c__

```c
#include <pcap.h>
#include "hacking.h"
#include "hacking-network.h"

void pcap_fatal(const char*, const char*);
void decode_ethernet(const u_char*);
void decode_ip(const u_char*);
u_int decode_tcp(const u_char*);
void caught_packet(u_char*, const struct pcap_pkthdr*, const u_char*);

int main() 
{
    struct pcap_pkthdr cap_header;
    const u_char* packet, *pkt_data;
    char errbuf[PCAP_ERRBUF_SIZE];
    char* device;

    pcap_t* pcap_handle;

    device = pcap_lookupdev(errbuf);
    if (device == NULL)
        pcap_fatal("pcap_lookupdev", errbuf);

    printf("Sniffing on device %s\n", device);

    pcap_handle = pcap_open_live(device, 4096, 1, 0, errbuf);
    if (pcap_handle == NULL)
        pcap_fatal("pcap_open_live", errbuf);

    pcap_loop(pcap_handle, 3, caught_packet, NULL);

    pcap_close(pcap_handle);
}
```

At the beginning of this program, the prototype for the callback function, called _caught_packet()_, is declared along with several decoding functions. Everything else in _main()_ is basically the same, except that the for loop has been replaced with a single call to _pcap_loop()_. This function is passed the __pcap_handle__, told to capture three packets, and pointed to the callback function, _caught_packet()_. The final argument is __NULL__, since we don’t have any additional data to pass along to _caught_packet()_. Also, notice that the _decode_tcp()_ function returns a _u_int_. Since the TCP header length is variable, this function returns the length of the TCP header.

```c
void caught_packet(u_char* user_args, const struct pcap_pkthdr* cap_header, const u_char* packet) 
{
    int tcp_header_length, total_header_size, pkt_data_len;
    u_char* pkt_data;

    printf("==== Got a %d byte packet ====\n", cap_header->len);

    decode_ethernet(packet);
    decode_ip(packet + ETHER_HDR_LEN);
    tcp_header_length = decode_tcp(packet + ETHER_HDR_LEN + sizeof(struct ip_hdr));

    total_header_size = ETHER_HDR_LEN + sizeof(struct ip_hdr) + tcp_header_length;
    pkt_data = (u_char*)packet + total_header_size; // pkt_data points to the data portion.
    pkt_data_len = cap_header->len - total_header_size;
    if (pkt_data_len > 0) 
    {
        printf("\t\t\t%u bytes of packet data\n", pkt_data_len);
        dump(pkt_data, pkt_data_len);
    } 
    else
        printf("\t\t\tNo Packet Data\n");
}

void pcap_fatal(const char* failed_in, const char* errbuf) 
{
    printf("Fatal Error in %s: %s\n", failed_in, errbuf);
    exit(1);
}
```

The _caught_packet()_ function gets called whenever _pcap_loop()_ captures a packet. This function uses the header lengths to split the packet up by layers and the decoding functions to print out details of each layer’s header.

```c
void decode_ethernet(const u_char* header_start) 
{
    int i;
    const struct ether_hdr* ethernet_header;
    
    ethernet_header = (const struct ether_hdr*)header_start;
    printf("[[ Layer 2 :: Ethernet Header ]]\n");
    printf("[ Source: %02x", ethernet_header->ether_src_addr[0]);
    for (i = 1; i < ETHER_ADDR_LEN; i++)
        printf(":%02x", ethernet_header->ether_src_addr[i]);

    printf("\tDest: %02x", ethernet_header->ether_dest_addr[0]);
    for (i = 1; i < ETHER_ADDR_LEN; i++)
        printf(":%02x", ethernet_header->ether_dest_addr[i]);
    printf("\tType: %hu ]\n", ethernet_header->ether_type);
}

void decode_ip(const u_char* header_start) 
{
    const struct ip_hdr* ip_header;
    
    ip_header = (const struct ip_hdr*)header_start;
    printf("\t(( Layer 3 ::: IP Header ))\n");
    printf("\t( Source: %s\t", inet_ntoa(ip_header->ip_src_addr));
    printf("Dest: %s )\n", inet_ntoa(ip_header->ip_dest_addr));
    printf("\t( Type: %u\t", (u_int) ip_header->ip_type);
    printf("ID: %hu\tLength: %hu )\n", ntohs(ip_header->ip_id), ntohs(ip_header->ip_len));
}

u_int decode_tcp(const u_char* header_start) 
{
    u_int header_size;
    const struct tcp_hdr* tcp_header;

    tcp_header = (const struct tcp_hdr*)header_start;
    header_size = 4 * tcp_header->tcp_offset;

    printf("\t\t{{ Layer 4 :::: TCP Header }}\n");
    printf("\t\t{ Src Port: %hu\t", ntohs(tcp_header->tcp_src_port));
    printf("Dest Port: %hu }\n", ntohs(tcp_header->tcp_dest_port));
    printf("\t\t{ Seq #: %u\t", ntohl(tcp_header->tcp_seq));
    printf("Ack #: %u }\n", ntohl(tcp_header->tcp_ack));
    printf("\t\t{ Header Size: %u\tFlags: ", header_size);
    if (tcp_header->tcp_flags & TCP_FIN)
        printf("FIN ");
    if (tcp_header->tcp_flags & TCP_SYN)
        printf("SYN ");
    if (tcp_header->tcp_flags & TCP_RST)
        printf("RST ");
    if (tcp_header->tcp_flags & TCP_PUSH)
        printf("PUSH ");
    if (tcp_header->tcp_flags & TCP_ACK)
        printf("ACK ");
    if (tcp_header->tcp_flags & TCP_URG)
        printf("URG ");
    printf(" }\n");

    return header_size;
}
```

The decoding functions are passed a pointer to the start of the header, which is typecast to the appropriate structure. This allows accessing various fields of the header, but it’s important to remember these values will be in network byte order. This data is straight from the wire, so the byte order needs to be converted for use on an x86 processor.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o decode_sniff decode_sniff.c -lpcap
reader@hacking:~/booksrc $ sudo ./decode_sniff
Sniffing on device eth0
==== Got a 75 byte packet ====
[[ Layer 2 :: Ethernet Header ]]
[ Source: 00:01:29:15:65:b6 Dest: 00:01:6c:eb:1d:50 Type: 8 ]
    (( Layer 3 ::: IP Header ))
    ( Source: 192.168.42.1 Dest: 192.168.42.249 )
    ( Type: 6 ID: 7755 Length: 61 )
        {{ Layer 4 :::: TCP Header }}
        { Src Port: 35602 Dest Port: 7890 }
        { Seq #: 2887045274 Ack #: 3843058889 }
        { Header Size: 32 Flags: PUSH ACK }
            9 bytes of packet data
            74 65 73 74 69 6e 67 0d 0a | testing..
==== Got a 66 byte packet ====
[[ Layer 2 :: Ethernet Header ]]
[ Source: 00:01:6c:eb:1d:50 Dest: 00:01:29:15:65:b6 Type: 8 ]
    (( Layer 3 ::: IP Header ))
    ( Source: 192.168.42.249 Dest: 192.168.42.1 )
    ( Type: 6 ID: 15678 Length: 52 )
        {{ Layer 4 :::: TCP Header }}
        { Src Port: 7890 Dest Port: 35602 }
        { Seq #: 3843058889 Ack #: 2887045283 }
        { Header Size: 32 Flags: ACK }
            No Packet Data
==== Got a 82 byte packet ====
[[ Layer 2 :: Ethernet Header ]]
[ Source: 00:01:29:15:65:b6 Dest: 00:01:6c:eb:1d:50 Type: 8 ]
    (( Layer 3 ::: IP Header ))
    ( Source: 192.168.42.1 Dest: 192.168.42.249 )
    ( Type: 6 ID: 7756 Length: 68 )
        {{ Layer 4 :::: TCP Header }}
        { Src Port: 35602 Dest Port: 7890 }
        { Seq #: 2887045283 Ack #: 3843058889 }
        { Header Size: 32 Flags: PUSH ACK }
            16 bytes of packet data
            74 68 69 73 20 69 73 20 61 20 74 65 73 74 0d 0a | this is a test..
reader@hacking:~/booksrc $
</pre>

With the headers decoded and separated into layers, the TCP/IP connection is much easier to understand. Notice which IP addresses are associated with which MAC address. Also, notice how the sequence number in the two packets from _192.168.42.1_ (the first and last packet) increases by _nine_, since the first packet contained _nine bytes_ of actual data: _2887045283 – 2887045274 = 9_. This is used by the TCP protocol to make sure all of the data arrives in order, since packets could be delayed for various reasons.

Despite all of the mechanisms built into the packet headers, the packets are still visible to anyone on the same network segment. Protocols such as _FTP_, _POP3_, and _telnet_ transmit data without encryption. Even without the assistance of a tool like `dsniff`, it’s fairly trivial for an attacker sniffing the network to find the usernames and passwords in these packets and use them to compromise other systems. From a security perspective, this isn’t too good, so more intelligent switches provide switched network environments.