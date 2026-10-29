# Reverse Proxy

When running Grimoire behind a reverse proxy, set `BASE_URL` to your public URL so that OPDS feed links and OIDC redirect URIs are generated correctly:

```yaml
environment:
  - BASE_URL=https://grimoire.example.com
```

Grimoire reads the client's address from `X-Forwarded-For` only when the request comes from a trusted proxy. The default list - loopback plus `172.16.0.0/12`, the range Docker gives its bridge networks - covers a proxy or tunnel on the same host or in the same Docker Compose project, so the setups below need nothing more when run that way. If your proxy reaches Grimoire from anywhere else - another machine on your LAN, a Kubernetes ingress, a Podman network, a VPS, or Cloudflare's proxy without Cloudflare Tunnel - add its address to `TRUSTED_PROXIES`, or every client will share the proxy's address for rate limiting and Active Sessions. See [Behind a reverse proxy](/configuration/security#behind-a-reverse-proxy).

## Nginx

```nginx
server {
    listen 443 ssl;
    server_name grimoire.example.com;

    location / {
        proxy_pass http://localhost:9481;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Needed for large PDF downloads
        proxy_read_timeout 300;
        proxy_send_timeout 300;
        client_max_body_size 0;
    }
}
```

## Caddy

```
grimoire.example.com {
    reverse_proxy localhost:9481
}
```

## Traefik (Docker labels)

```yaml
services:
  grimoire:
    image: hunterreadca/grimoire:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.grimoire.rule=Host(`grimoire.example.com`)"
      - "traefik.http.routers.grimoire.entrypoints=websecure"
      - "traefik.http.routers.grimoire.tls.certresolver=letsencrypt"
      - "traefik.http.services.grimoire.loadbalancer.server.port=9481"
```
