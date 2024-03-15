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

The first argument to this function is the socket (referenced by a file descriptor),  the second specifies the level of the option, and the third specifies the option itself. Since __SO_REUSEADDR__ is a socket-level option, the level is set to __SOL_SOCKET__. There are many different socket options defined in _/usr/include/asm/socket.h_. The final two arguments are a pointer to the data that the option should be set to and the length of that data. A pointer to data and the length of that data are two arguments that are often used with socket functions. This allows the functions to handle all sorts of data, from single bytes to large data structures. The __O_REUSEADDR__ options uses a _32-bit_ integer for its value, so to set this option to true, the final two arguments must be a pointer to the integer value of _1_ and the size of an integer (which is _4 bytes_).

```c
    host_addr.sin_family = AF_INET; // Host byte order
    host_addr.sin_port = htons(PORT); // Short, network byte order
    host_addr.sin_addr.s_addr = 0; // Automatically fill with my IP.
    memset(&(host_addr.sin_zero), '\0', 8); // Zero the rest of the struct.

    if (bind(sockfd, (struct sockaddr *)&host_addr, sizeof(struct sockaddr)) == -1)
        fatal("binding to socket");

    if (listen(sockfd, 5) == -1)
        fatal("listening on socket");
```

These next few lines set up the __host_addr__ structure for use in the bind call. The address family is __AF_INET__, since we are using _IPv4_ and the __sockaddr_in__ structure. The port is set to __PORT__, which is defined as _7890_. This short integer value must be converted into network byte order, so the _htons()_ function is used. The address is set to _0_, which means it will automatically be filled with the host’s current IP address. Since the value _0_ is the same regardless of byte order, no conversion is necessary.

The _bind()_ call passes the socket file descriptor, the address structure, and the length of the address structure. This call will bind the socket to the current IP address on port _7890_.

The _listen()_ call tells the socket to listen for incoming connections, and a subsequent _accept()_ call actually accepts an incoming connection. The _listen()_ function places all incoming connections into a backlog queue until an _accept()_ call accepts the connections. The last argument to the _listen()_ call sets the maximum size for the backlog queue.

```c
// Accept loop.
    while(1) 
    {
        sin_size = sizeof(struct sockaddr_in);
        new_sockfd = accept(sockfd, (struct sockaddr*)&client_addr, &sin_size);
        if (new_sockfd == -1)
            fatal("accepting connection");
        printf("server: got connection from %s port %d\n",
            inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));
        send(new_sockfd, "Hello, world!\n", 13, 0);
        recv_length = recv(new_sockfd, &buffer, 1024, 0);
        while (recv_length > 0) 
        {
            printf("RECV: %d bytes\n", recv_length);
            dump(buffer, recv_length);
            recv_length = recv(new_sockfd, &buffer, 1024, 0);
        }
        close(new_sockfd);
    }
    return 0;
}
```

Next is a loop that accepts incoming connections. The _accept()_ function’s first two arguments should make sense immediately; the final argument is a pointer to the size of the address structure. This is because the _accept()_ function will write the connecting client’s address information into the address structure and the size of that structure into __sin_size__. For our purposes, the size never changes, but to use the function we must obey the calling convention. The _accept()_ function returns a new socket file descriptor for the accepted connection. This way, the original socket file descriptor can continue to be used for accepting new connections, while the new socket file descriptor is used for communicating with the connected client.

After getting a connection, the program prints out a connection message, using _inet_ntoa()_ to convert the __sin_addr__ address structure to a dotted-number IP string and _ntohs()_ to convert the byte order of the __sin_port__ number.

The _send()_ function sends the _13_ bytes of the string _Hello, world!\n_ to the new socket that describes the new connection. The final argument for the _send()_ and _recv()_ functions are flags, that for our purposes, will always be _0_.

Next is a loop that receives data from the connection and prints it out. The _recv()_ function is given a pointer to a buffer and a maximum length to read from the socket. The function writes the data into the buffer passed to it and returns the number of bytes it actually wrote. The loop will continue as long as the _recv()_ call continues to receive data.

When compiled and run, the program binds to port _7890_ of the host and waits for incoming connections:

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc simple_server.c
reader@hacking:~/booksrc $ ./a.out
</pre>

A __telnet__ client basically works like a generic _TCP_ connection client, so it can be used to connect to the simple server by specifying the target IP address and port.

__From a Remote Machine__

<pre style="color: white;">
matrix@euclid:~ $ telnet 192.168.42.248 7890
Trying 192.168.42.248...
Connected to 192.168.42.248.
Escape character is '^]'.
Hello, world!
this is a test
fjsghau;ehg;ihskjfhasdkfjhaskjvhfdkjhvbkjgf
</pre>

Upon connection, the server sends the string _Hello, world!_, and the rest is the local character echo of me typing this is a test and a line of keyboard mashing. Since _telnet_ is line-buffered, each of these two lines is sent back to the server when _ENTER_ is pressed. Back on the server side, the output shows the connection and the packets of data that are sent back.

__On a Local Machine__

<pre style="color: white;">
reader@hacking:~/booksrc $ ./a.out
server: got connection from 192.168.42.1 port 56971
RECV: 16 bytes
74 68 69 73 20 69 73 20 61 20 74 65 73 74 0d 0a | This is a test...
RECV: 45 bytes
66 6a 73 67 68 61 75 3b 65 68 67 3b 69 68 73 6b | fjsghau;ehg;ihsk
6a 66 68 61 73 64 6b 66 6a 68 61 73 6b 6a 76 68 | jfhasdkfjhaskjvh
66 64 6b 6a 68 76 62 6b 6a 67 66 0d 0a          | fdkjhvbkjgf...
</pre>