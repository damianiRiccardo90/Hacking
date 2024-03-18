# *__Raw_Socket_Sniffer__*

So far in our code examples, we have been using _stream sockets_. When sending and receiving using stream sockets, the data is neatly wrapped in a TCP/IP connection. Accessing the _OSI model_ of the session (5) layer, the operating system takes care of all of the lower-level details of transmission, correction, and routing. It is possible to access the network at lower layers using __raw sockets__. At this lower layer, all the details are exposed and must be handled explicitly by the programmer. _Raw sockets_ are specified by using __SOCK_RAW__ as the type. In this case, the protocol matters since there are multiple options. The protocol can be __IPPROTO_TCP__, __IPPROTO_UDP__, or __IPPROTO_ICMP__. The following example is a TCP sniffing program using raw sockets.

__raw_tcpsniff.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#include "hacking.h"

int main(void) 
{
    int i, recv_length, sockfd;
    u_char buffer[9000];
    
    if ((sockfd = socket(PF_INET, SOCK_RAW, IPPROTO_TCP)) == -1)
        fatal("in socket");

    for (i = 0; i < 3; i++) 
    {
        recv_length = recv(sockfd, buffer, 8000, 0);
        printf("Got a %d byte packet\n", recv_length);
        dump(buffer, recv_length);
    }
}
```

This program opens a _raw TCP socket_ and listens for three packets, printing the raw data of each one with the _dump()_ function. Notice that buffer is declared as a __u_char__ variable. This is just a convenience type definition from _sys/socket.h_ that expands to _“unsigned char.”_ This is for convenience, since unsigned variables are used a lot in network programming and typing unsigned every time is a pain.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o raw_tcpsniff raw_tcpsniff.c
reader@hacking:~/booksrc $ ./raw_tcpsniff
[!!] Fatal Error in socket: Operation not permitted
reader@hacking:~/booksrc $ sudo ./raw_tcpsniff
Got a 68 byte packet
45 10 00 44 1e 36 40 00 40 06 46 23 c0 a8 2a 01 | E..D.6@.@.F#..*.
c0 a8 2a f9 8b 12 1e d2 ac 14 cf 92 e5 10 6c c9 | ..*...........l.
80 18 05 b4 32 47 00 00 01 01 08 0a 26 ab 9a f1 | ....2G......&...
02 3b 65 b7 74 68 69 73 20 69 73 20 61 20 74 65 | .;e.this is a te
73 74 0d 0a                                     | st..
Got a 70 byte packet
45 10 00 46 1e 37 40 00 40 06 46 20 c0 a8 2a 01 | E..F.7@.@.F ..*.
c0 a8 2a f9 8b 12 1e d2 ac 14 cf a2 e5 10 6c c9 | ..*...........l.
80 18 05 b4 27 95 00 00 01 01 08 0a 26 ab a0 75 | ....'.......&..u
02 3c 1b 28 41 41 41 41 41 41 41 41 41 41 41 41 | .<.(AAAAAAAAAAAA
41 41 41 41 0d 0a                               | AAAA..
Got a 71 byte packet
45 10 00 47 1e 38 40 00 40 06 46 1e c0 a8 2a 01 | E..G.8@.@.F...*.
c0 a8 2a f9 8b 12 1e d2 ac 14 cf b4 e5 10 6c c9 | ..*...........l.
80 18 05 b4 68 45 00 00 01 01 08 0a 26 ab b6 e7 | ....hE......&...
02 3c 20 ad 66 6a 73 64 61 6c 6b 66 6a 61 73 6b | .< .fjsdalkfjask
66 6a 61 73 64 0d 0a                            | fjasd..
reader@hacking:~/booksrc $
</pre>

While this program will capture packets, it isn’t reliable and will miss some packets, especially when there is a lot of traffic. Also, it only captures TCP packets—to capture UDP or ICMP packets, additional raw sockets need to be opened for each. Another big problem with raw sockets is that they are notoriously inconsistent between systems. Raw socket code for Linux most likely won’t work on _BSD_ or _Solaris_. This makes multiplatform programming with raw sockets nearly impossible.