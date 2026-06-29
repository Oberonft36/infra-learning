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
Master process creates a listen socket on port 80
    ↓
Master process creates worker processes
    ↓
Worker processes wait for new connections
```

## Request Handling Flow

```text
curl sends a TCP connection
    ↓
Kernel receives the connection on the listen socket
    ↓
Kernel gives the connection to one Nginx worker process
    ↓
Worker parses the HTTP request
    ↓
Worker returns the HTTP response
```

## Key Distinctions

| Concept | Role |
|---|---|
| systemctl | User command-line client |
| systemd | Service manager |
| nginx.service | Unit file used by systemd |
| /usr/sbin/nginx | Nginx executable program |
| Master process | Reads config and manages workers |
| Worker process | Handles client connections and HTTP processing |
| Listen socket | Kernel object bound to a port |
| Port 80 | Default HTTP port |

## Engineering Meaning

This model connects Linux service management, process management, socket, port, TCP, HTTP, and Nginx architecture.

It is a bridge from Linux and networking to Docker and Kubernetes.
