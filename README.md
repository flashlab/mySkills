Collection of agent skills.

## Intro

### 1. ppt-agent-workflow-san

![GitHub last commit](https://img.shields.io/github/last-commit/mucsbr/ppt-agent-workflow-san?path=SKILL.md&label=ppt-agent-workflow-san)

渐进交互式 ppt 生成 skill

### 2. mineru-document-extractor

![GitHub last commit](https://img.shields.io/github/last-commit/opendatalab/MinerU-Ecosystem?path=skills%2FSKILL.md&label=MinerU)

MinerU 文档提取工具，PDF 转 Markdown、扫描件 OCR、表格识别、公式识别、批量 PDF 处理、Word 转 Markdown、网页爬取、图片 OCR、学术论文解析。MinerU 支持 PDF、Word、PPT、图片等多格式文档智能转换，命令行一键提取，免登录快速模式或高精度专业模式。

### 3. humanizer-zh

![GitHub last commit](https://img.shields.io/github/last-commit/op7418/Humanizer-zh?path=SKILL.md&label=humanizer-zh)

Humanizer-zh 是一个用于去除文本中 AI 生成痕迹的工具，帮助你将 AI 生成的内容改写得更自然、更像人类书写的文本。

### 4. web-access

![GitHub last commit](https://img.shields.io/github/last-commit/sevico/web-access/edge-browser-support?path=SKILL.md&label=web-access)

功能包括：搜索、网页抓取、登录后操作、网络交互等。触发场景：用户要求搜索信息、查看网页内容、访问需要登录的网站、操作网页界面、抓取社交媒体内容（小红书、微博、推特等）、读取动态渲染页面、以及任何需要真实浏览器环境的网络任务。

## Manage skills by Git

### 拉取根目录

``` bash
git remote add <name> <url>
git subtree add --prefix=skills/<name> <name> <branch> --squash
## pull
git subtree pull --prefix=skills/<name> <name> <branch> --squash

```

### 拉取子目录

``` bash
git remote add <name> <url>
git fetch <name>
git subtree split --prefix=<remote-subdir> <name>/<branch> -b _split/<name>
git subtree add --prefix=skills/<name> _split/<name> --squash
git branch -D _split/<name>
## pull
git fetch <name>
git subtree split --prefix=<remote-subdir> <name>/<branch> -b _split/<name>
git subtree merge --prefix=skills/<name> _split/<name> --squash
git branch -D _split/<name>
```

### 删除子目录

``` bash
git rm -r skills/<name>
```