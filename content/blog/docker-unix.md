+++
title = "在 Docker 中使用 unix 域套接字"
date = 2024-08-31

[taxonomies]
tags = ["docker", "unix-domain-socket"]
+++

_`Some docker compose snippets.`_

## unix domain socket

Bind a host Unix socket to a container port.

Refer to [this](https://serverfault.com/questions/1156554/bind-a-host-unix-socket-to-a-container-port).

```yml
services:
  app:
    image: someapp
  web:
    image: alpine/socat
    network_mode: service:app
    restart: always
    command:
      - unix-listen:/run/web.sock,fork,reuseaddr,mode=666
      - tcp-connect:localhost:PORT # or udp-connect:localhost:PORT
    volumes:
      - .:/run
```

## uptime kuma

```yml
services:
  app:
    image: louislam/uptime-kuma:1
    network_mode: bridge
    restart: always
    volumes:
      - .:/app/data
  web:
    image: alpine/socat
    network_mode: service:app
    restart: always
    command:
      - unix-listen:/run/web.sock,fork,reuseaddr,mode=666
      - tcp-connect:localhost:3001
    volumes:
      - .:/run
```

## librespeed

```yml
services:
  app:
    image: lscr.io/linuxserver/librespeed:latest
    network_mode: bridge
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./config:/config
  web:
    image: alpine/socat
    network_mode: service:app
    restart: always
    command:
      - unix-listen:/run/web.sock,fork,reuseaddr,mode=666
      - tcp-connect:localhost:80
    volumes:
      - .:/run
```

## caddy

```yml
services:
  app:
    image: caddy
    network_mode: host
    restart: always
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - data:/data
      - conf:/config
      - /opt/cert:/cert
      - /srv:/srv
      - /run:/run
volumes:
  data:
  conf:
```

Caddyfile Example

```caddyfile
example.com,
*.example.com {
    tls _CERT_ _KEY_
    encode zstd gzip

    @api host api.example.com
    handle @api {
        reverse_proxy [::1]:8080
    }

    # other sites


    # fallback site
    handle {
        root * /usr/share/caddy
        file_server
    }
}
```

## memos

```yml
services:
  app:
    image: neosmemo/memos:stable
    network_mode: none
    restart: always
    volumes:
      - ./memos/:/var/opt/memos
  web:
    image: alpine/socat
    network_mode: service:app
    restart: always
    command:
      - unix-listen:/run/web.sock,fork,reuseaddr,mode=666
      - tcp-connect:localhost:5230
    volumes:
      - .:/run
```
