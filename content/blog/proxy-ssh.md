+++
title = "为 Windows 的 ssh 连接用上 socks 代理"
date = 2025-04-12

[taxonomies]
tags = ["ssh", "proxy", "windows"]
+++

_`为了远程一些直连网络质量不太好的服务器，希望在 Windows 下透过 socks5 代理建立连接。`_

## 前言

ssh 客户端支持透过代理进行连接，关键是配置 ProxyCommand 参数。这需要一个外部程序来建立 tcp 连接，支持这种行为的程序在 Windows 平台下有 `ncat` (`nmap` 的子程序) 和 `connect`。

## ncat 方案

安装 nmap 即可拥有 ncat，推荐使用 scoop 安装：`scoop install nmap`。

```sh
ssh -o ProxyCommand="ncat --proxy 127.0.0.1:8080 %h %p" <host>
```

虽然不确定具体原因，但在我的使用场景下透过 ncat 建立的连接有可能在终端输出过长内容时断开连接。

## connect 方案

同样推荐使用 scoop 安装：`scoop install connect`。

```sh
ssh -o ProxyCommand="connect -S 127.0.0.1:8080 %h %p" <host>
```

## 通过 .ssh/config 预先定义行为

我们可以在主机名前定义一个是否需要代理的标记，在这里我用 **+** 代表需要代理。具体配置如下：

```conf
# +[HOST]:PROXY, _[HOST]:DIRECT.
Host +*
  ProxyCommand connect -S 127.0.0.1:8080 %h %p
```

这样，在连接远程服务器时只需要在主机名前加上 **+** 标记即可透过代理连接服务器。