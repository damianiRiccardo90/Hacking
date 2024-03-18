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

