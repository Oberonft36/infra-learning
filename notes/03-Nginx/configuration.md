# Nginx Configuration Loading and Reload

## Goal

Build the model:

```text
Configuration file
    ↓
Master process
    ↓
Worker process
    ↓
HTTP service
```

This note only covers configuration loading and reload. It does not cover reverse proxy, location, upstream, or HTTPS.

## Core Question

After Nginx is started, how does the master process know which port to listen on, where the web root is, where logs are written, and how many workers should be started?

The answer is configuration.

```text
Nginx program
    ↓
reads configuration files
    ↓
creates HTTP service according to configuration
```

Program is code. Configuration is runtime rule.

This is a common Linux service design:

```text
sshd -> sshd_config
MySQL -> my.cnf
Redis -> redis.conf
Nginx -> nginx.conf
```

The same idea appears later in Docker, Kubernetes, and Prometheus:

```text
Container -> config
Pod -> YAML
Prometheus -> prometheus.yml
```

## Important Paths

Main configuration entry:

```text
/etc/nginx/nginx.conf
```

Site templates:

```text
/etc/nginx/sites-available/
```

Enabled sites:

```text
/etc/nginx/sites-enabled/
```

Default log directory:

```text
/var/log/nginx/
```

Important distinction:

```text
sites-available != sites-enabled
```

On Ubuntu, `sites-enabled` usually contains symbolic links pointing to files in `sites-available`.

This is similar to systemd enable behavior:

```text
enable service
    ↓
target.wants
    ↓
symbolic link
```

## Configuration Loading Flow

```text
systemctl start nginx
    ↓
systemd
    ↓
starts nginx program
    ↓
master process
    ↓
reads nginx.conf
    ↓
reads included files
    ↓
builds final configuration
    ↓
creates listen socket
    ↓
forks worker processes
```

Key point:

```text
The master process reads configuration.
Workers do not independently read configuration.
```

## include

The main configuration can load child configuration files through `include`.

Example:

```nginx
include /etc/nginx/sites-enabled/*;
```

The model is:

```text
main configuration
    ↓
include directive
    ↓
child configuration files
```

This pattern appears in many infrastructure systems.

## Why Configuration Changes Do Not Take Effect Immediately

Changing a file on disk does not automatically change the configuration already loaded by the running master process.

```text
configuration file on disk
    ↓
was already read by master process
    ↓
change requires reload or restart
```

Therefore:

```text
editing configuration != applying configuration
```

## nginx -t

`nginx -t` checks configuration syntax.

It does not reload Nginx, does not start workers, and does not replace the currently running configuration.

```text
nginx -t = configuration health check
```

## Reload vs Restart

Restart model:

```text
stop old process
    ↓
create new process
```

Reload model:

```text
systemd
    ↓
notifies master process
    ↓
master rereads configuration
    ↓
new workers are created
    ↓
old workers finish existing connections
    ↓
old workers exit
```

Key point:

```text
Reload != Restart
```

In reload, the master process does not exit.

## Troubleshooting Model Upgrade

Old model:

```text
Service -> Process -> Port -> HTTP -> Logs
```

New model:

```text
Service -> Process -> Config -> Port -> HTTP -> Logs
```

Before restarting repeatedly, check configuration syntax first.

## Experiment Commands

```bash
ls -l /etc/nginx
ls -l /etc/nginx/sites-enabled
ls -l /etc/nginx/sites-enabled/default
ls -l /etc/nginx/sites-available
grep include /etc/nginx/nginx.conf
sudo nginx -t
```

## Understanding Questions

1. Why does changing `nginx.conf` not take effect immediately?
2. What does `nginx -t` do? Does it start or reload Nginx?
3. Why does Ubuntu use both `sites-available` and `sites-enabled`?
4. Who reads `nginx.conf`: master, worker, or systemd?

## Engineering Meaning

This lesson is not only about Nginx. It builds a general infrastructure model:

```text
program
    ↓
configuration
    ↓
runtime behavior
```

This model will appear again in Docker, Kubernetes, Prometheus, and AI infrastructure.
