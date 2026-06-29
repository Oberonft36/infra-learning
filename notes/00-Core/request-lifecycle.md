# Request Lifecycle

This document tracks how a user request flows through infrastructure layers.

---

## Current Basic Model

```text
Browser
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
Browser
    ↓
DNS
    ↓
TCP
    ↓
HTTP
    ↓
Nginx
    ↓
Upstream Application
    ↓
Response
```

---

## Future Kubernetes Model

```text
Browser
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

This file answers one question:

```text
What happens after a user sends a request?
```
