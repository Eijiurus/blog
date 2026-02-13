---
title: 复盘 | 双系统安装
published: 2026-02-10T18:33:10.621Z
description: ''
updated: ''
tags:
  - Tag
draft: false
pin: 0
toc: true
lang: ''
abbrlink: 'linux-and-windows'
---



# Ubuntu + Windows 11 双系统安装复盘

## 一、任务概览

在一台华硕 Vivobook 笔记本上,基于已有的 Ubuntu 24.04 LTS 系统,安装 Windows 11 双系统。主要目的是在 Windows 下使用 Unity Hub 进行游戏开发。

### 硬件环境

| 项目 | 详情 |
|------|------|
| 电脑型号 | ASUS Vivobook |
| 磁盘 | NVMe SSD 476.9 GiB(HFM512GD3JX013N) |
| 网卡 | MediaTek MT7921 802.11ax |
| 引导模式 | UEFI + GPT |
| 已有系统 | Ubuntu 24.04 LTS(Lubuntu 桌面) |

---

## 二、涉及的计算机知识

### 2.1 磁盘与分区

GPT(GUID Partition Table): 

现代磁盘分区方案,取代了传统的 MBR(Master Boot Record)。GPT 支持超过 2TB 的磁盘、最多 128 个分区,并且内置冗余校验(在磁盘首尾各存一份分区表)。通过 `fdisk -l` 可以看到 `Disklabel type: gpt`。

NVMe(Non-Volatile Memory Express): 

一种基于 PCIe 总线的 SSD 协议,比传统 SATA SSD 快很多。在 Linux 中设备名为 `/dev/nvme0n1`,分区名为 `/dev/nvme0n1p1`、`/dev/nvme0n1p2` 等。其中 `nvme0` 表示第一个 NVMe 控制器,`n1` 表示第一个命名空间(namespace),`p1` 表示第一个分区。

EFI 系统分区(ESP): 

UEFI 引导必需的 FAT32 分区,存放各操作系统的引导文件(如 `\EFI\ubuntu\shimx64.efi` 和 `\EFI\Microsoft\Boot\bootmgfw.efi`)。多个操作系统可以共用一个 ESP。本次安装中,Ubuntu 和 Windows 共用了 `/dev/nvme0n1p1` 这个 1GiB 的 EFI 分区。

ext4: 

Linux 最常用的文件系统,支持日志(journaling)、大文件、在线扩容等特性。我们的 Linux 根分区使用的就是 ext4。

### 2.2 UEFI 引导

UEFI(Unified Extensible Firmware Interface): 

现代计算机的固件接口,取代了传统 BIOS。UEFI 直接从 ESP 分区中读取 `.efi` 文件来启动操作系统,而不像 BIOS 那样读取 MBR 中的引导代码。

判断方法: 通过 `ls /sys/firmware/efi` 判断当前系统是否运行在 UEFI 模式。如果该目录存在,说明是 UEFI 模式。

UEFI 启动管理器: 

UEFI 固件内部维护了一个启动项列表(Boot0000、Boot0001、Boot0004 等),每个启动项指向 ESP 中的一个 `.efi` 文件。`efibootmgr` 命令可以查看和修改这个列表。

### 2.3 GRUB 引导加载器

GRUB(GRand Unified Bootloader): Linux 最常用的引导加载器。它的工作流程是:

1. UEFI 固件加载 `shimx64.efi`(或直接加载 `grubx64.efi`)
2. GRUB 读取配置文件 `/boot/grub/grub.cfg`
3. 显示启动菜单供用户选择
4. 加载选中的操作系统内核

GRUB 配置体系:

- `/etc/default/grub`: 主配置文件,定义超时时间、默认启动项、分辨率等
- `/etc/default/grub.d/*.cfg`: 附加配置文件,会覆盖主配置(本次遇到 `lubuntu-grub-theme.cfg` 覆盖主题设置的问题)
- `/etc/grub.d/`: GRUB 菜单生成脚本,按编号顺序执行
  - `05_debian_theme`: 设置主题/背景
  - `10_linux`: 检测 Linux 内核
  - `20_memtest86+`: 添加内存测试条目
  - `30_os-prober`: 检测其他操作系统(如 Windows)
