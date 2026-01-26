---
author: openbit
pubDatetime: 2026-01-26T01:20:00Z
title: 使用 WSL2 远程开发方案的探讨
postSlug: wsl-remote-setup-guide
featured: true
draft: false
tags:
  - Linux
  - WSL
  - windows
  - dev
  - VSCode 
description: 本文档将详细给出这两种方案的完整配置步骤、关键注意事项、性能对比及常见问题解决方法，帮助你在实际环境中快速选择并上手最适合自己的方式。
---

**引言**

 在远程访问，尤其是局域网中，使用 WSL2 结合 VS Code 远程开发项目主要有两种成熟且实用的方案，都已被广泛验证且适合不同使用偏好：

- **方案一：VS Code Remote - SSH**  
  通过 SSH 直接连接 WSL2 中的 SSH 服务。该方式延迟极低、文件读写接近原生性能、资源占用少、支持多设备同时连接，是目前官方文档和社区最常推荐的远程开发路径，尤其适合专注于代码编辑、编译、调试、终端操作的开发流程。

- **方案二：远程桌面 (RDP) + Windows VS Code + Remote - WSL**  
  先通过 RDP 登录 Windows 主机桌面，再启动 Windows 版的 VS Code 并使用 Remote - WSL 扩展接入 Linux 环境。该方案能提供完整的 Windows 桌面体验、原生字体渲染、DPI 适配以及方便同时使用 Windows GUI 工具，适合对 Windows 原生界面有较高需求或需要 Windows 生态软件配合开发的场景。

两种方案在局域网环境下均稳定可靠，没有绝对的优劣之分，主要取决于你的具体需求：  
- 如果追求最低延迟、最流畅的代码体验 → 优先 Remote-SSH  
- 如果更喜欢 Windows 原生桌面感、需要 GUI 工具无缝配合 → 优先 RDP 方案  

本文档将详细给出这两种方案的完整配置步骤、关键注意事项、性能对比及常见问题解决方法，帮助你在实际环境中快速选择并上手最适合自己的方式。下面先给优缺点对比，然后给出两种完整配置路径。

### 方案对比：RDP vs VS Code Remote-SSH（针对 WSL2 开发）

| 项目                  | RDP + Windows VS Code + Remote-WSL          | VS Code Remote-SSH 直接连 WSL                     | 备注 |
|-----------------------|-----------------------------------------------|---------------------------------------------------|------|
| 图形界面体验          | 完整 Windows 桌面 + Windows 版 VS Code       | 通常只有 VS Code 窗口（可加 X11/GUI 转发）       | RDP 胜 |
| 输入延迟（键盘/鼠标） | 中等～较高（RDP 压缩 + 网络）                 | 很低（VS Code 协议优化）                          | Remote-SSH 明显更好 |
| 文件 I/O 性能         | Windows → WSL 文件系统有损耗                  | 直接在 WSL 内，几乎无损耗                         | Remote-SSH 胜 |
| GPU / GUI 应用支持    | 完整（可跑 Windows GUI + WSLg GUI）           | 有限（需额外 X11/VcXsrv 或 WSLg RDP）            | RDP 胜 |
| 配置复杂度            | ★★★☆☆                                         | ★★☆☆☆                                             | Remote-SSH 更简单 |
| 资源占用（主机侧）    | 高（开完整 RDP 会话 + VS Code）               | 低（只跑 VS Code Server）                         | Remote-SSH 胜 |
| 局域网多设备同时开发  | 较差（RDP 默认单会话，除非多用户/第三方）     | 优秀（可多个 VS Code 同时连）                     | Remote-SSH 胜 |
| 最适合场景            | 需要完整 Windows 桌面 + GUI 工具链           | 纯代码开发 / 测试 / 终端密集型工作                | — |



### 方案一：主流推荐 - VS Code Remote-SSH 直接连 WSL（局域网）

#### 在 Windows 主机（有 WSL2 的机器）上准备

