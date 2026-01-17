---
author: openbit-j
pubDatetime: 2026-01-17T09:00:00Z
title: Linux 上正确安装和配置 JACK
postSlug: Linux-Audio
featured: true
draft: false
tags:
  - Linux
description: Ubuntu Studio 音频软件包
---




## 四步安装和配置 JACK（适用于基于 Debian/Ubuntu 的系统）

以下步骤以 **Zorin OS** 为例，这是一个全新的安装。

### 步骤一：更新软件包列表

在终端中执行以下命令：

Bash

```
sudo apt update
```

输入密码并等待软件包列表更新。这能确保你下载最新版本的 JACK。

---

### 步骤二：安装 JACK、GUI 和 PipeWire 集成

同时安装 **JACK (jackd2)**、**图形用户界面 (qjackctl)** 以及最重要的 **PipeWire JACK 集成 (pipewire-jack)**。

Bash

```
sudo apt install jackd2 qjackctl pipewire-jack
```

系统提示是否继续时，输入 `y` 并回车。安装过程中，JACK 可能会询问关于**实时权限**的问题，选择 **“是”** 并回车。

(可选测试)：

此时，如果你直接打开 QJackCtl 并点击“启动”，它仍然会看不到任何设备。但如果你通过 PipeWire 运行它：

Bash

```
pw-jack qjackctl
```

它就能正常工作，显示 OBS、你的声卡、内置音频等所有设备。但这只是一个临时测试，我们希望直接点击图标就能实现这个效果。

---

### 步骤三：重命名原始 JACK 库

我们需要将原始 JACK 调用的库文件重命名，阻止系统使用它。

原始库文件位于 `/usr/lib/` 文件夹内，名称是 `libjack.so.0`。

使用 **`sudo mv`** 命令来重命名（本质上就是移动到一个不同的名称）：

Bash

```
sudo mv /usr/lib/libjack.so.0 /usr/lib/libjack.so.0.bak
```

**说明：** 这将原始库文件重命名为 `libjack.so.0.bak`，将其备份，这样 JACK 图标就找不到它了。如果你现在尝试打开 JACK，它会因为找不到库而无法启动。

---

### 步骤四：创建 PipeWire JACK 库的软链接

最后一步是创建一个**符号链接 (symlink)**，将 PipeWire 的 JACK 集成库链接到系统期望找到原始 JACK 库的位置和名称。

PipeWire 的 JACK 库位于 `/usr/lib/pipewire-0.3/jack/libjack.so`。

使用 **`sudo ln -s`** 命令创建软链接：

Bash

```
sudo ln -s /usr/lib/pipewire-0.3/jack/libjack.so /usr/lib/libjack.so.0
```

**说明：**

- `ln -s`：创建一个符号链接。
    
- `/usr/lib/pipewire-0.3/jack/libjack.so`：这是**目标文件**（PipeWire 库）。
    
- `/usr/lib/libjack.so.0`：这是**链接名称**（系统期望的原始 JACK 库名称）。
    

现在，当你点击 JACK 图标时，它会调用 `libjack.so.0`，但实际上运行的是 PipeWire 的 JACK 集成，从而实现**无缝的音频管理**。

---

## 额外建议

**最佳安装顺序：**

为了避免像视频中那样，在安装其他 JACK 应用程序（如 Jack Mixer、JACK MIDI 键盘）后需要重复步骤三和步骤四，**建议你先安装所有需要的 JACK 应用程序，然后一次性执行步骤三（重命名）和步骤四（创建链接）**。

例如，你可以将所有需要的 JACK 应用程序一次性安装：

Bash

```
sudo apt install jackd2 qjackctl pipewire-jack jack-mixer jack-keyboard zam-mu1
```

然后再执行步骤三和步骤四。

---

### 总结命令

如果你想快速复制粘贴，这是四个关键命令的总结：

1. **更新列表:** `sudo apt update`
    
2. **安装核心包:** `sudo apt install jackd2 qjackctl pipewire-jack` (可添加其他 JACK 应用程序)
    
3. **重命名旧库:** `sudo mv /usr/lib/libjack.so.0 /usr/lib/libjack.so.0.bak`
    
4. **创建软链接:** `sudo ln -s /usr/lib/pipewire-0.3/jack/libjack.so /usr/lib/libjack.so.0`
    

---

这些步骤完成后，你的 JACK 应用程序应该能通过 PipeWire 正常运行，并能看到系统中的所有音频输入、输出和应用程序。