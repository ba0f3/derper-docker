# Derper

[![docker workflow](https://github.com/ba0f3/derper-docker/actions/workflows/docker-image.yml/badge.svg)](https://hub.docker.com/r/rgv151/derper)
[![docker pulls](https://img.shields.io/docker/pulls/rgv151/derper.svg?color=brightgreen)](https://hub.docker.com/r/rgv151/derper)
[![platfrom](https://img.shields.io/badge/platform-amd64%20%7C%20arm64-brightgreen)](https://hub.docker.com/r/rgv151/derper/tags)

This Docker image is built on [Chainguard's Wolfi](https://www.chainguard.dev/unchained/introducing-wolfi-the-first-linux-un-distro), a security-focused, minimal Linux distribution designed for containers.

# Setup

> required: set env `DERP_DOMAIN` to your domain

```bash
docker run -e DERP_DOMAIN=derper.your-domain.com -p 80:80 -p 443:443 -p 3478:3478/udp fredliang/derper
```

| env                    | required | description                                                                 | default value     |
| -------------------    | -------- | ----------------------------------------------------------------------      | ----------------- |
| DERP_DOMAIN            | true     | derper server hostname                                                      | your-hostname.com |
| DERP_CERT_DIR          | false    | directory to store LetsEncrypt certs(if addr's port is :443)                | /app/certs        |
| DERP_CERT_MODE         | false    | mode for getting a cert. possible options: manual, letsencrypt              | letsencrypt       |
| DERP_ADDR              | false    | listening server address                                                    | :443              |
| DERP_STUN              | false    | also run a STUN server                                                      | true              |
| DERP_STUN_PORT         | false    | The UDP port on which to serve STUN.                                        | 3478              |
| DERP_HTTP_PORT         | false    | The port on which to serve HTTP. Set to -1 to disable                       | 80                |
| DERP_VERIFY_CLIENTS    | false    | verify clients to this DERP server through a local tailscaled instance      | false             |
| DERP_VERIFY_CLIENT_URL | false    | if non-empty, an admission controller URL for permitting client connections | ""                |

# Docker Compose Example

Here's a complete Docker Compose example for easy deployment:

```yaml
services:
  derper:
    image: rgv151/derper:latest
    container_name: derper
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "3478:3478/udp"
    environment:
      - DERP_DOMAIN=derper.your-domain.com
      - DERP_CERT_DIR=/app/certs
      - DERP_CERT_MODE=letsencrypt
      - DERP_ADDR=:443
      - DERP_STUN=true
      - DERP_STUN_PORT=3478
      - DERP_HTTP_PORT=80
      - DERP_VERIFY_CLIENTS=false
    volumes:
      - derper_certs:/app/certs
      # Uncomment the following line if using client verification
      # - /var/run/tailscale/tailscaled.sock:/var/run/tailscale/tailscaled.sock
    networks:
      - derper_network

volumes:
  derper_certs:
    driver: local

networks:
  derper_network:
    driver: bridge
```

To deploy:

```bash
# Update DERP_DOMAIN in the compose file
docker-compose up -d
```

# Usage

Fully DERP setup offical documentation: https://tailscale.com/kb/1118/custom-derp-servers/

## Client verification

In order to use `DERP_VERIFY_CLIENTS`, the container needs access to Tailscale's Local API, which can usually be accessed through `/var/run/tailscale/tailscaled.sock`. If you're running Tailscale bare-metal on Linux, adding this to the `docker run` command should be enough: `-v /var/run/tailscale/tailscaled.sock:/var/run/tailscale/tailscaled.sock`
