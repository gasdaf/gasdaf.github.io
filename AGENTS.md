# 个人博客方案（可扩展 + 可迁移）

更新时间：2026-03-19

## 0) 执行约定

- 本文档用于沉淀后续对话中的可执行信息，便于你快速检索。
- 后续每次给出有用结论时，我都会同步更新本文件。

---

## 1) 严厉结论（先统一认知）

- 你可以用 Codex + GitHub Pages 在几乎不手写代码的前提下上线个人博客。
- 但 GitHub Pages 是静态托管，只适合展示型博客，不适合复杂产品能力（会员、数据库驱动评论、后台系统等）。
- 若目标是“先上线并持续写作”，该方案正确；若目标是“复杂产品”，需后续升级架构。

---

## 2) 你的专属路线（基于 Python / Docker / FastAPI 基础）

### 阶段1（立即执行，1~3天）

- 技术：`GitHub Pages + Hugo + Markdown`
- 目标：快速上线、稳定发布、低维护成本
- 核心产出：可访问博客网址、至少3篇文章、自动发布流程

### 阶段2（3~8周）

- 增加轻动态能力（评论、订阅、统计），但不自建后端。
- 保持“内容优先”，不要过早建设复杂系统。

### 阶段3（后续扩展）

- 迁移到 `FastAPI + PostgreSQL + Docker`。
- 采用并行迁移与切流，保留原 URL 结构，避免 SEO 损失。

---

## 3) 为什么不是一开始就 FastAPI 动态站

- 早期瓶颈是内容生产，不是架构能力。
- 动态系统会显著增加维护成本与运维复杂度。
- 静态站在性能、安全、成本上更稳，更适合个人博客冷启动。

---

## 4) 固定技术选型（当前推荐）

- 博客引擎：`Hugo`
- 托管：`GitHub Pages`
- 内容：`Markdown + Front Matter`
- 发布：`GitHub Actions` 自动构建与部署
- 资产组织：
  - 文章：`content/`
  - 图片与静态资源：`static/`

---

## 5) 可迁移设计规范（从第一天就执行）

- 永久链接固定：`/posts/<slug>/`
- 每篇文章最少字段：
  - `title`
  - `date`
  - `slug`
  - `tags`
  - `summary`
- 图片路径规范：`/images/YYYY/MM/<slug>/...`
- 避免把业务逻辑写死在主题层（降低换主题与迁移成本）。
- 内容与展示分离思维：优先保证 Markdown 资产可复用。

---

## 6) 未来迁移目标架构（草图）

- `fastapi`：对外 API 与管理接口
- `postgres`：文章、标签、草稿、订阅数据
- `docker compose`：本地一键运行
- 迁移策略：新旧系统并行 -> 验证 -> 切流 -> 301 重定向兜底

---

## 7) 执行纪律（强约束）

- 前3个月不做用户系统，不做自建后台，不做重交互。
- 优先持续产出：每周至少2篇文章。
- 若连续2个月无法稳定更新，暂停架构升级，回到内容节奏。

---

## 8) 现在就做的4件事

1. 创建仓库：`<yourname>.github.io` 并初始化 Hugo。
2. 配置 GitHub Actions 自动发布到 Pages。
3. 先完成并发布3篇文章，再进行外观微调。
4. 第4周再决定是否接入评论/订阅能力。

---

## 9) 后续更新记录

- 2026-03-19：建立初版路线图与规范基线（本次）。

---

## 10) 阶段1可执行SOP（零手写代码优先）

说明：以下步骤按顺序执行；所有 `<...>` 都要替换成你的真实值。

### Step 1. 前置检查（本地）

```bash
git --version
docker --version
```

通过标准：两条命令都返回版本号。

### Step 2. 创建 GitHub 仓库（网页操作）

1. 在 GitHub 新建仓库，仓库名必须是：`<你的GitHub用户名>.github.io`
2. 设为 Public。
3. 勾选 `Add a README file`（可选，但建议勾选）。

通过标准：仓库地址可访问，形如 `https://github.com/<username>/<username>.github.io`

### Step 3. 克隆并初始化 Hugo 站点（Docker方式）

```bash
export BLOG_USER="<your_github_username>"
export BLOG_REPO="${BLOG_USER}.github.io"
cd /mnt/d/dailywork
git clone "https://github.com/${BLOG_USER}/${BLOG_REPO}.git"
cd "${BLOG_REPO}"
docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest new site . --force
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

通过标准：目录下出现 `hugo.toml`、`content/`、`themes/PaperMod/`

### Step 4. 写入站点配置（`hugo.toml`）

将 `hugo.toml` 覆盖为：

```toml
baseURL = "https://<your_github_username>.github.io/"
languageCode = "zh-cn"
title = "<你的博客名>"
theme = "PaperMod"

[params]
  ShowReadingTime = true
  ShowPostNavLinks = true
  ShowCodeCopyButtons = true

[permalinks]
  posts = "/posts/:slug/"

[menu]
  [[menu.main]]
    identifier = "posts"
    name = "Posts"
    url = "/posts/"
    weight = 10
  [[menu.main]]
    identifier = "tags"
    name = "Tags"
    url = "/tags/"
    weight = 20
```

### Step 5. 新建首篇文章

```bash
docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest new content posts/hello-blog.md
```

将 `content/posts/hello-blog.md` 改为（示例）：

```markdown
+++
title = "Hello Blog"
date = 2026-03-19T20:00:00+08:00
slug = "hello-blog"
tags = ["start", "intro"]
summary = "我的第一篇博客文章"
+++

这是我的第一篇文章。
```

### Step 6. 本地预览

```bash
docker run --rm -p 1313:1313 -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest server --bind 0.0.0.0 --baseURL http://localhost/ -D
```

浏览器打开：`http://localhost:1313`

### Step 7. 配置 GitHub Actions 自动发布

创建文件：`.github/workflows/hugo.yml`

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Step 8. 提交并触发部署

```bash
git add .
git commit -m "init: hugo blog with pages deploy"
git push origin main
```

然后到 GitHub 仓库设置：

1. `Settings` -> `Pages`
2. `Build and deployment` 选择 `GitHub Actions`

通过标准：

- `Actions` 页面里 workflow 绿色通过。
- 博客可访问：`https://<your_github_username>.github.io/`

---

## 11) 内容与迁移字段规范（必须执行）

每篇文章都必须包含：

- `title`
- `date`
- `slug`
- `tags`
- `summary`

图片路径固定：

- `static/images/YYYY/MM/<slug>/xxx.png`

这样未来迁移到 FastAPI 时可以直接脚本化导入，不改历史链接。

---

## 12) 后续更新记录

- 2026-03-19：补充阶段1完整SOP（含命令、配置文件、验收标准）。
