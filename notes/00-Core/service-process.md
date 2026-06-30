# Service → Process

This document explains how Linux services become running processes.

---

## Core Relationship

```text
systemd
    ↓
Service
    ↓
Process
    ↓
PID
    ↓
Socket
    ↓
Port
```

---

## Nginx Example

```text
systemd
    ↓
nginx.service
    ↓
Nginx master process
    ↓
nginx.conf
    ↓
Listen socket
    ↓
Nginx worker process
    ↓
HTTP port
```

Key point:

```text
Service is the management unit.
Process is the real running entity.
Master process reads configuration.
Worker process handles real client connections.
```

---

## Nginx Master / Worker / Socket Detail

A more precise Nginx startup model is:

```text
systemctl start nginx
    ↓
systemd reads nginx.service
    ↓
systemd starts /usr/sbin/nginx
    ↓
Kernel creates the Nginx master process and assigns PID
    ↓
Master reads nginx.conf
    ↓
Master prepares the runtime environment
    ↓
Master creates the listen socket through socket(), bind(), listen()
    ↓
Master fork()s worker processes
    ↓
Workers inherit the listen socket file descriptor
    ↓
Workers wait in accept()
```

Important distinction:

```text
systemd manages the service lifecycle.
Master manages Nginx runtime and workers.
Kernel owns PID, socket, and TCP state.
Workers accept established connections and handle HTTP.
```

`fork()` inheritance here mainly means file descriptor inheritance.
It does not mean the master and workers share one normal memory space.

---

## Configuration to Runtime Model

Many infrastructure components follow the same lifecycle.

```text
Program
    ↓
Manager process or daemon
    ↓
Read configuration
    ↓
Build runtime environment
    ↓
Create runtime resources
    ↓
Start worker processes or child processes
```

Core idea:

```text
Configuration does not execute itself.
A manager process reads configuration and builds the runtime environment.
```

This model prepares future learning of containerd, kubelet, Prometheus, CoreDNS, and other infrastructure components.

---

## Future Docker Extension

```text
systemd
    ↓
dockerd
    ↓
Container
    ↓
Process
```

---

## Future Kubernetes Extension

```text
systemd
    ↓
kubelet
    ↓
Pod
    ↓
Container
    ↓
Process
```

---

## Purpose

This file answers one question:

```text
Who starts whom? Who manages whom?
```
