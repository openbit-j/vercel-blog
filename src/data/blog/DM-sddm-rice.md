---
author: openbit
pubDatetime: 2026-01-19T09:00:00Z
title: SDDM 登录管理器美化完全指南
postSlug: 与现代桌面环境完美兼容的 Display Manager
featured: true
draft: false
tags:
  - Linux
  - DM
description: Arch Linux 社区公认对 Wayland 支持最好的登录管理器、主题扩展性极强，支持 QML 开发、稳定可靠，与现代桌面环境完美兼容
---



## 一、SDDM 核心优势
- Arch Linux 社区公认对 Wayland 支持最好的登录管理器
- 主题扩展性极强，支持 QML 开发
- 稳定可靠，与现代桌面环境完美兼容

## 二、主题推荐

| 主题 | 风格特点 | 适配程度 | 安装命令 |
|------|----------|----------|----------|
| **Sugar Candy** | 极简、毛玻璃、高清背景 | ⭐⭐⭐⭐⭐ | `yay -S sddm-sugar-candy-git` |
| **WhiteSur** | 仿 macOS 精致风格 | ⭐⭐⭐⭐ | `yay -S sddm-theme-whitesur-git` |
| **Chili** | 现代、简洁、大字体 | ⭐⭐⭐⭐ | `yay -S sddm-theme-chili-git` |
| **Corners** | 圆角卡片式设计 | ⭐⭐⭐⭐⭐ | `yay -S sddm-theme-corners-git` |

> **推荐首选**：`sddm-sugar-candy-git`，带有毛玻璃效果的现代风格

## 三、完整安装配置流程

### 1. 基础安装
```bash
# 安装 SDDM 及 Wayland 支持
sudo pacman -S sddm qt6-wayland kwayland
sudo systemctl enable sddm
```

### 2. 安装主题（以 Sugar Candy 为例）
```bash
# 通过 AUR 安装
yay -S sddm-sugar-candy-git
```

### 3. 应用主题配置
```bash
# 创建配置文件
sudo sddm --example-config | sudo tee /etc/sddm.conf

# 编辑配置文件
sudo nano /etc/sddm.conf
```

在配置文件中修改：
```ini
[Theme]
Current=sugar-candy
```

### 4. 虚拟机强制支持分辨率（1080p）

检查显示器
```bash
# 安装xorg-xrandr专门用于管理屏幕分辨率、旋转和多显示器的命令行工具
sudo pacman -S xorg-xrandr
xrandr
```

 创建或者分辨率设置脚本
```
sudo nano /usr/share/sddm/scripts/Xsetup
```

添加内容：
```bash
#!/bin/sh
xrandr --output Virtual-1 --mode 1920x1080
```

设置执行权限：
```bash
sudo chmod +x /usr/share/sddm/scripts/Xsetup
```

### 5. Wayland 模式支持（可选）--兼容性差
在 `/etc/sddm.conf` 中启用 Wayland 模式：
```ini
[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=layer-shell
```
检查显示器
```bash
# 安装wlr-randr用于管理屏幕分辨率、旋转和多显示器的命令行工具
sudo pacman -S wlr-randr 
wlr-randr 

```

编辑分辨率设置脚本
```bash
sudo nano /usr/share/sddm/scripts/Xsetup
```

添加内容：
```bash
#!/bin/sh
wlr-randr --output Virtual-1 --mode 1920x1080
```


## 四、主题预览与测试
```bash
# 预览主题效果（无需注销）
sddm-greeter --test-mode --theme /usr/share/sddm/themes/sugar-candy
```

## 五、高级定制技巧

### 1. 统一壁纸设置
- 将主壁纸复制到：`/usr/share/backgrounds/`
- 编辑主题配置文件：
```bash
sudo nano /usr/share/sddm/themes/sugar-candy/theme.conf
```
查找并修改：
```ini
Background=/usr/share/backgrounds/your-wallpaper.jpg
his```

### 2. 调整界面布局
在 `theme.conf` 中可调整：
```ini
[General]
FontSize=10
BackgroundHDPI=2.0
LoginButtonText=登录
```

### 3. 自定义用户头像
- 用户头像位置：`~/.face` 或 `~/.face.icon`
- 系统默认头像：`/usr/share/sddm/faces/`

## 六、常见问题解决

### 1. 主题不显示
```bash
# 检查主题安装路径
ls /usr/share/sddm/themes/

# 重启 SDDM 服务
sudo systemctl restart sddm
```

### 2. 分辨率设置不生效
```bash
# 检查脚本权限
ls -la /usr/share/sddm/scripts/

# 手动测试脚本
sudo /usr/share/sddm/scripts/Xsetup
```

### 3. Wayland 模式黑屏
```bash
# 临时回退到 X11
sudo nano /etc/sddm.conf
```
注释掉或删除：
```ini
DisplayServer=wayland
```

## 七、快速恢复命令
```bash
# 重置到默认主题
sudo sed -i 's/Current=.*/Current=/g' /etc/sddm.conf

# 重启显示管理器
sudo systemctl restart sddm
```

## 八、最佳实践建议
1. **备份配置**：修改前备份 `/etc/sddm.conf`
2. **测试模式**：使用 `--test-mode` 预览效果
3. **版本控制**：将自定义主题配置加入 Git
4. **性能监控**：观察 SDDM 内存占用

![截屏](https://gitee.com/openbit/notes/tree/master/preview/屏幕截图2026-01-03-130534.png)

## 九、主题开发资源
- 官方文档：https://github.com/sddm/sddm
- QML 教程：https://doc.qt.io/qt-6/qmlapplications.html
- 主题示例：`/usr/share/sddm/themes/` 下的示例代码

---

**💡 提示**：Sugar Candy 主题支持完整的壁纸模糊、颜色覆盖、时间格式定制等功能，可以通过编辑其 `theme.conf` 文件深度个性化你的登录界面，实现桌面的完美视觉统一。


## 参考

#### 参考配置
- [配置文件](https://gitee.com/openbit/notes/tree/master/sddm-config)

#### sddm仓库
- [sddm仓库](https://github.com/sddm/sddm)
- [sddm-sugar-candy](https://github.com/Kangie/sddm-sugar-candy)


#### 壁纸

![壁纸](https://gitee.com/openbit/wsl-sway/raw/master/background/gh01.png)





