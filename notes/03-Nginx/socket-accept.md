# Nginx Socket and accept() Model

## Purpose

This note explains how an Nginx worker receives a TCP connection from the Linux kernel.

It focuses on this path:

```text
Nginx master
    ↓
listen socket
    ↓
fork workers
    ↓
worker accept()
    ↓
connected socket
    ↓
HTTP request / response
```

---

## Master Creates the Listen Socket

In the normal Nginx model, workers do not independently bind port 80.

```text
Nginx master
    ↓
reads nginx.conf
    ↓
creates listen socket through Kernel
    ↓
fork() workers
```

The listen socket is a kernel object. Port 80 is only part of the socket address.

---

## Workers Inherit the Listen Socket

After `fork()`, worker processes inherit the listen socket file descriptor.

```text
Master owns listen socket fd
    ↓
fork()
    ↓
Worker inherits listen socket fd
    ↓
Worker waits in accept()
```

Important correction:

```text
fork() inherits file descriptors.
fork() does not mean master and worker share one normal memory space.
```

---

## accept() Returns a Connected Socket

A worker blocked in `accept()` is waiting for the kernel to provide an established TCP connection.

```text
connection becomes established
    ↓
kernel wakes one worker
    ↓
accept() returns
    ↓
worker receives a connected socket fd
    ↓
worker handles HTTP
```

The fd returned by `accept()` is not the original listen socket fd.
It is a new fd for one client connection.

```text
listen socket fd
    ↓
accept()
    ↓
connected socket fd
    ↓
read request / write response
```

---

## Client Side Is Different

A client such as `curl` does not listen for incoming connections.
It actively creates a socket and connects to a server.

```text
curl
    ↓
socket()
    ↓
connect(target IP:Port)
    ↓
TCP connection established
    ↓
HTTP request bytes
```

DBus is not part of the normal `curl → TCP → Nginx` request path.
DBus belongs to service-management flows such as:

```text
systemctl
    ↓
DBus
    ↓
systemd
```

---

## Debugging Meaning

```text
systemctl status nginx
    checks service state

ps / pstree
    check process state

ss -tulnp
    checks kernel listen socket state

curl
    checks TCP plus HTTP behavior
```

This model is a bridge between Linux process management, TCP sockets, and Nginx request handling.
