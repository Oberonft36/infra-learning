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
Kernel wakes one worker waiting on accept()
    ↓
Worker receives the connection
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
