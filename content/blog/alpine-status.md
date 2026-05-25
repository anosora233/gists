+++
title = "在 Alpine 系统上配置 ServerStatus"
date = "2025-03-22"
taxonomies.tags = ["alpine"]
+++

## 前言

Alpine 是一个基于 musl libc 和 BusyBox 的小型发行版，其镜像体积小、启动速度快和资源消耗低。我在我的一些小型服务器（内存 <= 256 MB）上部署了这个发行版，体验很好。后续为了使用 ServerStatus-Rust 监控这些运行 Alpine 发行版的小型服务器使用情况，折腾学习了一下 OpenRC 服务的编写。

## 服务端

我将项目相关的文件都放在 /opt/ssr 目录下。此外，由于原始二进制文件名较冗长，在此将其重命名为。没有必要使用 root 用户运行该程序，这里我使用大多数 Linux 发行版内置的 daemon 用户。

```sh
mkdir -p /opt/ssr
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/server-x86_64-unknown-linux-musl.zip | unzip -po - stat_server > /opt/ssr/ssrd
chmod +x /opt/ssr/ssrd && chown -R daemon:daemon /opt/ssr
```

编写配置文件：HTTP 和 gRPC 端口分别开放在本地的 8080、9394 端口。之后可以通过 nginx 反代并添加 SSL 支持。请将所有的 **N** 替换为你的配置。

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

编写 init.d 文件：这里使用 daemon 用户运行进程。

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

要使 ServerStatus-Rust 客户端在 Alpine 上正常运行，需要再额外添加一些软件包。

```sh
apk add procps iproute2 coreutils vnstat
rc-service vnstatd start
rc-update add vnstatd
```

同服务端命名格式，将客户端原始二进制文件名重命名为 ssr。

```sh
mkdir -p /opt/ssr
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/client-x86_64-unknown-linux-musl.zip | unzip -po - stat_client > /opt/ssr/ssr
chmod +x /opt/ssr/ssr && chown -R daemon:daemon /opt/ssr
```

对于客户端参数的管理可以使用 .env 文件。要从 .env 文件中读取环境变量，我参考这篇[讨论](https://stackoverflow.com/questions/19331497/set-environment-variables-from-file-of-key-value-pairs)，可以在 OpenRC 脚本中使用命令 `export $(xargs < FILE)`，当然这种方法不允许在 .env 中使用注释。

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

有关这些环境变量的含义可以参考 ServerStatus-Rust 仓库。

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
