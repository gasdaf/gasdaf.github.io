# 博客搭建 Day-1 TODO（Docker 版，E2E 到 GitHub）

更新时间：2026-03-24

目标：1 天内完成个人博客上线（先 1 篇样例文章），并由 Codex 完成代码提交与推送。

---

## A. 基础准备（阻塞项优先）

- [ ] A1 安装 Docker 与 GitHub CLI（Ubuntu 22.04）
  - 命令：
    ```bash
    sudo apt-get update
    sudo apt-get install -y docker.io gh
    ```
  - 验收：
    ```bash
    docker --version
    gh --version
    ```

- [ ] A2 启动 Docker 服务并验证可用
  - 命令：
    ```bash
    sudo service docker start || sudo systemctl start docker
    sudo docker run --rm hello-world
    ```
  - 验收：`hello-world` 输出成功信息。

- [ ] A3 完成 GitHub CLI 登录
  - 命令：
    ```bash
    gh auth login
    gh auth status
    ```
  - 验收：`gh auth status` 显示已登录 GitHub。

---

## B. 仓库与站点初始化

- [ ] B1 创建 GitHub Pages 仓库（必须命名为 `<username>.github.io`）
  - 命令（可选，若网页已创建可跳过）：
    ```bash
    gh repo create <username>.github.io --public --confirm
    ```
  - 验收：仓库地址可访问：`https://github.com/<username>/<username>.github.io`

- [ ] B2 进入本地工作目录并初始化 Git 仓库
  - 命令（在当前目录执行）：
    ```bash
    git init -b main
    git remote add origin https://github.com/<username>/<username>.github.io.git
    ```
  - 验收：`git remote -v` 显示 `origin`。

- [ ] B3 用 Docker 初始化 Hugo 站点
  - 命令：
    ```bash
    sudo docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest new site . --force
    ```
  - 验收：生成 `hugo.toml`、`content/`、`layouts/` 等目录/文件。

- [ ] B4 添加 PaperMod 主题（submodule）
  - 命令：
    ```bash
    git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
    ```
  - 验收：存在 `themes/PaperMod/` 且 `.gitmodules` 已生成。

---

## C. 内容与配置（遵守可迁移规范）

- [ ] C1 配置 `hugo.toml`
  - 必须满足：
    - `baseURL = "https://<username>.github.io/"`
    - `theme = "PaperMod"`
    - `languageCode = "zh-cn"`
    - 永久链接：`/posts/:slug/`
    - 菜单含 `Posts`、`Tags`
  - 验收：配置无语法错误，后续构建成功。

- [ ] C2 创建样例文章（1 篇）
  - 命令：
    ```bash
    sudo docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest new content posts/hello-blog.md
    ```
  - Front Matter 必填字段：
    - `title`
    - `date`
    - `slug`
    - `tags`
    - `summary`
  - 验收：文章路径为 `content/posts/hello-blog.md`，字段齐全。

- [ ] C3 预留图片路径规范
  - 目录规则：`static/images/YYYY/MM/<slug>/...`
  - 验收：至少创建一个示例目录（可空目录）。

---

## D. 本地验证（Docker 构建与预览）

- [ ] D1 生产构建验证
  - 命令：
    ```bash
    sudo docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest hugo --minify
    ```
  - 验收：返回码为 0，生成 `public/`。

- [ ] D2 本地预览验证
  - 命令：
    ```bash
    sudo docker run --rm -p 1313:1313 -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest server --bind 0.0.0.0 --baseURL http://localhost/ -D
    ```
  - 验收：
    - 浏览器打开 `http://localhost:1313`
    - 文章 URL 为 `/posts/hello-blog/`

---

## E. 自动部署与 E2E 提交

- [ ] E1 新建 GitHub Actions 文件 `.github/workflows/hugo.yml`
  - 要求：
    - `push` 触发 `main`
    - checkout 含 submodules
    - 构建 + upload artifact + deploy pages
  - 验收：YAML 校验通过。

- [ ] E2 提交代码
  - 命令：
    ```bash
    git add .
    git commit -m "init: hugo blog with docker workflow"
    ```
  - 验收：`git log -1 --oneline` 可见提交。

- [ ] E3 推送到 GitHub（Codex 执行）
  - 命令：
    ```bash
    git push -u origin main
    ```
  - 验收：远端分支创建成功。

- [ ] E4 仓库开启 Pages（GitHub Actions 作为构建源）
  - 路径：`Settings -> Pages -> Build and deployment -> GitHub Actions`
  - 验收：Actions 工作流跑绿。

- [ ] E5 线上验收
  - 验收：
    - 首页可访问：`https://<username>.github.io/`
    - 文章页可访问：`https://<username>.github.io/posts/hello-blog/`

---

## F. Day-1 完成定义（Definition of Done）

- [ ] DoD1：`<username>.github.io` 线上可访问。
- [ ] DoD2：至少 1 篇文章发布成功。
- [ ] DoD3：`/posts/<slug>/` 永久链接生效。
- [ ] DoD4：Front Matter 字段规范已落地。
- [ ] DoD5：Codex 已完成一次 E2E 提交与推送记录。

