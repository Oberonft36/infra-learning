# Nginx Master / Worker Process Model

## Essence

Nginx is started as a Linux service, but the real work is done by processes.

The basic relationship is:

```text
systemd
    ↓
nginx.service
    ↓
/usr/sbin/nginx
    ↓
Nginx master process
    ↓
Nginx worker process
    ↓
HTTP request / response
```

Key idea:

```text
Service describes how to start the program.
Program becomes process after running.
Process owns PID.
Kernel manages PID and socket.
Nginx worker handles HTTP traffic.
```

---

## Startup Flow

```text
systemctl start nginx
    ↓
systemctl notifies systemd
    ↓
systemd reads nginx.service
    ↓
systemd starts /usr/sbin/nginx
    ↓
Kernel creates the Nginx master process and assigns a PID
    ↓
Master process reads nginx.conf
    ↓
Master process creates the runtime environment
    ↓
Master process creates a listen socket according to the config
    ↓
Master process forks worker processes
    ↓
Worker processes inherit the listen socket
    ↓
Worker processes wait in accept() for new connections
```

---

## Configuration Drives Runtime

Nginx does not decide its runtime behavior only from program code.
It reads configuration first.

Typical configuration path on Ubuntu/Debian:

```text
/etc/nginx/nginx.conf
    ↓
include /etc/nginx/sites-enabled/*;
    ↓
/etc/nginx/sites-available/default
```

The master process reads the configuration and then decides:

```text
which port to listen on
which address to bind
how many workers to create
where logs should be written
which document root should be used
```

Important distinction:

```text
systemd reads nginx.service.
Nginx master reads nginx.conf.
Worker processes do not independently parse nginx.conf during normal startup.
```

---

## Listen Socket Model

A port is only a number.
The real kernel object used for listening is a socket.

```text
IP address
    +
Port
    ↓
Listen Socket
```

Example:

```text
0.0.0.0
    +
80
    ↓
Listen Socket bound to 0.0.0.0:80
```

The listen socket is not a disk file and not a systemd unit.
It is a kernel object created through system calls such as:

```text
socket()
    ↓
bind()
    ↓
listen()
```

In the normal Nginx master/worker model:

```text
Master process
    ↓
reads nginx.conf
    ↓
asks Kernel to create listen socket
    ↓
fork() workers
    ↓
workers inherit the listen socket
```

---

## Why Workers Do Not Each Create Their Own Listen Socket

A common misunderstanding is:

```text
Worker1 listens on 80
Worker2 listens on 80
Worker3 listens on 80
```

This is not the normal model.

If one process has already bound a listen socket to port 80, another process cannot normally bind the same IP:port again.
The kernel would reject it with an error similar to:

```text
Address already in use
```

Therefore the normal model is:

```text
Master creates one listen socket
    ↓
Master forks multiple workers
    ↓
Workers share the inherited listen socket
```

Special mechanisms such as `SO_REUSEPORT` exist, but they are not the basic model at this stage.

---

## fork() and Socket Inheritance

`fork()` is important because child processes inherit the file descriptors opened by the parent process.

For Nginx:

```text
Master process already owns the listen socket
    ↓
fork()
    ↓
Worker process inherits the same listen socket
```

This means workers do not need to create the listen socket again.
They can directly wait for incoming connections through the inherited socket.

Important correction:

```text
fork() inherits file descriptors.
fork() does not mean master and worker share one normal memory space.
```

For Nginx, the key inherited object is the file descriptor that refers to the listen socket.

---

### Kernel Object Behind the Inherited Socket FD

The inherited listen socket fd points to kernel-managed socket objects.

```text
worker fd
    ↓
struct file
    ↓
struct socket
    ↓
struct sock
```

For Nginx learning, the key point is:

```text
Workers inherit the fd.
Kernel owns the socket and TCP state.
Workers call accept() to receive connected socket fds.
```

For the deeper kernel structure, see:

```text
notes/02-Network/socket-kernel-structure.md
```

---

## accept() Model

Workers wait for new TCP connections by calling `accept()`.

Conceptually:

```text
Worker:
"Kernel, is there a new connection?"
    ↓
Kernel:
"Yes. Here is one established connection."
    ↓
Worker handles the connection
```

Multiple workers may be waiting on the same listen socket:

```text
Worker1 → accept()
Worker2 → accept()
Worker3 → accept()
```

When a new client connection arrives:

```text
TCP SYN
    ↓
Kernel
    ↓
Listen Socket
    ↓
Kernel wakes one worker
    ↓
accept() returns
    ↓
That worker handles the connection
```

The worker selection is not done by systemd.
It is not manually done by the Nginx master for each request.
At this level of abstraction, the kernel wakes one worker waiting on `accept()`.

---

## Listen FD vs Connected FD

The listen socket is used to receive new connections.

When `accept()` succeeds, it returns a new file descriptor.
That new descriptor represents one established client connection.

```text
listen socket fd
    ↓
accept()
    ↓
connected socket fd
    ↓
read HTTP request / write HTTP response
```

So the worker does not use the listen socket itself to exchange HTTP data with every client.
It uses a connected socket returned by `accept()`.

---

## Request Handling Flow

```text
curl sends a TCP connection
    ↓
Kernel receives the connection on the listen socket
    ↓
Kernel wakes one Nginx worker waiting in accept()
    ↓
accept() returns an established connection to that worker
    ↓
Worker parses the HTTP request
    ↓
Worker returns the HTTP response
```

Expanded flow:

```text
curl http://127.0.0.1
    ↓
connect()
    ↓
TCP SYN reaches Kernel
    ↓
Kernel finds the listen socket for 127.0.0.1:80
    ↓
TCP connection is established
    ↓
Kernel gives the accepted connection to one worker
    ↓
Worker reads HTTP Request
    ↓
Worker writes HTTP Response
```

---

## Key Distinctions

| Concept | Role |
|---|---|
| systemctl | User command-line client |
| systemd | Service manager |
| nginx.service | Unit file used by systemd |
| /usr/sbin/nginx | Nginx executable program |
| Master process | Reads config, creates listen socket, forks and manages workers |
| Worker process | Waits on accept(), handles client connections and HTTP processing |
| Listen socket | Kernel object bound to an IP:port and used to receive new TCP connections |
| Connected socket | Kernel object representing one established TCP connection |
| Port 80 | Default HTTP port number |
| Kernel | Owns socket state, TCP state, PID allocation, and connection delivery |
| accept() | Operation workers use to receive established connections from the listen socket |

---

## Troubleshooting View

When debugging Nginx, separate the layers:

```text
Service layer:
systemctl status nginx

Process layer:
ps -ef | grep nginx
pstree -p | grep nginx

Config layer:
sudo nginx -t

Socket / network layer:
ss -tulnp | grep :80

HTTP layer:
curl http://127.0.0.1

Log layer:
journalctl -u nginx
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Interpretation:

```text
systemctl active does not necessarily prove port 80 is listening.
ps shows whether master/worker processes exist.
ss shows whether the kernel has a listen socket.
curl shows whether TCP and HTTP work together.
logs show service lifecycle and Nginx runtime errors.
```

---

## Engineering Meaning

This model connects Linux service management, process management, socket, port, TCP, HTTP, and Nginx architecture.

It is a bridge from Linux and networking to Docker and Kubernetes.

The key long-term infrastructure model is:

```text
Process
    ↓
Socket
    ↓
Port
    ↓
TCP connection
    ↓
Application protocol
```

In later stages, the same idea will reappear in Docker containers, Kubernetes Pods, Services, Ingress, and AI model serving gateways.
