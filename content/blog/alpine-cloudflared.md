+++
title = "在 Alpine 系统上配置 Cloudflared"
date = "2025-04-02"
taxonomies.tags = ["alpine"]
+++

## 前言

Cloudflared 可以安全将处于严格 NAT 限制服务器的 web 服务透过 Cloudflare 的服务器开放在公网上，以下是我在 Alpine 上配置 Cloudflared 的经验。

## 安装

截至目前官方仓库的 cloudflared 包还在 testing 分支。参考 Alpine 文档有关[新增仓库的语法](https://wiki.alpinelinux.org/wiki/Repositories)，可以考虑使用 `@` 语法临时启用 testing 分支仓库。具体而言，修改 /etc/apk/repositories 为如下所示。

```conf
https://dl-cdn.alpinelinux.org/alpine/v3.23/main
https://dl-cdn.alpinelinux.org/alpine/v3.23/community
@testing https://dl-cdn.alpinelinux.org/alpine/edge/testing
```

之后通过 `apk add cloudflared@testing` 即可安装，这个软件包已经包含了 OpenRC 启动脚本，无需再手动编写。顺带一提，虽然 Cloudflared 本身支持生成守护进程配置文件，但这个功能仅仅支持 systemd 发行版。

## 运行

通过查看 `/etc/conf.d/cloudflared` 发现 cloudflared 启动时读取 `/etc/cloudflared/config.yml` 配置文件。除了修改指向的配置文件，这里也可以考虑直接修改启动参数，只需要 Cloudflare Tunnel 的 token 即可。具体而言，修改 `/etc/conf.d/cloudflared` 的启动参数为以下内容：

```sh
command_args="tunnel run --token $YOUR_TOKEN"
```

其中 `$YOUR_TOKEN` 为你的通道令牌，如果服务器的 IPv4 网络性能不佳，也可以连接 IPv6 的边缘节点：

```sh
command_args="--edge-ip-version 6 tunnel run --token $YOUR_TOKEN"
```

最后，通过以下命令启动和添加开机自启动

```sh
rc-service cloudflared start
rc-update add cloudflared
```
