+++
title = "在基于 Alpine 上配置 ServerStatus"
date = 2025-03-22

[taxonomies]
tags = ["openrc", "alpine", "serverstatus"]
+++

_`用于监控基于 OpenRC 的发行版使用情况`_

虽然在基于 systemd 的发行版上配置 ServerStatus-Rust 非常简单，但该存储库缺乏针对 OpenRC 系统用户的官方指导。为了解决这个问题，我想分享我在 OpenRC 基础发行版上部署这个小工具的经验。

整个过程被简化为一段脚本，只需复制并粘贴代码即可运行。

## 服务器端配置

由于原始二进制文件名相当冗长，因此我更喜欢将其重命名为 ssrd，以方便使用。

> 警告：请记得将所有 N 替换为你的配置。

```sh
mkdir -p /opt/ssr

# server binary
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/server-x86_64-unknown-linux-musl.zip \
| unzip -po - stat_server > /opt/ssr/ssrd && chmod +x /opt/ssr/ssrd

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

respawn_max=30
supervisor="supervise-daemon"
supervise_daemon_args="--chdir /opt/ssr"
command="/opt/ssr/ssrd"
command_args="-c /opt/ssr/config.toml"

depend() {
  after net dns
}
EOF

chmod +x /etc/init.d/ssrd
rc-service ssrd start
rc-update add ssrd
```

## 客户端配置

为了有效记录带宽使用情况，必须安装 vnstatd。此外，还应安装其他重要依赖项，如 iproute2 和 coreutils，以确保正常功能。

对于客户端参数的管理，这里使用 .env 文件来定义客户端设置。要从 .env 文件中读取环境变量，可以使用命令 export $(xargs < FILE)，正如 [Stack Overflow 讨论](https://stackoverflow.com/questions/19331497/set-environment-variables-from-file-of-key-value-pairs) 中所建议的那样。

```sh
mkdir -p /opt/ssr

# some dependencies
apk add procps iproute2 coreutils vnstat
rc-service vnstatd start
rc-update add vnstatd

# client binary
wget -qO- https://github.com/zdz/ServerStatus-Rust/releases/download/v1.8.1/client-x86_64-unknown-linux-musl.zip \
| unzip -po - stat_client > /opt/ssr/ssr && chmod +x /opt/ssr/ssr


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

cat << EOF > /etc/init.d/ssr
#!/sbin/openrc-run

export RUST_BACKTRACE=1 $(xargs</opt/ssr/.env)

respawn_max=30
supervisor=supervise-daemon
supervise_daemon_args="--chdir /opt/ssr"
command=/opt/ssr/ssr

depend() {
  after net dns
}
EOF

chmod +x /etc/init.d/ssr
rc-service ssr start
rc-update add ssr
```