- `/boot/grub/grub.cfg`: 最终生成的配置文件,由 `update-grub` 自动生成,不应手动编辑

os-prober: 一个工具,用于扫描磁盘上的其他操作系统并生成对应的 GRUB 菜单项。Ubuntu 24.04 默认禁用了 os-prober(安全考虑),需要手动在 `/etc/default/grub` 中添加 `GRUB_DISABLE_OS_PROBER=false` 来启用。

### 2.4 chroot 技术

chroot(Change Root): 改变当前进程的根目录。在 Live USB 环境中修复已安装系统时,需要将已安装系统的根分区挂载到 `/mnt`,然后通过 `chroot /mnt` "进入"该系统,这样执行的命令(如 `grub-install`)就会像在已安装系统中运行一样。

chroot 的必要挂载: 要让 chroot 环境正常工作,需要绑定挂载以下虚拟文件系统:

- `/dev`: 设备文件(使 chroot 能访问硬件设备)
- `/dev/pts`: 伪终端设备
- `/proc`: 进程信息虚拟文件系统
- `/sys`: 系统硬件信息虚拟文件系统
- `/run`: 运行时数据

本次安装中遇到了 `-B`(bind mount)不够的问题——NVMe 设备节点在 `/dev` 的子挂载点中,需要使用 `--rbind`(recursive bind)才能正确传递。

---

## 三、使用的终端命令详解
**磁盘信息查看**
```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,LABEL
```
- `lsblk`: **List Block devices**,列出所有块设备(磁盘、分区、loop 设备等)
- `-o`: 指定输出的列。NAME(设备名)、SIZE(大小)、FSTYPE(文件系统类型)、MOUNTPOINT(挂载点)、LABEL(卷标)
```bash
sudo fdisk -l
```
- `fdisk`: 磁盘分区工具
- `-l`: 列出所有磁盘的分区表信息,包括分区类型(GPT/MBR)、每个分区的起止扇区、大小、类型
```bash
df -h
```
- `df`: **Disk Free**,显示文件系统的磁盘空间使用情况
- `-h`: human-readable,以 GiB/MiB 等人类可读单位显示

**网卡信息查看**
```bash
lspci | grep -i network
```
- `lspci`: 列出所有 PCI/PCIe 设备(包括显卡、网卡、NVMe 控制器等)
- `|`: 管道符,把前一个命令的输出传给后一个命令
- `grep -i network`: 过滤包含 "network" 的行,`-i` 表示忽略大小写

**UEFI 模式检测**
```bash
ls /sys/firmware/efi 2>/dev/null && echo "UEFI模式" || echo "Legacy BIOS模式"
```
- `ls /sys/firmware/efi`: 查看 EFI 固件接口目录是否存在
- `2>/dev/null`: 把标准错误(stderr)重定向到 /dev/null(丢弃错误信息)
- `&&`: 前一个命令成功则执行后面的
- `||`: 前一个命令失败则执行后面的

**GRUB 安装与修复**
```bash
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu
```
- `grub-install`: 安装 GRUB 引导加载器
- `--target=x86_64-efi`: 指定目标平台为 64 位 UEFI
- `--efi-directory=/boot/efi`: 指定 EFI 系统分区的挂载点
- `--bootloader-id=ubuntu`: 在 ESP 中创建 `/EFI/ubuntu/` 目录并设置 UEFI 启动项名称为 "ubuntu"
```bash
sudo update-grub
```
- 实际上是 `grub-mkconfig -o /boot/grub/grub.cfg` 的简写
- 读取 `/etc/default/grub` 和 `/etc/grub.d/` 中的脚本,生成最终的 GRUB 配置文件
- 会自动检测已安装的 Linux 内核和其他操作系统(如果 os-prober 启用)

