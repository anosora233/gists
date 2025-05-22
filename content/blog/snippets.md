+++
title = "废弃的配置和代码片段"
date = 2024-08-20

[taxonomies]
tags = ["wsl", "zsh", "ndp", "ipv6", "fish"]
+++

_`Just some snippets.`_

<!-- more -->

## wsl

- locale

```zsh
localectl set-locale LANG=en_US.UTF-8
# localectl set-locale LANG=zh_CN.UTF-8
```

- tun mode `resolv.conf`

```zsh
sudo rm /etc/resolv.conf
echo "nameserver 1.0.0.1" | sudo tee /etc/resolv.conf
# immutable filesystem attribute
sudo chattr +i /etc/resolv.conf
```

## zsh

use antidote as zsh plugin manager

- clone antidote

```zsh
git clone --depth=1 https://github.com/mattmc3/antidote.git ${ZDOTDIR:-$HOME}/.antidote
```

- configuration

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

1. Install

```zsh
sudo apt install ndppd
```

2. Configuration

```zsh
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

## color os 13

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

## sync ssl cert

_`Deprecated`_

- Server Side

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

- Client Side

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

edit `/etc/network/interfaces`

```zsh
 auto he6
  iface he6 inet6 v4tunnel
    address YOURS
    netmask YOURS
    endpoint YOURS
    local YOUR
    ttl 255
    up ip -6 route add default via YOURS dev he6 table 51666
    up ip -6 rule add from 2001:470::/32 lookup 51666
    down ip -6 route del default table 51666
    down ip -6 rule del lookup 51666
```

## fish

```sh
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | \
 source && fisher install jorgebucaran/fisher

fisher install pure-fish/pure

echo "\
set -x EDITOR nano
bind \b backward-kill-word\
"> ~/.config/fish/conf.d/xtom.fish
```
