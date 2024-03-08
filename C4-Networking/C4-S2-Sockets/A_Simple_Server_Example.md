# *__A Simple Server Example__*

The best way to show how these functions are used is by example. The following server code listens for _TCP_ connections on port _7890_. When a client connects, it sends the message _Hello, world!_ and then receives data until the connection is closed. This is done using socket functions and structures from the include files mentioned earlier, so these files are included at the beginning of the program. A useful memory dump function has been added to _hacking.h_, which is shown on the following page.

__Added to hacking.h__

```c
// Dumps raw memory in hex byte and printable split format
void dump(const unsigned char* data_buffer, const unsigned int length) 
{
    unsigned char byte;
    unsigned int i, j;
    for (i = 0; i < length; i++) 
    {
        byte = data_buffer[i];
        printf("%02x ", data_buffer[i]); // Display byte in hex.
        if (((i % 16) == 15) || (i == length - 1)) 
        {
            for (j = 0; j < 15 - (i % 16); j++)
                printf(" ");
            printf("| ");
            // Display printable bytes from line.
            for (j = (i - (i % 16)); j <= i; j++) 
            {
                byte = data_buffer[j];
                if ((byte > 31) && (byte < 127)) // Outside printable char range
                    printf("%c", byte);
                else
                    printf(".");
            }
        printf("\n"); // End of the dump line (each line is 16 bytes)
        } // End if
    } // End for
}
```

This function is used to display packet data by the server program. However, since it is also useful in other places, it has been put into _hacking.h_, instead. The rest of the server program will be explained as you read the source code.

__simple_server.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "hacking.h"

#define PORT 7890 // The port users will be connecting to

int main(void) 
{
    int sockfd, new_sockfd; // Listen on sock_fd, new connection on new_fd
    struct sockaddr_in host_addr, client_addr; // My address information
    socklen_t sin_size;
    int recv_length = 1, yes = 1;
    char buffer[1024];

    if ((sockfd = socket(PF_INET, SOCK_STREAM, 0)) == -1)
        fatal("in socket");
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int)) == -1)
        fatal("setting socket option SO_REUSEADDR");
```

So far, the program sets up a socket using the _socket()_ function. We want a _TCP/IP_ socket, so the protocol family is __PF_INET__ for _IPv4_ and the socket type is __SOCK_STREAM__ for a _stream socket_. The final protocol argument is _0_, since there is only one protocol in the __PF_INET__ protocol family. This function returns a socket file descriptor which is stored in __sockfd__. 

The _setsockopt()_ function is simply used to set socket options. This function call sets the __SO_REUSEADDR__ socket option to true, which will allow it to reuse a given address for binding. Without this option set, when the program tries to bind to a given port, it will fail if that port is already in use. If a socket isn’t closed properly, it may appear to be in use, so this option lets a socket bind to a port (and take over control of it), even if it seems to be in use.

The first argument to this function is the socket (referenced by a file descriptor), the second specifies the level of the option, and the third specifies the option itself. Since __SO_REUSEADDR__ is a socket-level option, the level is set to __SOL_SOCKET__. There are many different socket options defined in _/usr/include/asm/socket.h_. The final two arguments are a pointer to the data that the option should be set to and the length of that data. A pointer to data and the length of that data are two arguments that are often used with socket functions. This allows the functions to handle all sorts of data, from single bytes to large data structures. The __O_REUSEADDR__ options uses a _32-bit_ integer for its value, so to set this option to true, the final two arguments must be a pointer to the integer value of _1_ and the size of an integer (which is _4 bytes_).

```c

```