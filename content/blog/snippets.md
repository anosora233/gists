+++
title = "废弃的配置和代码片段"
date = "2024-08-20"
taxonomies.tags = ["shell"]
+++

## wsl

Debian 系语言配置。

```zsh
localectl set-locale LANG=en_US.UTF-8
# localectl set-locale LANG=zh_CN.UTF-8
```

在 Tun 模式下 WSL 不能正常联网，可以通过固定 WSL 系统的 dns 服务器解决。

```zsh
sudo rm /etc/resolv.conf
echo "nameserver 1.0.0.1" | sudo tee /etc/resolv.conf
# immutable filesystem attribute
sudo chattr +i /etc/resolv.conf
```

## zsh

这里使用 antidote 作为 zsh 的插件管理器。

```zsh
git clone --depth=1 https://github.com/mattmc3/antidote.git ${ZDOTDIR:-$HOME}/.antidote
```

具体的 .zshrc 配置文件：

```zsh
cat << 'EOF' > .zshrc
# Lines configured by zsh-newuser-install
HISTFILE=~/.histfile
HISTSIZE=1000
SAVEHIST=1000
# End of lines configured by zsh-newuser-install
# The following lines were added by compinstall
zstyle :compinstall filename ${HOME}/.zshrc

autoload -Uz compinit
compinit
# End of lines added by compinstall
# antidote
source ${ZDOTDIR:-$HOME}/.antidote/antidote.zsh
antidote load ${ZDOTDIR:-$HOME}/.zsh_plugins.txt

if [ -d "${HOME}/.acme.sh" ]; then
. "${HOME}/.acme.sh/acme.sh.env"
fi

zstyle ':prompt:pure:path' color yellow
zstyle ':prompt:pure:prompt:success' color cyan
zstyle ':completion:*' rehash true

bindkey '^H' backward-kill-word
EOF

cat << EOF > .zsh_plugins.txt
# .zsh_plugins.txt
rupa/z              # some bash plugins work too
sindresorhus/pure   # enhance your prompt

# you can even use Oh My Zsh plugins
getantidote/use-omz
ohmyzsh/ohmyzsh path:lib
ohmyzsh/ohmyzsh path:plugins/extract

# add fish-like features
zsh-users/zsh-syntax-highlighting
zsh-users/zsh-autosuggestions
zsh-users/zsh-history-substring-search
EOF
```

## ndppd

配置 ndppd 让你的闲置的 /64 块 IPv6 发挥效果，现在可以通过该块下任意一地址访问目标主机。

```zsh
sudo apt install ndppd

# replace `2001:ff::/64` with yours
cat << EOF > /etc/ndppd.conf
route-ttl 30000
proxy eth0 {
    router no
    timeout 500
    ttl 30000
    rule 2001:ff::/64 {
        static
    }
}
EOF

ip route add local 2001:ff::/64 dev lo
ip route del local 2001:ff::/64 dev lo
```

## ColorOS

以下我在 C13 系统上禁用的一些软件包

```zsh
# adb shell pm unsuspend <package>
adb shell pm suspend com.oplus.ota

# adb shell pm install-existing --user 0 <package>
adb shell pm disable-user com.opos.ads
adb shell pm disable-user com.heytap.yoli
adb shell pm disable-user com.heytap.music

adb shell pm uninstall --user 0 com.heytap.browser
adb shell pm uninstall --user 0 com.opuls.appdetail
adb shell pm uninstall --user 0 com.nearme.instant.platform
```

## SSL Cert

通过 rsync 在一组服务器直接同步 SSL 证书，非去中心化的方案。

- 服务端

```zsh
sudo apt install rsync

echo "USER:PASSWORD" > /etc/rsyncd.secrets
chmod 600 /etc/rsyncd.secrets

cat << EOF > /etc/rsyncd.conf
uid = 0
gid = 0
read only = yes
secrets file = /etc/rsyncd.secrets
[cert]
path = /root/cert
auth users = cert
EOF

sudo systemctl enable --now rsync
sudo systemctl status rsync
```

- 客户端

```zsh
sudo apt install rsync

echo "PASSWORD" > /root/syncpass
chmod 600 /root/syncpass

cat << EOF > /root/sync.sh
#!/bin/bash
rsync -avz --password-file=/root/syncpass cert@SERVER::cert /root/cert
EOF

# crontab -e then add a new line
@monthly bash /root/sync.sh
```

## hurricane electric ipv6 tunnel

编辑：`/etc/network/interfaces`

```zsh
auto he6
  iface he6 inet6 v4tunnel
    address YOURS
    netmask YOURS
    endpoint YOURS
    local YOUR
    ttl 255
    up ip route add ::/0 dev he6 metric 2048 
```

## fish

使用 fisher 作为 fish 的插件管理器。

```sh
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | \
 source && fisher install jorgebucaran/fisher

fisher install pure-fish/pure

echo "\
set -x EDITOR nano
bind \b backward-kill-word\
"> ~/.config/fish/conf.d/xtom.fish
```
