---
author: openbit
pubDatetime: 2026-01-18T09:00:00Z
title: 零成本的个人博客
postSlug: 🚀 AstroPaper 从零到一完美建站指南
featured: true
draft: false
tags:
  - Linux
  - Astro
  - Vercel
  - Cloudflare
description: 本指南记录了从 GitHub-Vercel-AstroPaper 部署到全站深度中文化的完整流程，旨在为使用 vercel 部署轻量博客，并使用 AstroPaper 模板，提供一份零成本、极简、准确且易于执行的部署指南。
---


对于 AstroPaper 博客在 Vercel 上的部署，**最合适、最干净、长期维护最容易的方法**：直接使用 `create astro` 命令以模板方式创建项目。

这是官方和 AstroPaper 作者强烈推荐的方式（2026 年最新 v5.5.1 版本依然如此），它能给你一个**零历史污染**的干净起点，后续想拉取上游更新也简单（加 remote 后 pull 就行）。

### 第一阶段：部署（本地 + GitHub + Vercel）

1. **在本地创建一个新项目（用 pnpm 或 npm，根据你习惯）**  
   AstroPaper README 首推 pnpm（更快、更省空间），但 npm 也完全 OK。
   ```bash
   # 方式一：用 pnpm（推荐）
   pnpm create astro@latest -- --template satnaing/astro-paper

   # 方式二：用 npm（如果你没装 pnpm）
   npm create astro@latest -- --template satnaing/astro-paper

   # 方式三：用 bun（如果你用 bun）
   bun create astro@latest --template satnaing/astro-paper
   ```

   - 命令执行后会问你项目名字（例如：my-blog）
   - 选 TypeScript（推荐）
   - 完成后会自动进入项目目录并提示下一步

2. **进入项目目录，安装依赖 & 本地跑起来测试**

   ```bash
   cd my-blog          # 替换成你起的名字
   pnpm install        # 或 npm install
   pnpm run dev        # 或 npm run dev
   ```

   测试：浏览器打开 http://localhost:4321，应该看到 AstroPaper 默认首页。

3. **初始化 Git 并推送到你的 GitHub 仓库**

   先在 GitHub 上新建一个空仓库（例如 vercel-blog，不要加 README/.gitignore）

   然后在本地：

```bash
   git init
   git add .
   git commit -m "Initial commit: AstroPaper v5.5.1 setup"
   
   git remote add origin https://github.com/你的仓库名字/vercel-blog.git   # 替换成你的仓库 URL
   git branch -M main
   git push -u origin main
```

4. **在 Vercel 上部署（网页方式，零配置）**

5. 打开 https://vercel.com/dashboard → “Add New...” → Project
6. Import Git Repository → 授权 GitHub → 搜索并选你的（vercel-blog）仓库
7. 点击 Import
8. **所有设置保持默认**（Vercel 自动识别 Astro）：
      - Framework Preset：Astro
      - Build Command：留空（默认 astro build）
      - Output Directory：留空（默认 dist）
      - Install Command：留空（默认 npm install / pnpm install）
9. 点击 Deploy（通常 30–90 秒）
   部署成功后立即得到域名：  
   https://vercel-blog.vercel.app （或类似）

10. **写第一篇文章 & 自动部署**

   - 文章放在 `src/data/blog/` 目录（.md 或 .mdx 文件）
   - Frontmatter 格式示例（和 AstroPaper 文档一致）：

```markdown
     ---
     author: 你的名字
     pubDatetime: 2026-01-19T17:00:00Z
     title: 我的第一篇 Astro 博客
     featured: true
     draft: false
     tags:
       - Astro
       - Vercel
     description: 使用 AstroPaper + Vercel 快速搭建个人博客的完整流程。
     ---

     这里是正文内容，支持 Markdown 和 MDX。
```

   - 修改后 git add/commit/push → Vercel 自动重新构建 & 部署（很快）

11. **完成后，你应该可以通过 Vercel 提供的临时域名正常访问博客了：**
	- 生产域名：https://my-blog-你的名字.vercel.app
	- 每个 git push 都会自动构建 & 部署

---

## 第二阶段：进阶配置（自定义域名）

域名获取与自定义解析 (Cloudflare + Vercel)：在 Vercel 临时域名访问正常后，即可通过以下步骤绑定你的专属域名。

### 1. 域名获取与 DNS 接管

1. **域名注册**：前往 [DigitalPlat](https://domain.digitalplat.org/) 申请免费域名（例如：`xxxx.dpdns.org`）。
2. **Cloudflare 接管**：
	* 登录 [Cloudflare](https://dash.cloudflare.com/)，点击 **Add a Site**，输入你申请的域名。
	* **修改 NS 记录**：回到 DigitalPlat 域名管理后台，将默认的名称服务器（Name Servers）修改为 Cloudflare 提供的地址（例如：`hal.ns.cloudflare.com` 和 `heather.ns.cloudflare.com`）。
	* **等待生效**：修改 NS 记录后，通常需要 10 分钟至几小时不等，直到 Cloudflare 显示“站点已激活”。

### 2. Vercel 绑定与 Cloudflare 解析配置

1. **Vercel 添加域名**：
* 进入 Vercel 项目后台，点击 **Settings > Domains**。
* 输入你的新域名并点击 **Add**。


2. **添加解析记录**：
- 在 Cloudflare 的 DNS 设置页面中，根据 Vercel 的提示添加以下记录：
	* **A 记录**：名称为 `@`，IP 地址指向 Vercel 提供的服务器地址（通常为 `76.76.21.21`）。
	* **TXT 记录**：按 Vercel 提示添加，用于验证域名的所有权。


3. **关键设置（防坑）**：
* **重要**：在 Vercel 完成域名验证和 SSL 证书颁发期间，请将 Cloudflare 记录的代理状态（Proxy status）设置为 **灰色云朵 (DNS Only)**。
* 待 Vercel 显示 "Active" 后，再根据需要开启橙色云朵（CDN 代理）。

### 3. 路径重写配置 (vercel.json)

如果你希望实现“访问根域名（如 `openbit.dpdns.org`）自动跳转或映射到文章列表页（`/posts`）”，请在项目根目录下创建 `vercel.json` 文件：

```json
{
  "rewrites": [
    { "source": "/", "destination": "/posts" },
    { "source": "/posts", "destination": "/posts" },
    { "source": "/((?!posts/).*)", "destination": "/posts/$1" }
  ]
}
```

---

**说明**：完成这一步后，你的博客即可通过正式域名访问。接下来的步骤是回到代码中修改 `src/config.ts` 的站点 URL，以确保 RSS 和 SEO 信息的准确性。

---

## 站点深度优化

### 1. 全局元数据修改

编辑 `src/config.ts` 以应用你的个人信息：

* **SITE.website**: 修改为你的正式域名（如 `https://openbit.dpdns.org/`）。
* **LOCALE**: 将 `main` 设为 `"zh-CN"` 以优化日期显示格式。

### 2. 路径映射策略

在项目根目录创建 `vercel.json`，以优化 URL 跳转逻辑（可选）：

```json
{
  "cleanUrls": true,
  "rewrites": [
    { "source": "/", "destination": "/" }
  ]
}

```

---

## 日常发布工作流

1. **创作**: 在 `src/data/blog/` 下新建 `.md` 文件并填入规范的头部信息。
2. **提交**: 执行 `git commit` 和 `push` 将代码推送至 GitHub。
3. **部署**: Vercel 会自动触发构建。
4. **验证**: 约 1 分钟后，访问你的正式域名查看更新。



