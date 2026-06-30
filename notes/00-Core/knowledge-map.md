# Infrastructure Knowledge Map

This document connects all infrastructure topics into one knowledge graph.

---

## Initial Map

```text
Linux
    ↓
Process
    ↓
Socket
    ↓
Port
    ↓
TCP
    ↓
HTTP
    ↓
Nginx
    ↓
Docker Container
    ↓
Kubernetes Pod
    ↓
Service / Ingress
    ↓
Cloud Native Infrastructure
    ↓
AI Infrastructure / LLMOps
```

---

## Nginx Socket Extension

Nginx connects Linux process, socket, TCP, and HTTP models into a single infrastructure execution path.

```text
systemd
    ↓
nginx.service
    ↓
Nginx master process
    ↓
listen socket
    ↓
fork worker processes
    ↓
accept()
    ↓
connected socket
    ↓
HTTP request / response
```

### Key Relationship

```text
Master prepares runtime resources.
Worker handles client connections.
Kernel owns socket and TCP state.
```

This execution model extends the core Linux concepts of **Service → Process → Socket → TCP → HTTP** into a complete request-processing pipeline.

The same architectural pattern will reappear throughout later infrastructure technologies, including:

```text
Docker container networking
Kubernetes Service
Kubernetes Ingress
Service Mesh (Envoy)
AI model serving gateways
```

Understanding this model provides the conceptual foundation for how modern infrastructure components receive network traffic, manage runtime resources, and deliver application requests.

---

## Purpose

This file answers one question:

```text
How are all topics connected?
```

Every new topic should be attached to this map instead of being learned as an isolated concept.
