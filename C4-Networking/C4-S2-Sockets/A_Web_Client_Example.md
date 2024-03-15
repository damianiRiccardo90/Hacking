# *__A Web Client Example__*

The _telnet_ program works well as a client for our server, so there really isn’t much reason to write a specialized client. However, there are thousands of different types of servers that accept standard _TCP/IP_ connections. Every time you use a web browser, it makes a connection to a web-server somewhere. This connection transmits the web page over the connection using _HTTP_, which defines a certain way to request and send information. By default, web-servers run on port _80_, which is listed along with many other default ports in _/etc/services_.

__From /etc/services__

<pre style="color: white;">
finger    79/tcp     # Finger
finger    79/udp
http      80/tcp www www-http # World Wide Web HTTP
</pre>

__HTTP__ exists in the _application layer_, the top layer, of the _OSI_ model. At this layer, all of the networking details have already been taken care of by the lower layers, so HTTP uses plaintext for its structure. Many other application layer protocols also use plaintext, such as __POP3__, __SMTP__, __IMAP__, and __FTP__’s control channel. Since these are standard protocols, they are all well documented and easily researched. Once you know the syntax of these various protocols, you can manually talk to other programs that speak the same language. There’s no need to be fluent, but knowing a few important phrases will help you when traveling to foreign servers. In the language of HTTP, requests are made using the command `GET`, followed by the resource path and the HTTP protocol version. For example, `GET / HTTP/1.0` will request the root document from the web-server using _HTTP version 1.0_. The request is actually for the root directory of _/_, but most web-servers will automatically search for a default HTML document in that directory of _index.html_. If the server finds the resource, it will respond using HTTP by sending several headers before sending the content. If the command `HEAD` is used instead of `GET`, it will only return the _HTTP headers_ without the content. These headers are plaintext and can usually provide information about the server. These headers can be retrieved manually using _telnet_ by connecting to port _80_ of a known website, then typing `HEAD / HTTP/1.0` and pressing _ENTER_ twice. In the output below, telnet is used to open a TCP-IP connection to the web-server at http://www.internic.net. Then the HTTP _application layer_ is manually spoken to request the headers for the main index page.

<pre style="color: white;">
reader@hacking:~/booksrc $ telnet www.internic.net 80
Trying 208.77.188.101...
Connected to www.internic.net.
Escape character is '^]'.
HEAD / HTTP/1.0

HTTP/1.1 200 OK
Date: Fri, 14 Sep 2007 05:34:14 GMT
Server: Apache/2.0.52 (CentOS)
Accept-Ranges: bytes
Content-Length: 6743
Connection: close
Content-Type: text/html; charset=UTF-8

Connection closed by foreign host.
reader@hacking:~/booksrc $
</pre>

This reveals that the web-server is _Apache version 2.0.52_ and even that the host runs _CentOS_. This can be useful for profiling, so let’s write a program that automates this manual process.

The next few programs will be sending and receiving a lot of data. Since the standard socket functions aren’t very friendly, let’s write some functions to send and receive data. These functions, called _send_string()_ and _recv_line()_, will be added to a new include file called _hacking-network.h_.

The normal _send()_ function returns the number of bytes written, which isn’t always equal to the number of bytes you tried to send. The _send_string()_ function accepts a socket and a string pointer as arguments and makes sure the entire string is sent out over the socket. It uses _strlen()_ to figure out the total length of the string passed to it.

You may have noticed that every packet the simple server received ended with the bytes _0x0D_ and _0x0A_. This is how telnet terminates the lines—it sends a _carriage return_ and a _newline_ character. The HTTP protocol also expects lines to be terminated with these two bytes. A quick look at an _ASCII table_ shows that _0x0D_ is a carriage return (_'\r'_) and _0x0A_ is the newline character (_'\n'_).

<pre style="color: white;">
reader@hacking:~/booksrc $ man ascii | egrep "Hex|0A|0D"
Reformatting ascii(7), please wait...
    Oct   Dec   Hex   Char                      Oct   Dec   Hex   Char
    012   10    0A    LF '\n' (new line)        112   74    4A    J
    015   13    0D    CR '\r' (carriage ret)    115   77    4D    M
reader@hacking:~/booksrc $
</pre>

