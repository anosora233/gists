+++
title = "在 Windows 上通过 SOCKS 代理连接 SSH"
aliases = ["/blog/proxy-ssh/"]
date = "2025-04-12"
taxonomies.tags = ["windows", "ssh", "proxy", "networking"]
+++

## 前言

最近收集了一些冷门地区的服务器用作探针，这些服务器的直连网络质量通常很差。为了远程连接这些服务器，我希望在 Windows 下通过 SOCKS5 代理建立 SSH 连接。查阅资料后发现，SSH 客户端支持通过代理连接，关键是配置 `ProxyCommand` 参数。这个参数需要调用外部程序来建立 TCP 连接，Windows 平台下可用的程序有 **ncat**（**nmap** 的子程序）和 **connect**。

## ncat 方案

安装 nmap 后即可使用 ncat，推荐通过 scoop 安装：`scoop install nmap`。

```sh
ssh -o ProxyCommand="ncat --proxy 127.0.0.1:8080 %h %p" <host>
```

在我的使用场景下，通过 ncat 建立的连接可能会在终端输出内容过长时断开，具体原因尚不确定。

## connect 方案

同样推荐使用 scoop 安装：`scoop install connect`。

```sh
ssh -o ProxyCommand="connect -S 127.0.0.1:8080 %h %p" <host>
```

## 扩展：通过 .ssh/config 预先定义行为

可以在主机名前加一个标记，表示需要通过代理连接。这里使用 `+` 作为标记，编辑 `.ssh/config`，加入以下配置：

```conf
# ^[HOST]:PROXY, _[HOST]:DIRECT.
Host +*
  ProxyCommand connect -S 127.0.0.1:8080 %h %p
```

这样，连接远程服务器时，只需在主机名前加上 `+` 标记，即可通过代理连接。
