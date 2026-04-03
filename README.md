# Traefik v3 - Reverse Proxy for Docker on a Single VPS

A production-ready Traefik v3 setup for running multiple web applications on a single VPS. Handles SSL termination, HTTP→HTTPS redirects, and automatic Let's Encrypt certificates for all your Docker containers.

## What This Does

- Routes traffic to multiple Docker containers by domain name
- Automatically issues and renews SSL certificates via Let's Encrypt
- Forces HTTPS on all traffic
- Provides a password-protected dashboard for monitoring
- Supports Nextcloud CalDAV/CardDAV redirects out of the box

## Prerequisites

- Docker and Docker Compose installed
- A server with ports 80 and 443 open
- Domain names pointing to your server's IP

## Setup

### 1. Create the shared Docker network

All containers that Traefik routes to must be on this network:

```bash
docker network create web
```

### 2. Configure Traefik

Edit `traefik.toml` and set your email address for Let's Encrypt:
```toml
email = "your@email.com"
```

Edit `traefik_dynamic.toml` and set:
- Your dashboard domain (`monitor.yourdomain.com`)
- A hashed password for dashboard access

### 3. Generate dashboard password

```bash
# Install htpasswd
sudo apt install apache2-utils

# Generate hashed password
htpasswd -nb admin yourpassword
```

Paste the output into `traefik_dynamic.toml`:
```toml
users = [
  "admin:$apr1$your_hash_here"
]
```

> **Note:** Dollar signs (`$`) do not need escaping in `.toml` files.

### 4. Set up acme.json

```bash
touch acme.json
chmod 600 acme.json
```

This file stores your Let's Encrypt certificates. It is gitignored and must never be committed.

### 5. Start Traefik

```bash
docker compose up -d
```

## Adding a New Application

Add these labels to any Docker Compose service to route it through Traefik:

### Basic HTTPS app

```yaml
services:
  myapp:
    image: myapp:latest
    networks:
      - web
      - internal
    labels:
      - traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)
      - traefik.http.routers.myapp.tls=true
      - traefik.http.routers.myapp.tls.certresolver=lets-encrypt

networks:
  web:
    external: true
  internal:
    external: false
```

### App on a non-standard port

```yaml
labels:
  - traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)
  - traefik.http.routers.myapp.tls=true
  - traefik.http.routers.myapp.tls.certresolver=lets-encrypt
  - traefik.http.services.myapp.loadbalancer.server.port=8080
```

### WWW to non-WWW redirect

```yaml
labels:
  - traefik.http.routers.myapp.rule=Host(`yourdomain.com`) || Host(`www.yourdomain.com`)
  - traefik.http.routers.myapp.tls=true
  - traefik.http.routers.myapp.tls.certresolver=lets-encrypt
  - traefik.http.middlewares.myapp-www-redirect.redirectregex.regex=^https://www\.(.+)
  - traefik.http.middlewares.myapp-www-redirect.redirectregex.replacement=https://$${1}
  - traefik.http.middlewares.myapp-www-redirect.redirectregex.permanent=true
  - traefik.http.routers.myapp.middlewares=myapp-www-redirect
```

### Disable Traefik for a container (e.g. database)

```yaml
labels:
  - traefik.enable=false
```

## Dashboard

The Traefik dashboard is not exposed publicly. Access it via SSH tunnel:

```bash
ssh -L 8080:localhost:8080 webadmin@yourserver -p 22022
```

Then open `http://localhost:8080` in your browser.

## File Structure

```
.
├── docker-compose.yml          # Traefik service definition
├── traefik.toml                # Static config: entrypoints, ACME, providers
├── traefik_dynamic.toml        # Dynamic config: dashboard auth, TLS options
├── acme.json                   # Let's Encrypt certificates (gitignored, chmod 600)
└── .gitignore
```

## References

- [Traefik v3 Documentation](https://doc.traefik.io/traefik/)
- [Traefik v2 to v3 Migration Guide](https://doc.traefik.io/traefik/migration/v2-to-v3/)
- [Let's Encrypt](https://letsencrypt.org/)