The _recv_line()_ function reads entire lines of data. It reads from the socket passed as the first argument into the a buffer that the second argument points to. It continues receiving from the socket until it encounters the last two line termination bytes in sequence. Then it terminates the string and exits the function. These new functions ensure that all bytes are sent and receive data as lines terminated by _'\r\n'_. They are listed below in a new include file called _hacking-network.h_.

__hacking-network.h__

```c
/* 
 * This function accepts a socket FD and a ptr to the null terminated
 * string to send. The function will make sure all the bytes of the
 * string are sent. Returns 1 on success and 0 on failure.
 */
int send_string(int sockfd, unsigned char* buffer) 
{
    int sent_bytes, bytes_to_send;
    bytes_to_send = strlen(buffer);
    while (bytes_to_send > 0) 
    {
        sent_bytes = send(sockfd, buffer, bytes_to_send, 0);
        if (sent_bytes == -1)
            return 0; // Return 0 on send error.
        bytes_to_send -= sent_bytes;
        buffer += sent_bytes;
    }
    return 1; // Return 1 on success.
}

/* 
 * This function accepts a socket FD and a ptr to a destination
 * buffer. It will receive from the socket until the EOL byte
 * sequence is seen. The EOL bytes are read from the socket, but
 * the destination buffer is terminated before these bytes.
 * Returns the size of the read line (without EOL bytes).
 */
int recv_line(int sockfd, unsigned char* dest_buffer) 
{
#define EOL "\r\n" // End-of-line byte sequence
#define EOL_SIZE 2
    unsigned char* ptr;
    int eol_matched = 0;

    ptr = dest_buffer;
    // Read a single byte.
    while (recv(sockfd, ptr, 1, 0) == 1) 
    {
        // Does this byte match terminator?
        if(*ptr == EOL[eol_matched]) 
        {
            eol_matched++;
            // If all bytes match terminator,
            if (eol_matched == EOL_SIZE) 
            {
                *(ptr + 1 - EOL_SIZE) = '\0'; // terminate the string.
                return strlen(dest_buffer); // Return bytes received
            }
        } 
        else 
        {
            eol_matched = 0;
        }
        ptr++; // Increment the pointer to the next byte.
    }
    return 0; // Didn't find the end-of-line characters.
}
```

Making a socket connection to a numerical IP address is pretty simple but named addresses are commonly used for convenience. In the manual HTTP __HEAD__ request, the _telnet_ program automatically does a __DNS__ (_Domain Name Service_) lookup to determine that _www.internic.net_ translates to the IP address _192.0.34.161_. _DNS_ is a protocol that allows an IP address to be looked up by a named address, similar to how a phone number can be looked up in a phone book if you know the name. Naturally, there are socket-related functions and structures specifically for _hostname_ lookups via _DNS_. These functions and structures are defined in _netdb.h_. A function called _gethostbyname()_ takes a pointer to a string containing a named address and returns a pointer to a __hostent__ structure, or __NULL__ pointer on error. The __hostent__ structure is filled with information from the lookup, including the numerical IP address as a _32-bit integer_ in network byte order. Similar to the _inet_ntoa()_ function, the memory for this structure is statically allocated in the function. This structure is shown below, as listed in _netdb.h_.

__From /usr/include/netdb.h__

```c
/* Description of database entry for a single host. */
struct hostent
{
    char* h_name; /* Official name of host. */
    char** h_aliases; /* Alias list. */
    int h_addrtype; /* Host address type. */
    int h_length; /* Length of address. */
    char** h_addr_list; /* List of addresses from name server. */
#define h_addr h_addr_list[0] /* Address, for backward compatibility. */
};
```

The following code demonstrates the use of the _gethostbyname()_ function.

__host_lookup.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#include <netdb.h>

#include "hacking.h"

int main(int argc, char* argv[]) 
{
    struct hostent* host_info;
struct in_addr *address;
if(argc < 2) {
printf("Usage: %s <hostname>\n", argv[0]);
exit(1);
}
host_info = gethostbyname(argv[1]);
if(host_info == NULL) {
printf("Couldn't lookup %s\n", argv[1]);
} else {
address = (struct in_addr *) (host_info->h_addr);
printf("%s has address %s\n", argv[1], inet_ntoa(*address));
}
}
```