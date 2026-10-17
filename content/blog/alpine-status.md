+++
title = "在 Alpine 系统上配置 ServerStatus"
date = "2025-03-22"
taxonomies.tags = ["alpine"]
+++

## 前言

Alpine 是一个基于 musl libc 和 BusyBox 的小型发行版，具有镜像体积小、启动速度快、资源消耗低的特点。我在一些小型服务器（内存 ≤ 256 MB）上部署了这个发行版，体验很好。后来，为了使用 ServerStatus-Rust 监控这些服务器的运行情况，又折腾了一番，学习了如何编写 OpenRC 服务脚本。

## 服务端

我将项目相关文件都放在 `/opt/ssr` 目录下。由于原始二进制文件名较长，这里将其重命名为 `ssrd`。该程序无需使用 root 用户运行，这里使用大多数 Linux 发行版内置的 daemon 用户。

```sh
mkdir -p /opt/ssr
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/server-x86_64-unknown-linux-musl.zip | unzip -po - stat_server > /opt/ssr/ssrd
chmod +x /opt/ssr/ssrd && chown -R daemon:daemon /opt/ssr
```

编写配置文件，让 HTTP 和 gRPC 服务分别监听本地的 8080 和 9394 端口。之后可以通过 nginx 反向代理，并添加 SSL 支持。请将所有的 **N** 替换为你的配置。

```sh
cat << EOF > /opt/ssr/config.toml
grpc_addr = "127.0.0.1:9394"
http_addr = "127.0.0.1:8080"

jwt_secret = "N"
admin_user = "N"
admin_pass = "N"

hosts_group = [
  {gid = "N", password = "N"}
]

group_gc = 3600
EOF
```

编写 init.d 脚本，使用 daemon 用户运行进程：

```sh
cat << EOF > /etc/init.d/ssrd
#!/sbin/openrc-run

respawn_delay=5
respawn_period=60
supervisor=supervise-daemon
directory=/opt/ssr
command_user=daemon:daemon
command=/opt/ssr/ssrd
command_args="-c /opt/ssr/config.toml"

depend() {
  after net dns
}
EOF

chmod +x /etc/init.d/ssrd
rc-service ssrd start && rc-update add ssrd
```

## 客户端

要使 ServerStatus-Rust 客户端在 Alpine 上正常运行，还需要安装一些额外的软件包：

```sh
apk add procps iproute2 coreutils vnstat
rc-service vnstatd start
rc-update add vnstatd
```

沿用服务端的命名方式，将客户端的二进制文件重命名为 `ssr`。

```sh
mkdir -p /opt/ssr
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/client-x86_64-unknown-linux-musl.zip | unzip -po - stat_client > /opt/ssr/ssr
chmod +x /opt/ssr/ssr && chown -R daemon:daemon /opt/ssr
```

客户端参数可以通过 `.env` 文件管理。参考这篇[讨论](https://stackoverflow.com/questions/19331497/set-environment-variables-from-file-of-key-value-pairs)，可以在 OpenRC 脚本中使用 `export $(xargs < FILE)` 读取文件中的环境变量。不过，这种方法不支持在 `.env` 文件中使用注释。

```sh
cat << EOF > /opt/ssr/.env
SSR_GID=N
SSR_PASS=N
SSR_LOC=us
SSR_TYPE=kvm
SSR_VNSTAT=true
SSR_VNSTAT_MR=10
SSR_ALIAS=N
SSR_ADDR=N
EOF
```

这些环境变量的含义可以参考 ServerStatus-Rust 仓库中的说明。

```sh
cat << 'EOF' > /etc/init.d/ssr
#!/sbin/openrc-run

export RUST_BACKTRACE=1 $(xargs</opt/ssr/.env)

respawn_delay=5
respawn_period=60
supervisor=supervise-daemon
directory=/opt/ssr
command=/opt/ssr/ssr
command_user=daemon:daemon

depend() {
  after net dns
}
EOF

chmod +x /etc/init.d/ssr
rc-service ssr start && rc-update add ssr
```
