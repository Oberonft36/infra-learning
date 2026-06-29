# Recording Skill

This document defines how to decide whether a learning point should be recorded and where it should be placed in this repository.

---

## Purpose

The repository is an infrastructure learning knowledge base.

Recording should support long-term engineering understanding, troubleshooting ability, experiments, and technical asset accumulation.

Recording should not interrupt the normal learning flow.

---

## Reminder Rule

During learning, recording reminders should be short and low-frequency.

Use a reminder only when the current content is worth becoming a long-term technical asset.

The reminder format should be:

```text
Recording hint: this point is worth recording in <path>, because <reason>.
```

Do not stop the lesson just to write notes unless the learner explicitly asks to pause.

---

## What Should Be Recorded

Record the following types of content:

1. Essence definitions
2. Architecture relationships
3. Service/process/network lifecycle diagrams
4. Experiments that can be reproduced
5. Troubleshooting cases
6. Easy-to-confuse concepts
7. Personal understanding that is not copied from documentation
8. Commands that are frequently used in debugging

---

## What Should Not Be Recorded

Do not record the following by default:

1. Long copied documentation
2. Full command manuals
3. Basic commands already mastered
4. Large screenshots without explanation
5. Temporary chat content without long-term value

---

## Directory Mapping

### Core Maps

Use this directory for global knowledge maps and long-term navigation:

```text
notes/00-Core/
```

Examples:

- learning roadmap
- infrastructure knowledge map
- request lifecycle
- service to process relationship

---

### Linux Knowledge

```text
notes/01-Linux/
```

Use for Linux concepts such as:

- process
- signal
- systemd
- service
- journalctl
- permission

---

### Network Knowledge

```text
notes/02-Network/
```

Use for networking concepts such as:

- IP
- DNS
- port
- TCP
- HTTP
- HTTPS
- TLS
- NAT

---

### Nginx Knowledge

```text
notes/03-Nginx/
```

Use for Nginx concepts such as:

- master process
- worker process
- reverse proxy
- load balancing
- configuration
- logs

---

### Docker Knowledge

```text
notes/04-Docker/
```

Use for Docker concepts such as:

- image
- container
- Dockerfile
- volume
- network
- container process model

---

### Kubernetes Knowledge

```text
notes/05-Kubernetes/
```

Use for Kubernetes concepts such as:

- Pod
- Deployment
- Service
- Ingress
- ConfigMap
- Secret
- kubelet
- CNI
- CoreDNS

---

### Distributed Systems

```text
notes/06-Distributed-System/
```

Use for distributed system concepts such as:

- CAP
- consistency
- consensus
- Raft
- replication
- sharding
- RPC
- gRPC

---

### Cloud Native

```text
notes/07-Cloud-Native/
```

Use for cloud native infrastructure topics such as:

- observability
- Prometheus
- Grafana
- OpenTelemetry
- service discovery
- health check
- autoscaling

---

### AI Infrastructure

```text
notes/08-AI-Infra/
```

Use for AI Infra and LLMOps topics such as:

- model serving
- vLLM
- Triton
- GPU
- CUDA
- KV Cache
- inference latency
- throughput

---

### Web3 Infrastructure

```text
notes/09-Web3-Infra/
```

Use only for Web3 infrastructure observation topics, not as the main path.

---

### Troubleshooting

```text
notes/10-Troubleshooting/
```

Use for real failure cases.

Each case should include:

- symptom
- context
- investigation
- root cause
- fix
- prevention

---

### Diagrams

```text
notes/11-Diagrams/
```

Use for architecture diagrams and lifecycle diagrams.

---

### Cheat Sheets

```text
notes/12-CheatSheet/
```

Use for short, high-density review documents.

---

### Labs

```text
labs/
```

Use for experiments, commands, configuration files, packet captures, logs, and outputs.

Examples:

```text
labs/linux/
labs/network/
labs/nginx/
labs/docker/
labs/kubernetes/
labs/scripts/
labs/capture/
```

---

### Templates

```text
templates/
```

Use for reusable document formats.

---

### Assets

```text
assets/
```

Use for images, screenshots, diagrams, and other static files.

---

## Teaching Integration Rule

Recording reminders should not slow down the lesson.

A good reminder is one sentence.

Example:

```text
Recording hint: this service-process relationship is worth recording in notes/00-Core/service-process.md because it will reappear in Nginx, Docker, and Kubernetes.
```

After the reminder, continue teaching normally.
