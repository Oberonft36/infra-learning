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
