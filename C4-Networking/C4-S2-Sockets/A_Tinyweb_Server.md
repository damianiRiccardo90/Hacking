# *__A Tinyweb Server__*

A web-server doesn’t have to be much more complex than the simple server we created in the previous section. After accepting a TCP-IP connection, the web-server needs to implement further layers of communication using the HTTP protocol.

The server code listed below is nearly identical to the simple server, except that connection handling code is separated into its own function. This function handles _HTTP_ __GET__ and __HEAD__ requests that would come from a web browser. The program will look for the requested resource in the local directory called web-root and send it to the browser. If the file can’t be found, the server will respond with a _404 HTTP response_. You may already be familiar with this response, which means _File Not Found_. The complete source code listing follows.

```c
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "hacking.h"
#include "hacking-network.h"

#define PORT 80 // The port users will be connecting to
#define WEBROOT "./webroot" // The webserver's root directory

void handle_connection(int, struct sockaddr_in*); // Handle web requests
int get_file_size(int); // Returns the filesize of open file descriptor

int main(void) 
{
    int sockfd, new_sockfd, yes = 1;
    struct sockaddr_in host_addr, client_addr; // My address information
    socklen_t sin_size;

    printf("Accepting web requests on port %d\n", PORT);

    if ((sockfd = socket(PF_INET, SOCK_STREAM, 0)) == -1)
        fatal("in socket");

    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int)) == -1)
        fatal("setting socket option SO_REUSEADDR");

    host_addr.sin_family = AF_INET; // Host byte order
    host_addr.sin_port = htons(PORT); // Short, network byte order
    host_addr.sin_addr.s_addr = INADDR_ANY; // Automatically fill with my IP.
    memset(&(host_addr.sin_zero), '\0', 8); // Zero the rest of the struct.

    if (bind(sockfd, (struct sockaddr*)&host_addr, sizeof(struct sockaddr)) == -1)
        fatal("binding to socket");

    if (listen(sockfd, 20) == -1)
        fatal("listening on socket");

    // Accept loop.
    while (1) 
    {
        sin_size = sizeof(struct sockaddr_in);
        new_sockfd = accept(sockfd, (struct sockaddr*)&client_addr, &sin_size);
        if (new_sockfd == -1)
            fatal("accepting connection");

        handle_connection(new_sockfd, &client_addr);
    }
    return 0;
}

/* 
 * This function handles the connection on the passed socket from the
 * passed client address. The connection is processed as a web request,
 * and this function replies over the connected socket. Finally, the
 * passed socket is closed at the end of the function.
 */
void handle_connection(int sockfd, struct sockaddr_in* client_addr_ptr) 
{
    unsigned char* ptr, request[500], resource[500];
    int fd, length;
    
    length = recv_line(sockfd, request);

    printf("Got request from %s:%d \"%s\"\n", inet_ntoa(client_addr_ptr->sin_addr),
        ntohs(client_addr_ptr->sin_port), request);

    ptr = strstr(request, " HTTP/"); // Search for valid-looking request.
    // Then this isn't valid HTTP.
    if (ptr == NULL) 
    {
        printf(" NOT HTTP!\n");
    } 
    else 
    {
        *ptr = 0; // Terminate the buffer at the end of the URL.
        ptr = NULL; // Set ptr to NULL (used to flag for an invalid request).
        if (strncmp(request, "GET ", 4) == 0) // GET request
            ptr = request + 4; // ptr is the URL.
        if (strncmp(request, "HEAD ", 5) == 0) // HEAD request
            ptr = request + 5; // ptr is the URL.

        // Then this is not a recognized request.
        if (ptr == NULL) 
        {
            printf("\tUNKNOWN REQUEST!\n");
        } 
        // Valid request, with ptr pointing to the resource name
        else 
        {
            if (ptr[strlen(ptr) - 1] == '/') // For resources ending with '/',
            strcat(ptr, "index.html"); // add 'index.html' to the end.
            strcpy(resource, WEBROOT); // Begin resource with web root path
            strcat(resource, ptr); // and join it with resource path.
            fd = open(resource, O_RDONLY, 0); // Try to open the file.
            printf("\tOpening \'%s\'\t", resource);
            // If file is not found
            if (fd == -1) 
            {
                printf(" 404 Not Found\n");
                send_string(sockfd, "HTTP/1.0 404 NOT FOUND\r\n");
                send_string(sockfd, "Server: Tiny webserver\r\n\r\n");
                send_string(sockfd, "<html><head><title>404 Not Found</title></head>");
                send_string(sockfd, "<body><h1>URL not found</h1></body></html>\r\n");
            }
            // Otherwise, serve up the file.
            else 
            {
                printf(" 200 OK\n");
                send_string(sockfd, "HTTP/1.0 200 OK\r\n");
                send_string(sockfd, "Server: Tiny webserver\r\n\r\n");
                // Then this is a GET request
                if (ptr == request + 4) 
                {
                    if ((length = get_file_size(fd)) == -1)
                        fatal("getting resource file size");
                    if ((ptr = (unsigned char*) malloc(length)) == NULL)
                        fatal("allocating memory for reading resource");
                    read(fd, ptr, length); // Read the file into memory.
                    send(sockfd, ptr, length, 0); // Send it to socket.
                    free(ptr); // Free file memory.
                }
                close(fd); // Close the file.
            } // End if block for file found/not found.
        } // End if block for valid request.
    } // End if block for valid HTTP.
    shutdown(sockfd, SHUT_RDWR); // Close the socket gracefully.
}

/* 
 * This function accepts an open file descriptor and returns
 * the size of the associated file. Returns -1 on failure.
 */
int get_file_size(int fd) 
{
    struct stat stat_struct;

    if (fstat(fd, &stat_struct) == -1)
        return -1;
    return (int) stat_struct.st_size;
}           
```

