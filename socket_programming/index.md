# Socket Programming (TCP)

The C socket API allows a developer to design and build a TCP/IP client/server program. To begin with a socket address structure is needed (<sys/socket.h>):
```C
struct sockaddr_in {
    sa_family_t     sin_family; // AF_INET for TCP/IP
    in_port_t       sin_port;   // port number
    struct in_addr  sin_addr;   // IP address
};

struct in_addr {                // internet address (in_addr)
    uint32_t        s_addr;     // IP address in network byte order
};
```
for TCP/IP connections sin_family is always set to **AF_INET**, sin_port contains the port number (in network byte order) and finally sin_addr is the host IP address, also in network byte order.

In this socket API, the server must create a socket and bind it with a sockaddr, which is the server's IP address and port number. A fixed port number can be used or left as 0 to allow the OS kernel to choose it. A client must also create a socket if it wishes to communicate/connect with this server. This client may bind its socket to a server address, if it doesn't then it must use that server's socket address in all subsequent sendto() and recvfrom() calls.
```C
int socket(int domain, int type, int protocol); // definition in socket.h

int tcp_sock = socket(AF_INET, SOCK_STREAM, 0); // creates a TCP socket for send/recv
```
This socket must be bound with a host address and port number once created via the bind system call:
```C
int bind(int sockfd, struct sockaddr *addr, socklen_t addrlen)
```
For TCP connections it must be bound to the server host address first. addrlen is the size in bytes of the address structure pointed to by addr. Once this connection is established a TCP server must use listen() and accept() to accept connections from client machines:
```C
int listen(int sockfd, int backlog); // allows for incoming connections on sockfd, backlog is the max queue size for pending connections
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); // accept a socket
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen); // connect that socket
```
The accept syscall takes the first socket in the pending queue and creates a new connected socket, returning the file descriptor to said socket. When accepting a new socket, the TCP server blocks until the connection is made via the connect syscall. This call connects sockfd to the address addr.

```C
ssize_t write(sockfd, void *buf, size_t, len);
ssize_t read(sockfd, void *buf, size_t len);

// Same but with optional flag parameters (can be 0 in simple cases)
ssize_t send(int sockfs, const void *buf, size_t len, int flags);
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

