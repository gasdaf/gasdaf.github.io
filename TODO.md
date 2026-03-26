# 博客搭建 Day-1 TODO（Docker 版，E2E 到 GitHub）

更新时间：2026-03-26

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
    sudo docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest --minify
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

## E. 自动部署与 E2E 提交（详细版）

- [ ] E0 预检查（必须先过）
  - 命令：
    ```bash
    cd /mnt/d/git/practice_blog
    git rev-parse --is-inside-work-tree
    git branch --show-current
    gh auth status
    git remote -v
    ```
  - 验收：
    - 在 Git 仓库内
    - 当前分支为 `main`
    - `gh` 已登录
    - `origin` 指向 `git@github.com:<username>/<username>.github.io.git`

- [ ] E1 新建 GitHub Actions 文件 `.github/workflows/hugo.yml`
  - 命令：
    ```bash
    mkdir -p .github/workflows
    cat > .github/workflows/hugo.yml <<'EOF'
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
      group: pages
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
              hugo-version: latest
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
    EOF
    ```
  - 验收：文件存在且 YAML 无语法错误。

- [ ] E2 本地构建自检（建议）
  - 命令：
    ```bash
    sudo docker run --rm -v "$PWD":/src -w /src ghcr.io/gohugoio/hugo:latest --minify
    ```
  - 验收：命令成功并生成 `public/`。

- [ ] E3 提交前检查
  - 命令：
    ```bash
    git status --short
    ```
  - 验收：至少包含 `hugo.toml`、`content/`、`.github/workflows/hugo.yml`、`.gitmodules`、`themes/PaperMod`。

- [ ] E4 提交代码
  - 命令：
    ```bash
    git add .
    git commit -m "init: hugo pages site with actions deploy"
    ```
  - 验收：`git log -1 --oneline` 可见提交记录。

- [ ] E5 推送到 GitHub（Codex 执行）
  - 命令：
    ```bash
    git push -u origin main
    ```
  - 验收：远端 `main` 分支创建/更新成功。

- [ ] E6 若 Pages 模式未自动启用，强制设为 workflow
  - 命令：
    ```bash
    gh api repos/<username>/<username>.github.io/pages || true
    gh api -X POST repos/<username>/<username>.github.io/pages -f build_type=workflow || \
    gh api -X PUT repos/<username>/<username>.github.io/pages -f build_type=workflow
    ```
  - 验收：Pages API 可返回配置，`build_type` 为 `workflow`。

- [ ] E7 观察 CI/CD 运行状态
  - 命令：
    ```bash
    gh run list --limit 5
    gh run watch
    ```
  - 验收：`build` 和 `deploy` Job 均为绿色成功。

- [ ] E8 线上验收
  - 验收：
    - 首页可访问：`https://<username>.github.io/`
    - 文章页可访问：`https://<username>.github.io/posts/hello-blog/`
    - `Settings -> Pages` 显示站点已发布状态

---

## F. Day-1 完成定义（Definition of Done）

- [ ] DoD1：`<username>.github.io` 线上可访问。
- [ ] DoD2：至少 1 篇文章发布成功。
- [ ] DoD3：`/posts/<slug>/` 永久链接生效。
- [ ] DoD4：Front Matter 字段规范已落地。
- [ ] DoD5：Codex 已完成一次 E2E 提交与推送记录。