**chroot 相关操作**
```bash
sudo mount /dev/nvme0n1p2 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
```
- `mount`: 将文件系统挂载到目录树中的某个位置
- 先挂载根分区到 `/mnt`,再挂载 EFI 分区到 `/mnt/boot/efi`(必须按顺序)
```bash
sudo mount --rbind /dev /mnt/dev
sudo mount --rbind /sys /mnt/sys
sudo mount --rbind /proc /mnt/proc
sudo mount --rbind /run /mnt/run
```
- `--rbind`: 递归绑定挂载,会连同子挂载点一起绑定
- 比 `-B`(`--bind`)更彻底,解决了 NVMe 设备节点在子挂载点中不可见的问题
```bash
sudo chroot /mnt
```
- 将根目录切换为 `/mnt`,之后的所有操作都在已安装系统的环境中执行
```bash
sudo umount -R /mnt
```
- `-R`:递归卸载 `/mnt` 下的所有挂载点

**文件下载**
```bash
wget -P ~/Downloads https://releases.ubuntu.com/24.04/ubuntu-24.04.2-desktop-amd64.iso
```
- `wget`: 命令行下载工具
- `-P ~/Downloads`: 指定下载保存目录

**GRUB 配置修改**
```bash
echo 'GRUB_DISABLE_OS_PROBER=false' | sudo tee -a /etc/default/grub
```
- `echo`: 输出字符串
- `| sudo tee -a`: 以 root 权限追加内容到文件。不能用 `sudo echo ... >> file`,因为重定向 `>>` 是由当前 shell 处理的,不受 sudo 影响
- `-a`: append 模式(追加而非覆盖)
```bash
sudo sed -i 's|GRUB_THEME=.*|GRUB_THEME=/boot/grub/themes/stylish/theme.txt|' /etc/default/grub
```
- `sed`: 流编辑器(**Stream Editor**)
- `-i`: 直接修改文件(in-place)
- `'s|旧内容|新内容|'`: 替换操作,这里用 `|` 作为分隔符(因为路径中包含 `/`)
```bash
sudo chmod -x /etc/grub.d/05_debian_theme
```
- `chmod`: 修改文件权限
- `-x`: 去掉可执行权限
- 效果: `update-grub` 不会再执行这个脚本,从而不会覆盖自定义主题设置
```bash
grep -i theme /boot/grub/grub.cfg
```
- `grep`: 文本搜索工具
- `-i`: 忽略大小写
- 用于验证生成的 GRUB 配置中是否包含正确的主题设置

**Plymouth 配置**
```bash
sudo sed -i 's/quiet splash/quiet/' /etc/default/grub
```
- 从内核启动参数中去掉 `splash`,禁用 Plymouth 启动画面
- `quiet`: 隐藏内核启动日志
- `splash`: 显示图形化启动画面

**GRUB 主题安装**
```bash
sudo mkdir -p /boot/grub/themes
sudo cp -r /usr/share/grub/themes/stylish /boot/grub/themes/stylish
```
- `mkdir -p`: 创建目录,`-p` 表示递归创建(父目录不存在时自动创建)
- `cp -r`: 递归复制目录及其所有内容

**Windows 安装时的命令**
```
oobe\bypassnro
```
- 在 Windows OOBE(Out-Of-Box Experience,开箱体验)阶段执行
- 修改注册表以允许跳过网络连接要求,然后自动重启安装程序
- 通过 `Shift + F10` 打开命令提示符后输入

---

## 四、遇到的问题与解决方案

### 问题 1: Windows 安装时无法连接 WiFi(错误 0xe0000100)

原因: Windows 11 安装镜像不包含 MediaTek MT7921 的驱动。

解决: 通过 `Shift + F10` → `oobe\bypassnro` 跳过联网要求,安装完成后通过手机 USB 共享网络获取驱动。

### 问题 2: 安装后 GRUB 菜单不显示

原因: Windows 安装时将自己的 Windows Boot Manager 设置为 UEFI 第一启动项,跳过了 GRUB。

解决: 使用 `efibootmgr -o 0004,0000` 将 Ubuntu 重新设为第一启动项。

---

## 五、启动流程
```
电源开机
  → UEFI 固件初始化硬件
  → 读取 BootOrder,找到 Ubuntu 启动项
  → 加载 /EFI/ubuntu/shimx64.efi
  → shim 加载 grubx64.efi
  → GRUB 读取 /boot/grub/grub.cfg
  → 显示启动菜单(Ubuntu / Windows)
  → 用户选择:
      Ubuntu → 加载 Linux 内核 → 启动系统
      Windows → 链式加载 Windows Boot Manager → 启动 Windows
```

