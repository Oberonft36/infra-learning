# TCP

TCP connects network knowledge with socket, port, process, and HTTP.

## Essence

TCP provides reliable, ordered, bidirectional communication between two endpoints.

For infrastructure learning, TCP should not be studied as an isolated protocol. It should be connected with Linux process, socket, port, and application protocol.

```text
Process
    ↓
Socket
    ↓
IP + Port
    ↓
TCP Connection
    ↓
Application Protocol
```

Key idea:

```text
IP decides which host.
Port helps identify which service endpoint on that host.
Socket is the kernel object that represents communication state.
Process is the user-space program instance that handles application logic.
```

---

## Client-Side Socket Lifecycle

For a client program such as `curl`, the simplified lifecycle is:

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
Client-side TCP socket becomes connected
    ↓
curl sends HTTP request bytes
```

Important distinction:

```text
Client socket:
socket() → connect() → connected TCP socket

Client does not have a listen queue.
Client actively connects to a server address.
```

`DBus` is not involved in normal client networking.

DBus belongs to service-management flows such as:

```text
systemctl
    ↓
DBus
    ↓
systemd
```

Normal network I/O uses socket-related system calls into the kernel networking stack:

```text
curl
    ↓
socket() / connect() / read() / write()
    ↓
System Call
    ↓
Kernel TCP/IP stack
```

---

## Server-Side Listen Socket Lifecycle

For a server program such as Nginx, the simplified lifecycle is:

```text
socket()
    ↓
create socket object
    ↓
bind()
    ↓
bind local IP:Port
    ↓
listen()
    ↓
enter listening state
    ↓
accept()
    ↓
return one established connection as a new fd
```

Important distinction:

```text
Listen socket fd:
used to receive new TCP connections

Connected socket fd:
used to read and write data for one established TCP connection
```

`accept()` does not return the original listen socket fd.

It returns a new file descriptor representing one established client connection.

```text
listen socket fd
    ↓
accept()
    ↓
connected socket fd
    ↓
read application data / write application data
```

---

## Queues Behind `listen()`

After a server calls `listen()`, the TCP listening state is associated with connection queues.

Simplified model:

```text
Half-open queue:
connections that have started the TCP handshake but are not fully established yet

Accept queue:
connections that have completed the TCP handshake and are waiting for accept()
```

Connection establishment path:

```text
client starts TCP handshake
    ↓
kernel records temporary connection state
    ↓
handshake completes
    ↓
connection enters accept queue
    ↓
kernel wakes a process blocked in accept()
    ↓
accept() returns a connected socket fd
```

This means:

```text
TCP connection establishment is handled by the kernel.
Application process receives the connection after it is ready to be accepted.
```

---

## Packet to Socket Lookup Model

When TCP traffic reaches the host, the kernel checks socket state before any user-space application receives data.

Simplified lookup model:

```text
TCP traffic reaches host
    ↓
Kernel checks:
- destination IP
- destination port
- protocol
- source IP
- source port
    ↓
If the packet belongs to an existing connection:
    deliver it to the connected socket
    ↓
Otherwise, if it is a new connection attempt and a listen socket matches local IP:Port:
    establish the connection
    ↓
After establishment, place it into the accept queue
```

A listen socket bound to:

```text
0.0.0.0:80
```

can match local IPv4 traffic sent to port 80.

A listen socket bound to:

```text
192.168.1.10:80
```

only matches that specific local address and port.

Key distinction:

```text
Established TCP traffic goes to a connected socket.
New TCP connection attempts are matched against a listen socket.
```

---

## Relationship to Nginx

Nginx uses this model as follows:

```text
Nginx master
    ↓
reads nginx.conf
    ↓
creates listen socket
    ↓
fork() workers
    ↓
workers inherit listen socket fd
    ↓
workers wait in accept()
    ↓
Kernel wakes one worker when a connection is ready
    ↓
accept() returns connected socket fd
    ↓
worker handles HTTP request / response
```

This explains why:

```bash
ss -tulnp
```

is a socket-level verification command.

It checks whether the kernel has a listening socket.

It does **not** prove that HTTP application logic is correct.

---

## Troubleshooting Meaning

When debugging TCP-level server problems:

```text
Service layer:
systemctl status nginx

Process layer:
ps -ef | grep nginx
pstree -p | grep nginx

Socket layer:
ss -tulnp | grep :80

HTTP layer:
curl http://127.0.0.1

Log layer:
journalctl -u nginx
/var/log/nginx/error.log
```

Interpretation:

```text
systemctl checks service lifecycle.
ps checks process existence.
ss checks listen socket state.
curl checks TCP plus HTTP behavior.
logs provide lifecycle and runtime evidence.
```

---

## Long-Term Infrastructure Meaning

This TCP model will reappear in later stages:

```text
Docker container networking
Kubernetes Service
Kubernetes Ingress
kube-proxy
CNI
Envoy / Service Mesh
AI model serving gateway
```

The core model remains:

```text
Client
    ↓
TCP
    ↓
Kernel Socket
    ↓
Process
    ↓
Application Protocol
```
