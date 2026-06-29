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

## Nginx Model

```text
Client
    ↓
DNS
    ↓
TCP Connection
    ↓
Nginx Worker Process
    ↓
HTTP Request
    ↓
HTTP Response
```

The master process manages workers. The worker process handles real HTTP traffic.

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

---

## Purpose

This file answers: what happens after a user sends a request?
