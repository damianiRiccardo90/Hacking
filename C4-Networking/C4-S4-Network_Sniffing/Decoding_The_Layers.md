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

```