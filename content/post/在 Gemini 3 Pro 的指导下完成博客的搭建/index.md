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
* **Git**: 已安装并配置好了代理。
    ```bash
    git config http.proxy http://127.0.0.1:7893
    ```
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
原先黑白主题切换图标使用的勾选条，我替换了太阳和月亮。在 [Tabler Icons](https://tabler.io/icons) 搜索即可。并将它们改名为与原图标名字一样的名字，分别是 toggle-left 和 toggle-right。    
在 `assets/scss/custom.scss` 文件中加入以下内容
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

## 5. 美化代码块（Code Block）
在 `assets/scss/custom.scss` 文件中加入以下内容   
```scss
/* 2. 代码块外层容器 */
.article-content .highlight {
    /* 强制使用 Stack 主题的背景色 (覆盖 Dracula 的紫色) */
    background-color: var(--pre-background-color) !important;
    
    /* 容器样式 */
    border-radius: 12px;
    box-shadow: var(--shadow-l1) !important;
    margin: 20px auto !important;
    width: 96% !important;
    
    /* 关键：为顶部的红绿灯留出 32px 的空间 */
    padding-top: 32px !important;
    position: relative;
    overflow: hidden; /* 防止圆角溢出 */
}

/* 3. macOS 红绿灯头部 (绝对定位) */
.article-content .highlight::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 32px;
    
    /* 图标与底部分割线 */
    background: url(/code-header.svg) no-repeat 12px 10px;
    background-size: 46px;
    background-color: inherit; /* 跟随父元素背景 */
    border-bottom: 1px solid var(--body-background);
    z-index: 10;
}

/* 4. 代码滚动区域 (PRE 标签) */
.article-content .highlight pre {
    /* 核心：再次强制背景透明，确保万无一失 */
    background-color: transparent !important;
    background: transparent !important;
    
    /* 核心：开启滚动条 */
    overflow-x: auto !important;
    
    margin: 0 !important;
    padding: 15px 15px !important;
    
    /* 字体与排版 */
    font-family: 'JetBrains Mono', 'Fira Code', Consolas, monospace;
    line-height: 1.5;
}

/* 5. 滚动条美化 */
.article-content .highlight pre::-webkit-scrollbar {
    height: 8px;
}
.article-content .highlight pre::-webkit-scrollbar-thumb {
    background-color: rgba(136, 136, 136, 0.3);
    border-radius: 4px;
}
.article-content .highlight pre::-webkit-scrollbar-thumb:hover {
    background-color: rgba(136, 136, 136, 0.6);
}
```
同时在 `config/_default/markup.toml` 文件后加入如下内容
```toml
[highlight]
    codeFences = true
    guessSyntax = true
    lineNoStart = 1
    
    # 【核心修改】关闭行号，简化结构，确保滚动条百分百生效
    lineNos = false
    lineNumbersInTable = false
    
    # 保持 Dracula 配色
    noClasses = true
    style = "dracula"
    
    tabWidth = 4
```

## 6. 自动化部署 (GitHub Actions)

为了实现 `git push` 后自动发布，我们需要配置 GitHub Actions。

### 6.1 创建 Workflow
在根目录创建 `.github/workflows/deploy.yaml`。

### 6.2 解决 Hugo 版本不兼容问题
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

### 6.3 开启 GitHub Pages
最后，进入 GitHub 仓库设置 -> **Settings** -> **Pages**。
将 **Source** 更改为 **GitHub Actions**。

## 7. 总结

通过以上步骤，你现在应该已经拥有了一个在本地和云端都能完美运行的技术博客。感谢 Gemini 3 Pro！