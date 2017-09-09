# Unix Domain Socket

Socket in Unix has been known for its capability to establish connection between client and server on the Internet, however the socket interface can also be used as a mean to let two processes to communicate with each other.

Before establishing any connection to an endpoint, we need to instantiate a file descriptor for holding the socket, it returns `-1` when it has failed to instantiate the socket.

`int socket(int socket_family, int socket_type, int protocol);`

The `socket()` function creates a network descriptor that can be used with later calls to bind,listen and accept. we need specify its socket family in the first argument it can either be `AF_INET` for IPv4 or `AF_INET6` for IPv6. **socket type** as a second argument let us decide which network protocol to choose from, it can either be `SOCK_STREAM` for TCP or `SOCK_DGRAM` for UDP. Last argument we need to pass to the function is an integer value that indicates the protocol, it is an optional configuration, ususally set to 0. 

`sockaddr_in` is a structure that we used to store the address information for both client and server:

```
struct sockaddr_in
{
  short   sin_family; /* must be AF_INET */
  u_short sin_port;
  struct  in_addr sin_addr;
  char    sin_zero[8]; /* Not used, must be zero */
};
```

### `bind()`

`int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`

`bind()` associates an abstract socket with an actual network interface and port. It is possible to call bind on a TCP client however it's unusually unnecessary to specify the outgoing port.

### `listen()`

`int listen(int sockfd, int backlog);`

`listen()` specifies the queue size for the number of incoming, unhandled connections i.e. that have not yet been assigned a network descriptor by accept Typical values for a high performance server are 128 or more.

Server sockets do not actively try to connect to another host; instead they wait for incoming connections. Additionally, server sockets are not closed when the peer disconnects. Instead the client communicates with a separate active socket on the server that is specific to that connection.

### `accept()`

`int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);`

Once the server socket has been initialized the server calls `accept()` to wait for new connections. Unlike socket `bind()` and `listen()`, this call will block. i.e. if there are no new connections, this call will block and only return when a new client connects. The returned TCP socket is associated with a particular tuple (client IP, client port, server IP, server port) and will be used for all future incoming and outgoing TCP packets that match this tuple.

`accept()` returns a new file descriptor. This file descriptor is specific to a particular client.

### `read() and write()`

* `ssize_t read(int fd, void *buf, size_t count);`

* `ssize_t write(int fd, const void *buf, size_t count);`

Once we have a successful connection we can read or write like any old file descriptor. Keep in mind if you are connected to a website, you want to conform to the HTTP protocol specification in order to get any sort of meaningful results back. There are libraries to do this, usually you don't connect at the socket level because there are other libraries or packages around it.

Both `read()` and `write()` requires a file descriptor as the first argument to put data into and getting it out, the second argument specifies the address to the data we want to write or read from the file descriptor, and the size of the data as the last argument. 

### `connect()`

`int connect(int socket, const struct sockaddr *address, socklen_t address_len);`

`connect()` initiate a connection on a socket on the client side, the first argument we need to pass in is the network descriptor we initialized during the `socket()` call for our client to hold a connection. The second argument is the address information we want to store about the other end, and lastly the length of the socket as the third argument.

## Example (Server)

```
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <unistd.h>
#include <arpa/inet.h>

int main(int argc, char **argv)
{
    int s;
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);

    struct addrinfo hints, *result;
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_family = AF_INET;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;

    s = getaddrinfo(NULL, "1234", &hints, &result);
    if (s != 0) {
            fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(s));
            exit(1);
    }

    if (bind(sock_fd, result->ai_addr, result->ai_addrlen) != 0) {
        perror("bind()");
        exit(1);
    }

    if (listen(sock_fd, 10) != 0) {
        perror("listen()");
        exit(1);
    }
    
    struct sockaddr_in *result_addr = (struct sockaddr_in *) result->ai_addr;
    printf("Listening on file descriptor %d, port %d\n", sock_fd, ntohs(result_addr->sin_port));

    printf("Waiting for connection...\n");
    int client_fd = accept(sock_fd, NULL, NULL);
    printf("Connection made: client_fd=%d\n", client_fd);

    char buffer[1000];
    int len = read(client_fd, buffer, sizeof(buffer) - 1);
    buffer[len] = '\0';

    printf("Read %d chars\n", len);
    printf("===\n");
    printf("%s\n", buffer);

    return 0;
}
```

## Example (Client)

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <unistd.h>

int main(int argc, char **argv)
{
    int s;
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);

    struct addrinfo hints, *result;
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_family = AF_INET; /* IPv4 only */
    hints.ai_socktype = SOCK_STREAM; /* TCP */

    s = getaddrinfo("www.ust.hk", "80", &hints, &result);
    if (s != 0) {
            fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(s));
            exit(1);
    }

    if(connect(sock_fd, result->ai_addr, result->ai_addrlen) == -1){
                perror("connect");
                exit(2);
        }

    char *buffer = "GET / HTTP/1.0\r\n\r\n";
    printf("SENDING: %s", buffer);
    printf("===\n");

        // For this trivial demo just assume write() sends all bytes in one go and is not interrupted

    write(sock_fd, buffer, strlen(buffer));


    char resp[1000];
    int len = read(sock_fd, resp, 999);
    resp[len] = '\0';
    printf("%s\n", resp);

    return 0;
}
```