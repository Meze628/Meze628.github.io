---
title: "在 Gemini 3 Pro 的指导下完成博客的搭建"
description: "一篇详细的技术复盘，记录如何在 Windows 环境下配置 Hugo + Stack 主题，解决图标报错、CSS 样式冲突及 GitHub Actions 部署版本问题。由 Gemini 3 Pro 整理"
date: 2026-02-12T17:00:00+08:00
image: cover.jpg
categories:
    - 记录
tags:
    - Hugo
    - 博客搭建
---

本文记录了我基于 **Hugo (Extended 版本)** 和 **Stack 主题** 搭建个人博客的全过程。几乎都是 Gemini 3 Pro 帮我完成的，包括这篇博客。

## 1. 环境

我的系统环境如下

* **OS**: Windows 10/11 (本文演示环境)
* **Git**: 已安装并配置好 SSH/HTTPS。
* **Hugo**: 必须安装 **Extended** 版本（Stack 主题依赖 SCSS 编译，普通版会报错）。
    * 验证命令：`hugo version`
    * 输出示例：`hugo v0.155.0+extended windows/amd64`

## 2. 初始化与主题安装

### 2.1 创建仓库与站点
在 GitHub 创建仓库 `username.github.io`，并在本地克隆：

```bash
git clone https://github.com/username/username.github.io.git
cd username.github.io
# 强制初始化 Hugo 站点结构
hugo new site . --force --format yaml
```

### 2.2 安装 Stack 主题
推荐使用 Git Submodule 管理主题，便于后续更新：

```bash
git submodule add https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack
```

### 2.3 获取配置文件 (关键步骤)
Stack 主题的配置较为复杂，直接手写容易出错。官方提供了一个 Starter 仓库，但为了保持目录整洁，我们采用“手动迁移”的方式。

1.  下载 [hugo-theme-stack-starter](https://github.com/CaiJimmy/hugo-theme-stack-starter) 的 ZIP 包。
2.  解压后，将以下三个文件夹复制到你的博客根目录，**覆盖**原有文件：
    * `config/` (核心配置)
    * `content/` (演示文章)
    * `assets/` (资源文件)

3.  **修正配置路径**：
    打开 `config/_default/config.toml`，找到 `theme` 字段，确保指定为本地主题路径（Starter 默认使用 Module，我们需要改为 Submodule 模式）：

    ```toml
    # 必须显式指定主题名称
    theme = "hugo-theme-stack"
    baseurl = "[https://你的用户名.github.io/](https://你的用户名.github.io/)"
    ```

## 3. 本地化配置 (解决 TOML 报错)

默认配置为英文，我们需要将其转换为中文环境。

### 3.1 基础参数
修改 `config/_default/config.toml`：
```toml
languageCode = "zh-cn"
defaultContentLanguage = "zh-cn"
hasCJKLanguage = true  # 开启中文字数统计，否则阅读时间计算不准
```

### 3.2 语言文件与菜单
在修改 `config/_default/languages.toml` 时，我遇到了 `unmarshal failed: toml: incomplete number` 报错。这是因为在定义菜单 `weight` 时格式不完整。

**正确的中文配置示例**：

```toml
[zh-cn]
    languageName = "中文"
    title = "我的博客"
    owner = "Meze"

    [zh-cn.params]
        readMore = "阅读全文"
        date_format = "2006-01-02" # Go语言特有的时间格式化

    [zh-cn.menu]
        [[zh-cn.menu.main]]
            identifier = "home"
            name = "首页"
            url = "/"
            weight = 1
            [zh-cn.menu.main.params]
                icon = "home"

        [[zh-cn.menu.social]]
            identifier = "github"
            name = "GitHub"
            url = "[https://github.com/username](https://github.com/username)"
            weight = 1
            [zh-cn.menu.social.params]
                icon = "github"
```

## 4. 图标系统与报错修复

Stack 主题的一大特点是**强依赖本地 SVG 图标**。如果在配置中引用了 `icon = "abc"`，但 `assets/icons/` 目录下没有 `abc.svg`，Hugo 会直接报错停止构建。

### 4.1 解决 "icon not found" 错误
**报错现象**：
`ERROR Error: icon 'github.svg' is not found under 'assets/icons' folder`

**解决方案**：
1.  在博客根目录创建路径：`assets/icons/`。
2.  从 [Tabler Icons](https://tabler.io/icons) 下载所需图标的 SVG 文件。
3.  **重命名**：下载的文件（如 `brand-github.svg`）必须重命名为配置文件中 `icon` 字段对应的值（如 `github.svg`）。

### 4.2 修复夜间模式切换图标重叠
为了个性化，我替换了默认的切换图标。但发现点击切换时，太阳和月亮图标同时显示。
**原因**：主题 CSS 是根据默认类名控制显示隐藏的，自定义图标导致类名变化，样式失效。

**解决方案**：
创建 `assets/scss/custom.scss`，写入以下 CSS 强制覆盖默认逻辑：

```scss
/* 强制修正 Stack 主题左下角切换图标的显示逻辑 */
/* 白天模式：显示第一个图标(日)，隐藏第二个(月) */
:root:not([data-scheme="dark"]) #dark-mode-toggle {
    svg:nth-of-type(1) { display: inline-block !important; }
    svg:nth-of-type(2) { display: none !important; }
}

/* 黑夜模式：隐藏第一个(日)，显示第二个(月) */
:root[data-scheme="dark"] #dark-mode-toggle {
    svg:nth-of-type(1) { display: none !important; }
    svg:nth-of-type(2) { display: inline-block !important; }
}
```

## 5. 自动化部署 (GitHub Actions)

为了实现 `git push` 后自动发布，我们需要配置 GitHub Actions。

### 5.1 创建 Workflow
在根目录创建 `.github/workflows/deploy.yaml`。

### 5.2 解决 Hugo 版本不兼容问题
**报错现象**：
`Deploy` 过程中出现 `function "hash" not defined` 错误，提示 `Module "hugo-theme-stack" is not compatible with this Hugo version`。

**原因**：
GitHub Actions 默认脚本或旧教程通常使用 Hugo `0.128.0`，而最新版 Stack 主题要求 Hugo 版本 `>= 0.154.0`。

**修正后的配置 (deploy.yaml)**：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main  # 确保分支名正确

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
    env:
      HUGO_VERSION: 0.155.0 # 【关键】显式指定新版本
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb [https://github.com/gohugoio/hugo/releases/download/v$](https://github.com/gohugoio/hugo/releases/download/v$){HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      # ... (其余步骤保持标准配置: Install Dart Sass, Checkout, Build, Upload)

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 5.3 开启 GitHub Pages
最后，进入 GitHub 仓库设置 -> **Settings** -> **Pages**。
将 **Source** 更改为 **GitHub Actions**。

## 6. 总结

通过以上步骤，你现在应该已经拥有了一个在本地和云端都能完美运行的技术博客。感谢 Gemini 3 Pro！