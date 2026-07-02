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

## Why the Kernel Splits Socket into Layers

Linux needs to support different communication protocols such as TCP, UDP, and Unix Domain Socket.

Applications should not need a completely different user-space interface for every protocol.

The kernel therefore separates:

```text
Generic socket interface
    ↓
Protocol-specific implementation
```

This is an example of interface and implementation separation.

The application sees a stable interface:

```text
socket()
connect()
send()
recv()
read()
write()
```

The kernel maps those operations to the correct protocol behavior.

---

## `struct socket`

For infrastructure learning, `struct socket` should be understood as the protocol-independent socket interface layer.

It connects user-visible socket operations to the correct kernel implementation.

Conceptually, it answers:

```text
What kind of socket is this?
Which operation table should handle this call?
Where is the protocol-specific state?
```

Important fields at the conceptual level:

| Concept | Meaning |
|---|---|
| type | Socket type, such as stream or datagram |
| state | Coarse socket-level state |
| ops | Operation table used for dispatch |
| pointer to sock | Link to protocol-specific state |

---

## `struct sock`

`struct sock` is the protocol-specific state layer.

In the TCP case, TCP-specific state is represented by TCP-specific structures such as `struct tcp_sock`.

For TCP, this layer conceptually contains:

```text
connection identity
connection state
send buffer
receive buffer
sequence and acknowledgement state
window state
congestion-control state
```

For UDP, the protocol-specific state is different because UDP does not have a TCP-style connection state machine.

Therefore:

```text
TCP and UDP share the generic socket interface.
TCP and UDP do not share the same detailed protocol state model.
```

---

## File Descriptor Path

A process does not directly hold the kernel TCP state.

It holds a file descriptor.

```text
process
    ↓
fd
    ↓
struct file
    ↓
struct socket
    ↓
struct sock
```

The fd is the user-space handle.

The kernel objects below the fd contain the socket and protocol state.

---

## Purpose

This file explains what is below the word "socket" in later notes about TCP, Nginx, Docker, Kubernetes, and AI Infrastructure.
