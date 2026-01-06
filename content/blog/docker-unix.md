+++
title = "在 Docker 中使用 unix 域套接字"
date = 2024-08-31

[taxonomies]
tags = ["docker", "unix-domain-socket"]
+++

_`曾经部署的一些自托管服务，用了一些不太寻常的方法让每个容器用上了域名套接字。这样做的好处有每个应用都可得到一个非常清晰的套接字名称并且避免了占用主机端口，但缺点是需要一个额外的容器及额外的资源开销。`_

# unix domain socket

以下是基本的 compose 模板，主要参考了这篇[讨论](https://serverfault.com/questions/1156554/bind-a-host-unix-socket-to-a-container-port).

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

_Caddyfile Example_

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
