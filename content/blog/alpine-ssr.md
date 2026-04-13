+++
title = "在 Alpine 系统上配置 ServerStatus"
date = 2025-03-22

[taxonomies]
tags = ["openrc", "alpine", "serverstatus"]
+++

_`为了监控运行 Alpine 发行版的小型服务器使用情况，折腾学习了一下 OpenRC 服务的编写。`_

## 服务器端配置

由于原始二进制文件名较冗长，在此将其重命名为 ssrd。

> 警告：请记得将所有 N 替换为你的配置。

```sh
mkdir -p /opt/ssr \
&& wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/server-x86_64-unknown-linux-musl.zip \
| unzip -po - stat_server > /opt/ssr/ssrd \
&& chmod +x /opt/ssr/ssrd \
&& chown -R daemon:daemon /opt/ssr

cat << EOF > /opt/ssr/config.toml
grpc_addr = "[::1]:9394"
http_addr = "[::1]:8080"

jwt_secret = "N"
admin_user = "N"
admin_pass = "N"

hosts_group = [
  {gid = "N", password = "N"}
]

group_gc = 3600
EOF


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

## 客户端配置

同服务端，将客户端原始二进制文件名重命名为 ssr。

对于客户端参数的管理可以使用 .env 文件来定义客户端设置。

要从 .env 文件中读取环境变量，我参考这篇[讨论](https://stackoverflow.com/questions/19331497/set-environment-variables-from-file-of-key-value-pairs)，可以在 OpenRC 脚本中使用命令 `export $(xargs < FILE)`。

```sh
# some dependencies
apk add procps iproute2 coreutils vnstat
rc-service vnstatd start
rc-update add vnstatd

# client environments
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

# client binary
mkdir -p /opt/ssr \
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/client-x86_64-unknown-linux-musl.zip \
| unzip -po - stat_client > /opt/ssr/ssr \
&& chmod +x /opt/ssr/ssr \
&& chown -R daemon:daemon /opt/ssr

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
