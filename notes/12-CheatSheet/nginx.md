# Nginx CheatSheet

## Service Management

```bash
# Check service status
systemctl status nginx

# Start
sudo systemctl start nginx

# Stop
sudo systemctl stop nginx

# Restart
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx
```

---

## Process

```bash
# View Nginx processes
ps -ef | grep nginx

# View process tree
pstree -p
```

---

## Configuration

Main configuration:

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

Logs:

```text
/var/log/nginx/
```

Common commands:

```bash
# View configuration directory
ls -l /etc/nginx

# View enabled sites
ls -l /etc/nginx/sites-enabled

# View available sites
ls -l /etc/nginx/sites-available

# View symbolic link
ls -l /etc/nginx/sites-enabled/default

# Find include directives
grep include /etc/nginx/nginx.conf

# View main configuration
cat /etc/nginx/nginx.conf
```

---

## Configuration Validation

```bash
# Check configuration syntax
sudo nginx -t
```

Expected output:

```text
syntax is ok
test is successful
```

Remember:

```text
nginx -t

=

Configuration syntax check

≠

Reload

≠

Restart
```

---

## Network

```bash
# Check listening ports
ss -tulnp

# Test HTTP locally
curl http://127.0.0.1

# Test another port
curl http://127.0.0.1:8080
```

---

## Logs

```bash
# systemd log
journalctl -u nginx

# Follow system log
journalctl -u nginx -f

# Access log
tail -f /var/log/nginx/access.log

# Error log
tail -f /var/log/nginx/error.log
```

---

## Troubleshooting Workflow

```text
① Service
        │
        ▼
systemctl status nginx

        │
        ▼
② Process
        │
        ▼
ps -ef | grep nginx

        │
        ▼
③ Config
        │
        ▼
sudo nginx -t

        │
        ▼
④ Port
        │
        ▼
ss -tulnp

        │
        ▼
⑤ HTTP
        │
        ▼
curl http://127.0.0.1

        │
        ▼
⑥ Logs
        │
        ▼
journalctl -u nginx
tail -f /var/log/nginx/error.log
```

---

## Configuration Loading Model

```text
systemctl start nginx
        │
        ▼
systemd
        │
        ▼
nginx Program
        │
        ▼
Master Process
        │
        ▼
Read nginx.conf
        │
        ▼
Read include files
        │
        ▼
Create Listen Socket
        │
        ▼
Fork Worker Process
        │
        ▼
HTTP Service
```

---

## Reload vs Restart

```text
Reload

Master
    │
    ▼
Reload configuration
    │
    ▼
Create new Workers
    │
    ▼
Old Workers finish existing requests
    │
    ▼
Old Workers exit

Master does NOT exit.
```

```text
Restart

Stop all processes
        │
        ▼
Start a new Master
        │
        ▼
Create new Workers
```

---

## Key Relationships

```text
Program
        │
        ▼
Configuration
        │
        ▼
Master Process
        │
        ▼
Worker Process
        │
        ▼
HTTP Service
```
