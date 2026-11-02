+++
title = "使用 Agent 配置 Arch Linux"
date = "2026-11-02"
taxonomies.tags = ["archlinux", "agent", "wayland"]
+++

## 前言

这几天重新安装了 Arch Linux，尝试了不少以前想用但一直没有配置的桌面环境。

以前安装 Linux，配置一个输入法都可能花半天。这次我自己完成 archinstall 的安装流程，主要花时间重新学习了 Btrfs 的分区和子卷配置。进入系统后，后续工作基本就交给 ChatGPT 了。

## 桌面环境

这次从比较完整的 KDE Plasma、GNOME，到 Hyprland、niri、Umbriel，都尝试了一遍。以前想做却没有做的事情，这几天基本试了个遍。

其中，[niri](https://github.com/niri-wm/niri) 和 [Umbriel](https://github.com/noctalia-dev/umbriel) 给我的感受最明显。niri 的滚动平铺，以及 Umbriel 提供的不同布局，让我重新考虑了窗口应该如何排列和切换。对我而言，这种体验确实称得上革新。如果此前没有尝试过，我觉得值得一试。

配合这些 Wayland 合成器，我也尝试了两种桌面方案：用 DMS、[Noctalia](https://noctalia.dev/shell) 这类集成 Shell，或者自己搭配 Waybar、[mako](https://github.com/emersion/mako) 和 Fuzzel，分别配置状态栏、通知和启动器。

以前最头疼的是这些组件各有一套配置文件。想调整快捷键、窗口规则或外观，就得分别查阅文档。现在可以直接描述需求，让 Agent 找到对应配置并修改，再根据实际效果调整。定制自己的桌面风格，从来没有像现在这样简单。

字体也折腾了一下。[Fontconfig 的别名规则](https://fontconfig.pages.freedesktop.org/fontconfig/fontconfig-user.html)配合 `binding="strong"`，可以让自己选的字体优先于 Arial、Roboto。只要程序通过 Fontconfig 匹配字体，就能这样调整，不用挨个去找应用里的字体设置。这点用起来很舒服。

## 快照与启动

这么多次调整，Agent 没有把系统改到无法启动过。每次较大的改动前，我都会创建 [Btrfs 子卷快照](https://btrfs.readthedocs.io/en/stable/btrfs-subvolume.html)，只是到现在还没有用它恢复过。它用来撤回系统改动比较方便，不过独立的 EFI 系统分区不在里面，磁盘坏了也救不了，备份还是另一回事。

这次还开始使用 Limine。唯一一次无法启动，就是我自己改了它的配置，漏了后续操作。按 [Limine 的要求](https://github.com/limine-bootloader/limine/blob/v12.x/USAGE.md)，启用配置校验后，修改 `limine.conf` 需要重新登记校验值，再给修改后的 EFI 程序签名。我只改了配置就重启，这次完全是自己的问题。

## Wayland 与游戏

我还尝试让 osu! 和 Minecraft 使用原生 Wayland。以前要自己查启动参数、运行时和环境变量，这次基本一两轮对话就能配好，自己启动试一下就行。

但配置变得容易，并没有消除 Linux 桌面本身的问题。

例如，我遇到过从托盘退出应用后仍有残留进程的情况。Plasma Login Manager 的界面语言也没有跟随系统 locale，需要单独调整配置。这些问题未必影响主要功能，但时不时就要停下来处理一下。

XWayland 的体验尤其让我不满意。我这里 CS2 明明能使用原生 Wayland，默认启动却还是走 X11，需要另外调整启动环境。鼠标无法正常控制、输入法候选框位置异常这些问题也遇到过。有时还得查是程序、输入法，还是合成器出了问题，用着用着就被打断，确实很烦。

游戏兼容性仍有些像开盲盒。Proton 和社区分享的经验解决了不少问题，但具体到某个游戏、显卡和桌面环境，仍可能需要额外处理。相比之下，Windows 用户通常安装后就能直接启动游戏，不必先处理这些兼容性配置。

## Linux 与 Windows

Linux 的自定义能力依然很吸引我。从字体、快捷键到窗口布局，几乎都可以按自己的习惯调整。Agent 又大幅降低了配置成本，让我更愿意尝试这些方案。但就目前的使用体验而言，我仍不觉得桌面 Linux 适合大多数人。

如果只有一台电脑，我可能还是会选择 Windows。「Windows 是最好的 Linux 发行版」虽然是句玩笑，但对我的需求确实有些道理。

我在 Hyper-V 中运行 Alpine，用来配置软路由、运行 Docker，体验已经很方便。Windows 本身可以用于游戏和远程串流，也省去了 Linux 宿主机上配置显卡直通、处理部分游戏对虚拟机检测的麻烦。

不过 Hyper-V 的动态内存让我不太满意。原本希望虚拟机闲着时少占点，有需要再加，实际用起来却不太顺：有时客户机内存偏低，程序 OOM，有时又长时间占着较多内存，Windows 开始频繁读写页面文件。[文档里](https://learn.microsoft.com/windows-server/virtualization/hyper-v/dynamic-memory)有最小、最大内存和缓冲的设置说明，我这套配置用下来，还是得留意两边的内存。

如果有第二台电脑，我非常愿意让它常驻 Linux。内存占用也让我满意：这次用的较轻桌面，启动后不到 2 GB，挂着 Firefox、Chrome、ChatGPT，再开 Minecraft、osu! 和 CS2，看到的占用大约是 14 GB。只是当时的读数，不过感觉 8 GB 内存用来跑较轻的 Linux 桌面和日常应用，也能比较宽裕。

这几天的体验让我确认，Agent 已经能帮我完成大量原本需要手动处理的配置工作，让 Linux 更容易按我的想法运行。至于兼容性和各组件之间的一致性，仍然离不开开源社区的持续完善，也感谢这些项目的开发者和贡献者。

*本文由 ChatGPT 根据个人经历和相关文档撰写，人类仅提供思路并引导修改。*
