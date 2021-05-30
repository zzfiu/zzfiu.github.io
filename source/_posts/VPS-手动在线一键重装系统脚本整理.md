---
title: VPS 手动在线一键重装系统脚本整理
abbrlink: 9d46
date: 2021-05-23 01:01:22
tags:
---

## VPS 手动在线一键重装系统脚本整理

总有一些 VPS 商家会对服务器系统动一些小手脚，又或者一些坑爹商家面板出 bug 导致无法正常重装系统。这时我们都可以尝试使用某些大佬制作的一键重装系统脚本，只要你还能使用 SSH 连接 VPS，剩下操作都十分简单。

本文只介绍 Linux 系统在服务器上的重装，Windows 系统由于版权因素还是建议使用官方的方法进行安装。

### 1. CentOS 6.9 及以下
> 来源:[萌咖](moeclub.org)
> 适用于由 GRUB 引导的 CentOS 系统，使用官方发行版去掉模板预装的软件。同时也可以解决内核版本与软件不兼容的问题。只要有 root 权限,还您一个纯净的系统
>
> **不适用于 `OpenVZ` 架构！**

首先更新已有工具包，并安装所需软件：

```
yum update
yum install -y xz openssl gawk coreutils file
```

之后下载脚本及使用（默认 root 密码：`Vicer`）：

```
wget --no-check-certificate -qO CentOSNET.sh 'https://moeclub.org/attachment/LinuxShell/CentOSNET.sh' && chmod a+x CentOSNET.sh
```

```
Usage:
        bash CentOSNET.sh       -c/--centos [dist-version]     #系统版本
                                -v/--ver [32/i386|64/amd64]    #指定格式
                                --ip-addr/--ip-gate/--ip-mask  #网络参数（默认自动识别）
                                -yum/--mirror                  #自定义镜像源
                                -a/-m                          #全自动安装/VNC安装
```

示例（从指定源全自动安装 64 位 CentOS 6.9）：

```
sh CentOSNET.sh -c 6.9 -v 64 -a --mirror 'http://mirror.centos.org/centos'
```

仅支持 CentOS 6.9 及以下版本，暂不支持 CentOS 7。

### 2.CentOS 7

> 来源 [hiCasper](https://blog.hicasper.com/post/135.html)
>
> 本一键脚本在萌咖大佬的脚本基础上开发，实现了懒人式一键网络重装 Debian / Ubuntu / CentOS 系统及dd方式安装系统。解决了云服务商提供模板镜像体积过大、预装软件过多、不够纯净等问题。

此脚本支持 CentOS 7 安装（仅 DD）：

```
wget --no-check-certificate -O AutoReinstall.sh https://git.io/AutoReinstall.sh && bash AutoReinstall.sh
```
输入编号选择你需要的系统，之后会提示 root 密码，只需等待一会便会全自动安装完成。

**支持重装的系统**

- Ubuntu 18.04/16.04
- Debian 9/10
- CentOS 6
- CentOS 7 （DD方式）
- 自定义DD镜像

**特性 / 优化**

- 自动获取IP地址、网关、子网掩码
- 自动判断网络环境，选择国内/外镜像，解决速度慢的问题
- 懒人一键化，无需复杂的命令
- 解决萌咖脚本中一些导致安装错误的问题
- CentOS 7 镜像抛弃LVM，回归ext4，减少不稳定因素

**注意**

- 重装后系统密码均在脚本中有提供，**安装后请尽快修改密码**，Linux系统建议启用密钥登陆。
- OpenVZ / LXC 架构系统不适用

### 3. Debian/Ubuntu

> 来源:[萌咖](moeclub.org)
>
> 仍是 萌咖 大佬的一键脚本，同样不适用于 `OpenVZ` 架构

更新工具包，安装所需软件：

```
apt-get update

apt-get install -y xz-utils openssl gawk file
```

下载脚本及使用（默认 root 密码：`MoeClub.org`）：

```
wget --no-check-certificate -qO InstallNET.sh 'https://moeclub.org/attachment/LinuxShell/InstallNET.sh' && chmod a+x InstallNET.sh
Usage:
        bash InstallNET.sh      -d/--debian [dist-name]
                                -u/--ubuntu [dist-name]
                                -c/--centos [dist-version]
                                -v/--ver [32/i386|64/amd64]
                                --ip-addr/--ip-gate/--ip-mask
                                -apt/-yum/--mirror
                                -dd/--image
                                -a/-m

# dist-name: 发行版本代号
# dist-version: 发行版本号
# -apt/-yum/--mirror : 使用定义镜像
# -a/-m : 询问是否能进入VNC自行操作. -a 为不提示(一般用于全自动安装), -m 为提示.
```

示例（默认源一键重装 64 位 Debian 8）：

```
sh InstallNET.sh -d 8 -v 64 -a
```

**重装系统可能需要 10 - 30 分钟不等，请耐心等待。安装完成后默认使用 22 端口进行 SSH 连接。**

**这里收集的脚本都支持一键安装，对我等小白特别友好。**