The handle_connection function uses the _strstr()_ function to look for the substring _HTTP/_ in the request buffer. The _strstr()_ function returns a pointer to the substring, which will be right at the end of the request. The string is terminated here, and the requests __HEAD__ and __GET__ are recognized as processable requests. A _HEAD_ request will just return the headers, while a _GET_ request will also return the requested resource (if it can be found).

The files _index.html_ and _image.jpg_ have been put into the directory web-root, as shown in the output below, and then the _tinyweb_ program is compiled. Root privileges are needed to bind to any port below _1024_, so the program is _setuid root_ and executed. The server’s debugging output shows the results of a web browser’s request of http://127.0.0.1:

<pre style="color: white;">
reader@hacking:~/booksrc $ ls -l webroot/
total 52
-rwxr--r-- 1 reader reader 46794 2007-05-28 23:43 image.jpg
-rw-r--r-- 1 reader reader 261 2007-05-28 23:42 index.html
reader@hacking:~/booksrc $ cat webroot/index.html
&lt;html&gt;
&lt;head&gt;&lt;title&gt;A sample webpage&lt;/title&gt;&lt;/head&gt;
&lt;body bgcolor="#000000" text="#ffffffff"&gt;
&lt;center&gt;
&lt;h1&gt;This is a sample webpage&lt;/h1&gt;
...and here is some sample text&lt;br&gt;
&lt;br&gt;
..and even a sample image:&lt;br&gt;
&lt;img src="image.jpg"&gt;&lt;br&gt;
&lt;/center&gt;
&lt;/body&gt;
&lt;/html&gt;
reader@hacking:~/booksrc $ gcc -o tinyweb tinyweb.c
reader@hacking:~/booksrc $ sudo chown root ./tinyweb
reader@hacking:~/booksrc $ sudo chmod u+s ./tinyweb
reader@hacking:~/booksrc $ ./tinyweb
Accepting web requests on port 80
Got request from 127.0.0.1:52996 "GET / HTTP/1.1"
Opening './webroot/index.html' 200 OK
Got request from 127.0.0.1:52997 "GET /image.jpg HTTP/1.1"
Opening './webroot/image.jpg' 200 OK
Got request from 127.0.0.1:52998 "GET /favicon.ico HTTP/1.1"
Opening './webroot/favicon.ico' 404 Not Found
</pre>

The address _127.0.0.1_ is a special loopback address that routes to the local machine. The initial request gets _index.html_ from the web-server, which in turn requests _image.jpg_. In addition, the browser automatically requests _favicon.ico_ in an attempt to retrieve an icon for the web page. The screenshot below shows the results of this request in a browser.

<div align="left" width="100%">
<img src="A_Sample_Webpage.png?raw=true" alt="A Sample Webpage" width="100%">
</div>