# *__Network Byte Order__*

The port number and IP address used in the __AF_INET__ socket address structure are expected to follow the network byte ordering, which is _big-endian_. This is the opposite of x86’s _little-endian_ byte ordering, so these values must be converted. There are several functions specifically for these conversions, whose prototypes are defined in the _netinet/in.h_ and _arpa/inet.h_ include files. Here is a summary of these common byte order conversion functions:

- `htonl(long value)` __Host-to-Network Long__: Converts a 32-bit integer from the host’s byte order to network byte order
- `htons(short value)` __Host-to-Network Short__: Converts a 16-bit integer from the host’s byte order to network byte order
- `ntohl(long value)` __Network-to-Host Long__: Converts a 32-bit integer from network byte order to the host’s byte order
- `ntohs(long value)` __Network-to-Host Short__: Converts a 16-bit integer from network byte order to the host’s byte order

For compatibility with all architectures, these conversion functions should still be used even if the host is using a processor with _big-endian_ byte ordering.