1. 确保 WSL 已安装 openssh-server

   ```bash
   # 在 WSL 内执行
   sudo apt update
   sudo apt install openssh-server -y

   # 编辑配置（可选，但推荐）
   sudo nano /etc/ssh/sshd_config
   # 修改或确认下面几行：
   Port 22
   PermitRootLogin prohibit-password   # 或 yes，如果你允许 root
   PasswordAuthentication yes
   PubkeyAuthentication yes

   sudo service ssh restart
   # 或 sudo systemctl enable --now ssh （如果用了 systemd）
   ```

2. （重要）用 netsh 把 SSH 端口转发到局域网（因为 WSL NAT）

 - 以管理员 PowerShell 执行（假设你想用 2222 避免与 Windows SSH 冲突）：

```powershell
   netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=$(wsl hostname -I | ForEach-Object { $_.Trim() -split '\s+' | Select-Object -First 1 })
```

- 防火墙放行：

```powershell
New-NetFirewallRule -DisplayName "WSL SSH 2222" -Direction Inbound -Protocol TCP -LocalPort 2222 -Action Allow
```

3. 获取 Windows 的局域网 IP（例如 192.168.50.100）

#### 在远程电脑（你的开发机，Windows / macOS / Linux 都可）上操作

1. 安装 VS Code + Remote - SSH 扩展
2. 按 F1 → Remote-SSH: Connect to Host... → Add New SSH Host...

- 输入：
```
ssh -p 2222 your_wsl_username@192.168.50.100
```

3. 第一次连接会要密码（或配置公钥免密）
4. 连接成功后 → 打开文件夹 → 选择 WSL 内的项目路径（/home/xxx/project）

**后续**：每次直接从 VS Code 选这个 host 连接即可，体验接近本地。

### 方案二：使用远程桌面 (RDP) + Windows VS Code + Remote-WSL

#### Windows 主机侧准备

1. 启用远程桌面

   - 设置 → 系统 → 远程桌面 → 启用远程桌面
   - 确认允许远程连接（NLA 可选，但建议开启）
   - 记下计算机名称或直接用 IP

2. 确保 Windows 防火墙允许 RDP（默认 3389 端口已开）

3. （推荐）设置固定 WSL IP 或用脚本更新（可选，避免 WSL IP 变动影响开发）

4. 确认 VS Code 已安装 Remote - WSL 扩展

#### 远程电脑侧（你的开发笔记本/台式机）

1. 用 Windows 自带“远程桌面连接” (mstsc.exe) 或 macOS 的 Microsoft Remote Desktop 连接到 Windows 主机的 IP:3389

   - 输入 Windows 用户名 + 密码登录
   - 得到完整的 Windows 桌面

2. 在这个远程桌面里打开 VS Code（Windows 版）

3. 在 VS Code 左下角点击绿色 “><” 图标 → “New WSL Window” 或 “Connect to WSL”

4. VS Code 会自动在 WSL 中启动 VS Code Server，你的项目就在 WSL 环境里运行了

   - 文件浏览器、终端、调试、扩展都运行在 WSL 内
   - 但 UI 是 Windows 版的 VS Code（通过 RDP 显示）

**优点**：你看到的 VS Code 是“原生 Windows 版”，字体渲染、DPI、主题等都和本地一样。  
**缺点**：需要 Windows 11 Pro 或第三方工具支持，由于整个桌面通过 RDP 传输，鼠标键盘有轻微延迟；如果网络抖动，体验下降明显。

#### 额外优化建议（RDP 方案）

- RDP 设置里把“体验”调成“局域网 (LAN)” 或 “高速宽带”
- 关闭不必要的视觉效果（壁纸、字体平滑等）
- 如果延迟仍高，可尝试降低分辨率或用 16-bit 色深
- 主机侧可安装 **RDP Wrapper** 或用 Windows 多会话补丁（非官方）实现多用户同时 RDP（企业/教育版原生支持）

### 最终推荐排序（局域网开发 WSL + VS Code）

1. **首选**：Remote-SSH 直接连 WSL（延迟最低、最省资源、最符合 VS Code 远程开发设计逻辑）
2. **次选**：RDP + Windows VS Code + Remote-WSL（当你非常需要 Windows 原生 UI 体验时）
3. **不推荐**：在 WSL 里装 code-server / openvscode-server 然后浏览器访问（GUI 体验差）

