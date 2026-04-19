Collection of agent skills.

## Intro

### 1. ppt-agent-workflow-san

![GitHub last commit](https://img.shields.io/github/last-commit/mucsbr/ppt-agent-workflow-san?path=SKILL.md&label=ppt-agent-workflow-san)

渐进交互式 ppt 生成 skill

### 2. ppt-agent-workflow-san

![GitHub last commit](https://img.shields.io/github/last-commit/opendatalab/MinerU-Ecosystem?path=skills%2FSKILL.md&label=MinerU)

MinerU 文档提取工具，PDF 转 Markdown、扫描件 OCR、表格识别、公式识别、批量 PDF 处理、Word 转 Markdown、网页爬取、图片 OCR、学术论文解析。MinerU 支持 PDF、Word、PPT、图片等多格式文档智能转换，命令行一键提取，免登录快速模式或高精度专业模式。

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