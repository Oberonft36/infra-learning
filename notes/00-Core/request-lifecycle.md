# Request Lifecycle

This document tracks how a user request flows through infrastructure layers.

---

## Current Basic Model

```text
Client
    ↓
DNS
    ↓
TCP Connection
    ↓
HTTP Request
    ↓
Server Process
    ↓
HTTP Response
```

---

## Kernel / Socket Model

Before a request reaches a user-space process, it first enters the Linux kernel networking stack.

```text
Client
    ↓
TCP connection attempt
    ↓
Kernel
    ↓
Listen Socket
    ↓
accept()
    ↓
Server Process
```

Key distinction:

```text
Port is a number.
Socket is the kernel object bound to an IP:port.
Process is the user-space program instance that handles application logic.
```

For a server process, the basic path is:

```text
IP + Port
    ↓
Listen Socket
    ↓
Kernel waits for new TCP connections
    ↓
Server process calls accept()
    ↓
Kernel returns an established connection
```

---

### Kernel Object Path Behind a Socket

When a process uses a socket fd, the request path enters several kernel-side objects.

```text
process read/write/send/recv
    ↓
file descriptor
    ↓
struct file
    ↓
struct socket
    ↓
struct sock / tcp_sock
    ↓
TCP/IP stack
```

This means the application does not directly manage TCP sequence state, window state, protocol buffers, or connection queues.

The process uses a file descriptor.

The kernel manages the socket and protocol state.

See:

```text
notes/02-Network/socket-kernel-structure.md
```

---

## Client System Call Path

For a client such as `curl`, the request starts in user space and enters the kernel through system calls.

```text
curl application
    ↓
socket()
    ↓
Kernel creates a socket object and returns fd
    ↓
connect(fd, target IP:Port)
    ↓
Kernel starts TCP connection establishment
    ↓
TCP connection becomes ESTABLISHED
    ↓
curl sends HTTP request bytes
```

Important distinction:

```text
systemctl uses DBus to talk to systemd.
curl does not use DBus for networking.
curl uses socket-related system calls to enter the kernel networking stack.
```

---

## Server Listen Socket Queues

A server listen socket has listening state after `listen()`.

Simplified lifecycle:

```text
socket()
    ↓
bind()
    ↓
listen()
    ↓
accept()
```

In a simplified TCP model, the listen socket is associated with two queues:

```text
SYN queue:
connections that have started but not fully completed the TCP handshake

Accept queue:
connections that have completed the TCP handshake and are waiting for accept()
```

Connection establishment path:

```text
client sends SYN
    ↓
server kernel records half-open state
    ↓
client completes handshake
    ↓
connection enters accept queue
    ↓
kernel wakes a process blocked in accept()
    ↓
accept() returns a new connected socket fd
```

The listen socket remains responsible for receiving new connections.
The connected socket fd is used for one established client connection.

---

## Packet to Socket Lookup

When TCP traffic reaches the host, the kernel checks socket state before handing anything to an application process.

Simplified lookup model:

```text
TCP traffic reaches host
    ↓
Kernel checks destination IP, destination port, protocol, source IP, source port
    ↓
If it belongs to an existing established connection:
    deliver it to the connected socket
    ↓
Otherwise, if it is a new connection attempt and a listen socket matches local IP:Port:
    perform TCP connection establishment
    ↓
After establishment, place the connection into the accept queue
```

A listen socket bound to `0.0.0.0:80` can match local IPv4 traffic to port 80.
A listen socket bound to a specific local address only matches that address and port.

Key distinction:

```text
Established TCP traffic goes to a connected socket.
New TCP connection attempts are matched against a listen socket.
```

---

## Nginx Model

```text
Client
    ↓
DNS
    ↓
TCP Connection
    ↓
Kernel
    ↓
Nginx Listen Socket
    ↓
Nginx Worker Process
    ↓
HTTP Request
    ↓
HTTP Response
```

The master process manages workers. The worker process handles real HTTP traffic.

More accurate Nginx flow:

```text
systemd
    ↓
nginx.service
    ↓
/usr/sbin/nginx
    ↓
Nginx master process
    ↓
Master reads nginx.conf
    ↓
Master creates listen socket through Kernel
    ↓
Master fork()s workers
    ↓
Workers inherit the listen socket
    ↓
Workers wait in accept()
```

Then a client request arrives:

```text
Client sends request
    ↓
Kernel finds the listen socket
    ↓
TCP connection is established
    ↓
Connection waits in the accept queue
    ↓
Kernel wakes one worker waiting on accept()
    ↓
Worker receives a connected socket fd
    ↓
Worker parses HTTP Request
    ↓
Worker returns HTTP Response
```

Key point:

```text
systemd starts and supervises nginx.service.
Nginx master prepares runtime state and manages workers.
Kernel owns socket and TCP connection state.
Worker processes accept connections and handle HTTP.
```

---

## Request Debugging Checkpoints

```text
1. Service layer
   systemctl status nginx

2. Process layer
   ps -ef | grep nginx
   pstree -p | grep nginx

3. Configuration layer
   sudo nginx -t

4. Socket / network layer
   ss -tulnp | grep :80

5. HTTP layer
   curl http://127.0.0.1

6. Log layer
   journalctl -u nginx
   /var/log/nginx/access.log
   /var/log/nginx/error.log
```

Interpretation:

```text
systemctl checks service state.
ps and pstree check process existence and parent-child relationship.
nginx -t checks configuration validity.
ss checks whether the kernel has a listen socket.
curl checks whether TCP and HTTP work together.
logs provide lifecycle and runtime evidence.
```

---

## Future Kubernetes Model

```text
Client
    ↓
DNS
    ↓
Ingress
    ↓
Service
    ↓
Pod
    ↓
Container
    ↓
Process
    ↓
Response
```

The Nginx model is important because Kubernetes traffic delivery still eventually reaches a process through kernel networking and sockets.

---

## Purpose

This file answers: what happens after a user sends a request?

Current learning focus:

```text
Linux Process
    ↓
Socket
    ↓
Port
    ↓
TCP
    ↓
HTTP
    ↓
Nginx Worker
```
