# *__Socket Functions__*

In C, sockets behave a lot like files since they use file descriptors to identify themselves. Sockets behave so much like files that you can actually use the _read()_ and _write()_ functions to receive and send data using socket file descriptors. However, there are several functions specifically designed for dealing with sockets. These functions have their prototypes defined in _/usr/include/sys/sockets.h_.

- `socket(int domain, int type, int protocol)` Used to create a new socket, returns a file descriptor for the socket or _-1_ on error.
- `connect(int fd, struct sockaddr* remote_host, socklen_t addr_length)` Connects a socket (described by file descriptor __fd__) to a remote host. Returns _0_ on success and _-1_ on error.
- `bind(int fd, struct sockaddr* local_addr, socklen_t addr_length)` Binds a socket to a local address so it can listen for incoming connections. Returns _0_ on success and _-1_ on error.
- `listen(int fd, int backlog_queue_size)` Listens for incoming connections and queues connection requests up to __backlog_queue_size__. Returns _0_ on success and _-1_ on error.
- `accept(int fd, sockaddr* remote_host, socklen_t* addr_length)` Accepts an incoming connection on a bound socket. The address information from the remote host is written into the remote_host structure and the actual size of the address structure is written __into *addr_length__. This function returns a new socket file descriptor to identify the connected socket or _-1_ on error.
- `send(int fd, void* buffer, size_t n, int flags)` Sends _n_ bytes from __buffer__ to socket __fd__; returns the number of bytes sent or _-1_ on error.
- `recv(int fd, void* buffer, size_t n, int flags)` Receives _n_ bytes from socket __fd__ into __buffer__; returns the number of bytes received or _-1_ on error.

When a socket is created with the _socket()_ function, the _domain_, _type_, and _protocol_ of the socket must be specified. The domain refers to the protocol family of the socket. A socket can be used to communicate using a variety of protocols, from the standard _Internet protocol_ used when you browse the Web to amateur radio protocols such as _AX.25_ (when you are being a gigantic nerd). These protocol families are defined in _bits/socket.h_, which is automatically included from _sys/socket.h_.

__From /usr/include/bits/socket.h__

```c
/* Protocol families. */
#define PF_UNSPEC    0 /* Unspecified. */
#define PF_LOCAL     1 /* Local to host (pipes and file-domain). */
#define PF_UNIX      PF_LOCAL /* Old BSD name for PF_LOCAL. */
#define PF_FILE      PF_LOCAL /* Another nonstandard name for PF_LOCAL. */
#define PF_INET      2 /* IP protocol family. */
#define PF_AX25      3 /* Amateur Radio AX.25. */
#define PF_IPX       4 /* Novell Internet Protocol. */
#define PF_APPLETALK 5 /* Appletalk DDP. */
#define PF_NETROM    6 /* Amateur radio NetROM. */
#define PF_BRIDGE    7 /* Multiprotocol bridge. */
#define PF_ATMPVC    8 /* ATM PVCs. */
#define PF_X25       9 /* Reserved for X.25 project. */
#define PF_INET6     10 /* IP version 6. */
// (...)
```

As mentioned before, there are several types of sockets, although stream sockets and datagram sockets are the most commonly used. The types of sockets are also defined in bits/socket.h. (The _/* comments */_ in the code above are just another style that comments out everything between the asterisks.)

__From /usr/include/bits/socket.h__

```c
/* Types of sockets. */
enum __socket_type
{
    SOCK_STREAM = 1, /* Sequenced, reliable, connection-based byte streams. */
#define SOCK_STREAM SOCK_STREAM
    SOCK_DGRAM = 2, /* Connectionless, unreliable datagrams of fixed maximum length. */
#define SOCK_DGRAM SOCK_DGRAM
// (...)
```

The final argument for the _socket()_ function is the protocol, which should almost always be _0_. The specification allows for multiple protocols within a protocol family, so this argument is used to select one of the protocols from the family. In practice, however, most protocol families only have one protocol, which means this should usually be set for _0_; the first and only protocol in the enumeration of the family. This is the case for everything we will do with sockets in this book, so this argument will always be _0_ in our examples.