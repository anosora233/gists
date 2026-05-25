+++
title = "让 Windows 的 ssh 连接用上 socks 代理"
date = "2025-04-12"
taxonomies.tags = ["tech"]
+++

## 前言

最近有收藏一些冷门地区的服务器作探针，这些服务器的网络直连质量通常很差。为了远程这些服务器，希望在 Windows 下透过 socks5 代理建立连接。经过查询资料，ssh 客户端本身支持透过代理进行连接，关键是配置 ProxyCommand 参数。这需要一个外部程序来建立 tcp 连接，支持这种行为的程序在 Windows 平台下有 **ncat** (**nmap** 的子程序) 和 **connect**。

## ncat 方案

在安装 nmap 后即可使用 ncat，推荐使用 scoop 安装：`scoop install nmap`。

```sh
ssh -o ProxyCommand="ncat --proxy 127.0.0.1:8080 %h %p" <host>
```

虽然不确定具体原因，但在我的使用场景下透过 ncat 建立的连接有可能在终端输出过长内容时断开连接。

## connect 方案

同样推荐使用 scoop 安装：`scoop install connect`。

```sh
ssh -o ProxyCommand="connect -S 127.0.0.1:8080 %h %p" <host>
```

## 扩展：通过 .ssh/config 预先定义行为

我们可以在主机名前定义一个是否需要代理的标记，在这里我用 `^` 代表需要被代理。编辑：`ssh_config`：

```conf
# ^[HOST]:PROXY, _[HOST]:DIRECT.
Host +*
  ProxyCommand connect -S 127.0.0.1:8080 %h %p
```

这样，在连接远程服务器时只需要在主机名前加上 `^` 标记即可透过代理连接服务器。
