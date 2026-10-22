# Reverse Proxy

When running Grimoire behind a reverse proxy, set `BASE_URL` to your public URL so that OPDS feed links and OIDC redirect URIs are generated correctly:

```yaml
environment:
  - BASE_URL=https://grimoire.example.com
```

Grimoire reads the client's address from `X-Forwarded-For` only when the request comes from a trusted proxy. The default list covers a proxy or tunnel on the same host, Docker network or LAN, so the setups below need nothing more. If your proxy reaches Grimoire from a public address (a VPS, or Cloudflare's proxy without Cloudflare Tunnel), add it to `TRUSTED_PROXIES`, or every client will share one rate-limit bucket. See [Behind a reverse proxy](/configuration/security#behind-a-reverse-proxy).

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
