# *__Socket Addresses__*

Many of the socket functions reference a __sockaddr__ structure to pass address information that defines a host. This structure is also defined in _bits/socket.h_, as shown on the following page.

__From /usr/include/bits/socket.h__

```c
/* Get the definition of the macro to define the common sockaddr members. */
#include <bits/sockaddr.h>

/* Structure describing a generic socket address. */
struct sockaddr
{
    __SOCKADDR_COMMON (sa_); /* Common data: address family and length. */
    char sa_data[14]; /* Address data. */
};
```

The macro for __SOCKADDR_COMMON__ is defined in the included _bits/sockaddr.h_ file, which basically translates to an unsigned short int. This value defines the address family of the address, and the rest of the structure is saved for address data. Since sockets can communicate using a variety of protocol families, each with their own way of defining endpoint addresses, the definition of an address must also be variable, depending on the address family. The possible address families are also defined in _bits/socket.h_, they usually translate directly to the corresponding protocol families.

__From /usr/include/bits/socket.h__

```c
/* Address families. */
#define AF_UNSPEC    PF_UNSPEC
#define AF_LOCAL     PF_LOCAL
#define AF_UNIX      PF_UNIX
#define AF_FILE      PF_FILE
#define AF_INET      PF_INET
#define AF_AX25      PF_AX25
#define AF_IPX       PF_IPX
#define AF_APPLETALK PF_APPLETALK
#define AF_NETROM    PF_NETROM
#define AF_BRIDGE    PF_BRIDGE
#define AF_ATMPVC    PF_ATMPVC
#define AF_X25       PF_X25
#define AF_INET6     PF_INET6
// (...)
```

Since an address can contain different types of information, depending on the address family, there are several other address structures that contain, in the address data section, common elements from the __sockaddr__ structure as well as information specific to the address family. These structures are also the same size, so they can be _typecast_ to and from each other. This means that a _socket()_ function will simply accept a pointer to a __sockaddr__ structure, which can in fact point to an address structure for _IPv4_, _IPv6_, or _X.25_. This allows the socket functions to operate on a variety of protocols.

In this book we are going to deal with _Internet Protocol version 4_, which is the protocol family __PF_INET__, using the address family __AF_INET__. The parallel socket address structure for __AF_INET__ is defined in the _netinet/in.h_ file.

__From /usr/include/netinet/in.h__

```c
/* Structure describing an Internet socket address. */
struct sockaddr_in
{
    __SOCKADDR_COMMON (sin_);
    in_port_t sin_port; /* Port number. */
    struct in_addr sin_addr; /* Internet address. */

    /* Pad to size of 'struct sockaddr'. */
    unsigned char sin_zero[sizeof (struct sockaddr) -
        __SOCKADDR_COMMON_SIZE -
        sizeof (in_port_t) -
        sizeof (struct in_addr)];
};
```

The __SOCKADDR_COMMON__ part at the top of the structure is simply the unsigned short int mentioned above, which is used to define the address family. Since a socket endpoint address consists of an Internet address and a port number, these are the next two values in the structure. The port number is a _16-bit short_, while the in_addr structure used for the Internet address contains a _32-bit_ number. The rest of the structure is just _8 bytes_ of padding to fill out the rest of the sockaddr structure. This space isn’t used for anything, but must be saved so the structures can be interchangeably typecast. In the end, the socket address structures end up looking like this:

<div align="left" width="100%">
<img src="sockaddr_structure.png?raw=true" alt="sockaddr Structure" width="100%">
</div>