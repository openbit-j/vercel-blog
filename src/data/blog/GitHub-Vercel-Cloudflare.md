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


> AstroPaper 博客在 Vercel 上的部署，**最合适、最干净、长期维护最容易的方法**：直接使用 `create astro` 命令以模板方式创建项目。

#### AstroPaper + Vercel 部署轻量博客（全站深度中文化指南）

目标：使用 AstroPaper 模板在 Vercel 上快速部署一个**极简、高性能、SEO 友好**的个人博客，支持中文优先，并绑定自定义域名。

**当前 AstroPaper 版本**：v5.5.1（2026年1月最新）

#### 第一阶段：基础部署（本地 → GitHub → Vercel）

1. **创建项目（强烈推荐 pnpm）**  
   AstroPaper 官方首推 pnpm（更快、省空间）。如果未安装 pnpm，可用 `npm install -g pnpm` 安装。

   ```bash
   # 首选（推荐）
   pnpm create astro@latest -- --template satnaing/astro-paper

   # 备选（npm）
   npm create astro@latest -- --template satnaing/astro-paper
   ```

   - 项目名建议：`my-blog` 或 `openbit-blog`
   - 选择 TypeScript（强烈推荐）
   - 完成后自动进入项目目录

2. **安装依赖 & 本地预览**

   ```bash
   cd my-blog          # 替换为你起的项目名
   pnpm install
   pnpm run dev
   ```

   浏览器访问 http://localhost:4321，确认看到 AstroPaper 默认首页。

3. **初始化 Git 并推送到 GitHub**

   先在 GitHub 创建**空仓库**（不要勾选 README、.gitignore、license）。

   ```bash
   git init
   git add .
   git commit -m "Initial commit: AstroPaper v5.5.1 setup"

   git remote add origin https://github.com/你的用户名/my-blog.git
   git branch -M main
   git push -u origin main
   ```

4. **Vercel 一键部署（零配置）**

   1. 登录 https://vercel.com/dashboard
   2. 点击 “Add New...” → Project
   3. Import Git Repository → 选你的 GitHub 仓库
   4. **全部保持默认**（Vercel 自动识别 Astro）：
      - Framework Preset: Astro
      - Build Command: 留空（默认 `astro build`）
      - Output Directory: 留空（默认 `dist`）
      - Install Command: 留空（pnpm install 会自动识别）
   5. 点击 Deploy（30–90 秒完成）

   部署成功后获得临时域名：`https://my-blog-你的用户名.vercel.app`

5. **写 & 发布第一篇文章（自动部署）**

   - 文章目录：`src/data/blog/`
   - 新建文件例如 `first-post.md`（或 .mdx）

     ```markdown
     ---
     author: 你的名字
     pubDatetime: 2026-01-19T21:00:00Z
     title: 我的第一篇中文博客
     featured: true
     draft: false
     tags:
       - Astro
       - Vercel
       - 中文博客
     description: 使用 AstroPaper + Vercel 快速搭建高性能个人博客的全流程。
     ---

     正文内容，支持 Markdown、MDX、代码高亮等。
     ```

   - 保存 → `git add .` → `git commit -m "Add first post"` → `git push`
   - Vercel 自动构建（约 1 分钟后生效）

#### 第二阶段：自定义域名 & 中文化深度优化（Cloudflare + Vercel）

1. **获取免费域名**（推荐  [DigitalPlat](https://domain.digitalplat.org/) 选择 .dpdns.org 免费子域）

   申请后立即接管到 [Cloudflare](https://dash.cloudflare.com/)（速度快、中国访问优化好）：
   - 登录 Cloudflare → Add a Site → 输入你的域名
   - 复制 Cloudflare 提供的两个 NS 记录
   - 回到域名注册商后台，替换 Name Servers 为 Cloudflare 的
   - 等待生效（几分钟到几小时，Cloudflare 显示 Active 即可）

2. **Vercel 绑定域名**

   - Vercel 项目 → Settings → Domains → 输入你的域名（e.g. openbit.dpdns.org）→ Add
   - Vercel 会提示 DNS 记录：

| **类型 (Type)** | **名称 (Name)** | **内容 (Content / Value)** | **代理状态 (Proxy Status)** | **备注**       |
| ------------- | ------------- | ------------------------ | ----------------------- | ------------ |
| **A**         | `@`           | `76.76.21.21`            | **DNS Only (灰色云朵)**     | 指向 Vercel 节点 |
| **TXT**       | `_vercel`     | `vc-domain-verify=XXXXX` | **DNS Only**                 | 域名所有权验证      |



3. **Cloudflare DNS 配置防坑**

   - 添加上述 A/TXT 记录
   - **关键**：初始验证阶段把 Proxy status 设为 **灰云（DNS Only）**（关闭橙云代理）
   - Vercel 显示 “Valid Configuration” 和 SSL “Active” 后，再考虑开启橙云（开启后可加速，但有时会干扰 Vercel 的边缘路由）

4. **站点配置中文化 & SEO 优化**

   编辑 `src/config.ts`：

   ```ts
   export const SITE = {
     website: 'https://openbit.dpdns.org/',  // 改成你的正式域名（注意末尾 /）
     // ... 其他保持默认
   };

   export const LOCALE = {
     lang: 'zh-CN',          // 中文优先
     // ... 
   };
   ```

   - 保存 → commit & push → Vercel 自动更新 RSS、sitemap、meta 等

5. **根路径优化（可选，推荐）**

   如果想访问根域名（/）直接显示文章列表（而非 Astro 默认首页），在项目根目录创建/修改 `vercel.json`：

   ```json
   {
     "cleanUrls": true,
     "rewrites": [
       { "source": "/", "destination": "/posts" }
     ]
   }
   ```

   - commit & push 后生效
   - 更简单方案：直接修改 `src/pages/index.astro` 为重定向或内容导入

#### 日常工作流（极简）

1. 在 `src/data/blog/` 新建/编辑 .md 文件
2. `git add . && git commit -m "Update post" && git push`
3. 等待 Vercel 构建（通常 < 1 分钟）
4. 访问你的域名查看更新

#### 常见问题速查

- **pnpm install 报错** → 确保全局安装 pnpm，或改用 npm install
- **域名不生效** → 检查 Cloudflare Proxy（必须先灰云验证）
- **图片加载慢** → 后期可加 `@astrojs/image` 或 Cloudflare Polish
- **想加评论** → 推荐 Giscus（GitHub Discussions 驱动，最简单）
- **想加搜索** → 加 Pagefind（全静态，pnpm astro add pagefind）


#### 相关项目链接（供参考）

- **AstroPaper 官方 GitHub 仓库**（源码、最新版本、README 文档）：  
  https://github.com/satnaing/astro-paper

- **AstroPaper 官方在线演示**（实时预览主题效果）：  
  https://astro-paper.pages.dev/

- **Astro 官方主题页面 - AstroPaper 详情**（包含特性列表、快速启动）：  
  https://astro.build/themes/details/astropaper

- **Astro 官方文档**（部署、配置、MDX 等进阶知识）：  
  https://docs.astro.build/

- **Vercel 官方 Astro 部署指南**（如果遇到构建问题可参考）：  
  https://vercel.com/docs/frameworks/astro



