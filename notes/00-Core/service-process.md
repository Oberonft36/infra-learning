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
