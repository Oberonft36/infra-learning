# Linux Socket Kernel Structure: `struct socket` vs `struct sock`

This note explains the internal kernel-side socket model behind Linux network programming.

It connects Linux process, file descriptor, socket, TCP, Nginx, Docker, Kubernetes, and AI model serving.

---

## Core Model

Linux separates the generic socket interface from protocol-specific state.

```text
User process
    ↓
file descriptor
    ↓
struct file
    ↓
struct socket
    ↓
struct sock
    ↓
TCP case: struct tcp_sock
```

Core idea:

```text
struct socket answers which generic socket operation should be dispatched.
struct sock stores protocol-specific state.
struct tcp_sock stores TCP-specific state.
```

---

## Purpose

This file explains what is below the word "socket" in later notes about TCP, Nginx, Docker, Kubernetes, and AI Infrastructure.
