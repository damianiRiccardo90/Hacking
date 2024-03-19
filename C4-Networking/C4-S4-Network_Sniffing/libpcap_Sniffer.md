# *__libpcap Sniffer__*

A standardized programming library called __libpcap__ can be used to smooth out the inconsistencies of raw sockets. The functions in this library still use raw sockets to do their magic, but the library knows how to correctly work with raw sockets on multiple architectures. Both `tcpdump` and `dsniff` use _libpcap_, which allows them to compile with relative ease on any platform. Let’s rewrite the raw packet sniffer program using the libpcap’s functions instead of our own. These functions are quite intuitive, so we will discuss them using the following code listing.

__pcap_sniff.c__

```c
#include <pcap.h>
#include "hacking.h"

void pcap_fatal(const char* failed_in, const char* errbuf) 
{
    printf("Fatal Error in %s: %s\n", failed_in, errbuf);
    exit(1);
}
```

First, _pcap.h_ is included providing various structures and defines used by the pcap functions. Also, I’ve written a _pcap_fatal()_ function for displaying fatal errors. The pcap functions use a error buffer to return error and status messages, so this function is designed to display this buffer to the user.

```c
int main() 
{
    struct pcap_pkthdr header;
    const u_char* packet;
    char errbuf[PCAP_ERRBUF_SIZE];
    char* device;
    pcap_t* pcap_handle;
    int i;
```

The __errbuf__ variable is the aforementioned error buffer, its size coming from a define in _pcap.h_ set to _256_. The header variable is a __pcap_pkthdr__ structure containing extra capture information about the packet, such as when it was captured and its length. The __pcap_handle__ pointer works similarly to a file descriptor, but is used to reference a packet-capturing object.

```c
    device = pcap_lookupdev(errbuf);
    if (device == NULL)
        pcap_fatal("pcap_lookupdev", errbuf);
    
    printf("Sniffing on device %s\n", device);
```

The _pcap_lookupdev()_ function looks for a suitable device to sniff on. This device is returned as a string pointer referencing static function memory. For our system this will always be _/dev/eth0_, although it will be different on a _BSD_ system. If the function can’t find a suitable interface, it will return __NULL__.

```c
    pcap_handle = pcap_open_live(device, 4096, 1, 0, errbuf);
    if (pcap_handle == NULL)
        pcap_fatal("pcap_open_live", errbuf);
```

Similar to the socket function and file open function, the _pcap_open_live()_ function opens a packet-capturing device, returning a handle to it. The arguments for this function are the device to sniff, the maximum packet size, a promiscuous flag, a timeout value, and a pointer to the error buffer. Since we want to capture in _promiscuous mode_, the promiscuous flag is set to _1_.

```c
    for (i = 0; i < 3; i++) 
    {
        packet = pcap_next(pcap_handle, &header);
        printf("Got a %d byte packet\n", header.len);
        dump(packet, header.len);
    }
    pcap_close(pcap_handle);
}
```

Finally, the packet capture loop uses _pcap_next()_ to grab the next packet. This function is passed the __pcap_handle__ and a pointer to a __pcap_pkthdr__ structure so it can fill it with details of the capture. The function returns a pointer to the packet and then prints the packet, getting the length from the capture header. Then _pcap_close()_ closes the capture interface.

When this program is compiled, the pcap libraries must be linked. This can be done using the `-l` flag with GCC, as shown in the output below. The pcap library has been installed on this system, so the library and include files are already in standard locations the compiler knows about.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o pcap_sniff pcap_sniff.c
/tmp/ccYgieqx.o: In function `main':
pcap_sniff.c:(.text+0x1c8): undefined reference to `pcap_lookupdev'
pcap_sniff.c:(.text+0x233): undefined reference to `pcap_open_live'
pcap_sniff.c:(.text+0x282): undefined reference to `pcap_next'
pcap_sniff.c:(.text+0x2c2): undefined reference to `pcap_close'
collect2: ld returned 1 exit status
reader@hacking:~/booksrc $ gcc -o pcap_sniff pcap_sniff.c -l pcap
reader@hacking:~/booksrc $ ./pcap_sniff
Fatal Error in pcap_lookupdev: no suitable device found
reader@hacking:~/booksrc $ sudo ./pcap_sniff
Sniffing on device eth0
Got a 82 byte packet
00 01 6c eb 1d 50 00 01 29 15 65 b6 08 00 45 10 | ..l..P..).e...E.
00 44 1e 39 40 00 40 06 46 20 c0 a8 2a 01 c0 a8 | .D.9@.@.F ..*...
2a f9 8b 12 1e d2 ac 14 cf c7 e5 10 6c c9 80 18 | *...........l...
05 b4 54 1a 00 00 01 01 08 0a 26 b6 a7 76 02 3c | ..T.......&..v.<
37 1e 74 68 69 73 20 69 73 20 61 20 74 65 73 74 | 7.this is a test
0d 0a                                           | ..
Got a 66 byte packet
00 01 29 15 65 b6 00 01 6c eb 1d 50 08 00 45 00 | ..).e...l..P..E.
00 34 3d 2c 40 00 40 06 27 4d c0 a8 2a f9 c0 a8 | .4=,@.@.'M..*...
2a 01 1e d2 8b 12 e5 10 6c c9 ac 14 cf d7 80 10 | *.......l.......
05 a8 2b 3f 00 00 01 01 08 0a 02 47 27 6c 26 b6 | ..+?.......G'l&.
a7 76                                           | .v
Got a 84 byte packet
00 01 6c eb 1d 50 00 01 29 15 65 b6 08 00 45 10 | ..l..P..).e...E.
00 46 1e 3a 40 00 40 06 46 1d c0 a8 2a 01 c0 a8 | .F.:@.@.F...*...
2a f9 8b 12 1e d2 ac 14 cf d7 e5 10 6c c9 80 18 | *...........l...
05 b4 11 b3 00 00 01 01 08 0a 26 b6 a9 c8 02 47 | ..........&....G
27 6c 41 41 41 41 41 41 41 41 41 41 41 41 41 41 | 'lAAAAAAAAAAAAAA
41 41 0d 0a                                     | AA..
reader@hacking:~/booksrc $
</pre>

Notice that there are many bytes preceding the sample text in the packet and many of these bytes are similar. Since these are raw packet captures, most of these bytes are layers of header information for _Ethernet_, _IP_, and _TCP_.