---
author: openbit
pubDatetime: 2026-01-17T13:20:00Z
title: Niri 安装及配置指南
postSlug: niri-wayland-setup-guide
featured: true
draft: false
tags:
  - Linux
  - ArchLinux
  - Niri
  - Wayland
description: 现代 Wayland 平铺窗口管理器的桌面体验，在 arch Linux上 从 yay 安装到 Niri 显示器、壁纸及中文化环境的完整配置教程。
---



### 安装yay

#### 设置golang的加速（可选）
```bash
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
```

- 安装git
```bash
sudo pacman -S git
```
- 克隆代码
```
 git clone https://aur.archlinux.org/yay.git
```
- 编译yay
```bash
 cd yay/
 makepkg -si
```

### 安装niri必要工具
```bash
sudo pacman -Sy alacritty fuzzel swaybg firefox ttf-jetbrains-mono-nerd xwayland-sattelite
```


### 显示器设置

- 编辑niri配置文件
```bash
vim ~/config/niri/config.kdl
```

- 如果是虚拟机，增加显示器
```js
output "Virtual-1" { //虚拟机专用
 mode "1920x1080@60.00"
 scale 1
 transform "normal"
 position x=0 y=0
}
```
- [光标样式](https://github.com/ful1e5/Bibata_Cursor)

```js

cursor {
    xcursor-theme "Bibata-Modern-Ice"
    xcursor-size 24
    // hide-when-typing
    // hide-after-inactive-ms 1000
}
```

### 简单壁纸设置

```js
spawn-sh-at-startup "swaybg -i ~/Pictures/wallpapers/01.png"
```
#### 下载壁纸
```bash
wget https://gitee.com/openbit/notes/blob/master/wallpapers/pexels-photo-12106722.png -O ~/Pictures/wallpapers/01.png
```

![壁纸](https://images.pexels.com/photos/12106722/pexels-photo-12106722.jpeg)


#### 设置壁纸

编辑`niri`配置文件
```bash
vim ~/.config/niri/config.kdl
```
1.  **设置工作区透明度 (在 `layout` 块内部):**
```bash
background-color "#00000000"
```
2.  **设置概览背景图片 (在文件顶层):**
```bash
spawn-sh-at-startup "sleep 1 && swaybg -i ~/Pictures/wallpapers/01.png -m fill"
```
3.  **隔离概览背景 (在文件顶层):**

```bash
layer-rule { 
     match namespace=r#"^wallpaper$"# 
     place-within-backdrop true
 }
```

4. **随机设置桌面壁纸(可选)**

`autostart.sh`

```bash
#! /bin/bash
# swaybg -i $(find ~/Pictures/wallpapers/ -type f | shuf -n1) -m fill
```

### 中文化

- Locale启用中文处理

```bash
sudo nano /etc/locale.gen  
sudo locale-gen
```

- 字体显示中文字符
```bash
sudo pacman -S noto-fonts ttf-dejavu adobe-source-han-sans-cn-fonts
```

- 输入法输入中文
```bash
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-chinese-addons 
```

- 在 /etc/environment 中添加环境变量(可选)
```bash
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

#### 其它可选安装
- 安装ble.sh(Bash Line Editor)
```bash
yay -S blesh-git
```
