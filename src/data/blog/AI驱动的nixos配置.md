---
author: openbit
pubDatetime: 2026-01-28T00:00:00Z
title: VS Code 配置 `mcp-nixos` 指南
postSlug: 离线设计、离线验证、远程投送
featured: true
draft: false
tags:
  - Linux
  - Nix
  - MCP
description: 在 Windows 上通过 VS Code 搭建环境，你可以利用 Windows 的便利性进行“离线设计”，然后通过 Git 把配置“远程投送”给 NixOS。以下是在 Windows VS Code 中配置 mcp-nixos 的详细手把手步骤：
---



###  VS Code 配置 `mcp-nixos` 指南

>在 Windows 上通过 VS Code 搭建环境，你可以利用 Windows 的便利性进行“离线设计”，然后通过 Git 把配置“远程投送”给 NixOS。以下是在 Windows VS Code 中配置 mcp-nixos 的详细手把手步骤：


---

#### 第一步：准备 Python 运行环境 (uv)

在 Windows 上，直接用传统的 Python 可能会遇到路径和环境变量问题。我们使用目前最火的 Python 工具 **uv**，它能自动处理所有依赖。

1. **检查/安装 Python**
   在 Windows 终端输入 python --version。如果没有安装，请去 [Python 官网](https://www.python.org/) 下载安装（记得勾选 "Add Python to PATH"）。

2. **安装 uv**：打开 PowerShell，粘贴并运行以下命令：
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

```


3. **验证安装**：重启你的终端（PowerShell），输入 `uv --version`。如果看到版本号，说明成功。

#### 第二步：安装 VS Code 插件 Cline

1. 在 VS Code 插件市场搜索 **Cline** 并安装。
2. 点击左侧边栏出现的 Cline 图标（通常是一个机器人或方块图标）。

#### 第三步：配置 MCP 服务器

这是核心步骤。我们需要让 Cline 知道如何调用 `mcp-nixos` 这个“脑库”。

1. 进入 Cline 界面，点击 **Settings (齿轮图标)**。
2. 向下滚动找到 **MCP Servers**，点击 **Edit MCP Settings**。
3. 在打开的 `cline_mcp_settings.json` 文件中，删除所有内容，替换为以下配置：

```json
{
  "mcpServers": {
    "nixos": {
      "command": "uvx",
      "args": [
        "mcp-nixos"
      ],
      "env": {
        "USE_NIX": "false"
      }
    }
  }
}

```

> **原理解析**：
> * **`uvx`**：类似于 npm 的 npx，它会自动下载 `mcp-nixos` 的最新版并在临时环境中运行，不污染你的系统。
> * **`USE_NIX: "false"`**：这非常关键！因为你在 Windows 上，没有本地的 `nix` 命令。设置为 `false` 后，它会通过网络 API 去查询 search.nixos.org 的实时数据。



#### 第四步：激活与测试

1. 保存 JSON 文件并关闭。
2. 回到 Cline 对话窗口，你应该能看到下方出现了一个带绿色圆点的 **"nixos"**。
3. **测试提问**：
> “请调用 nixos 工具，搜索一下在 NixOS 中如何配置 Zsh 和 Oh My Zsh，并给我代码建议。”


4. 如果 Cline 显示 **"Taking action..."** 并展示了搜索到的 Package 或 Option 信息，说明配置完美成功。

---

#### 第五步：开始构建你的 NixOS 仓库

现在你可以正式开始“云端练兵”了：

1. **建立本地文件夹**：在 Windows 上新建 `D:\NixConfig`，并在 VS Code 中打开。
2. **让 AI 动笔**：在 Cline 中输入：
>我有一个使用最小化的全新安装后的nixos，请在这个文件夹里帮我建立一个基础的 NixOS Flake 项目骨架，采用nix的标准模块化设计，包含 flake.nix 和 home-manager 配置，并引用nixos默认生成的configuration.nix 和硬件配置。

>使用用户名`jadmin`，主机名`my-nixos`，

3. **推送到 GitHub 或 Gitee**：
* 在 **Gitee** 上建一个仓库。
* 在 Windows 终端执行：
```powershell
git init
git add .
git commit -m "initial commit from ai"
git branch -M main
git remote add origin <你的仓库地址>
git push -u origin main

```


---

### 下一步操作

你可以尝试让 Cline 生成一段 **Kitty** 终端的配置。由于它有 `mcp-nixos` 插件，你可以问它：

> “查询一下 Home Manager 中 Kitty 终端的所有可用选项，并帮我写一份包含半透明效果、特定字体和快捷键自定义的 `kitty.nix` 模块。”

**这样生成的配置，当你回到 NixOS 机器上执行 `git pull` 后，就能立即生效。**





