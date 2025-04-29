# MkDocs 使用说明

MkDocs 是一个快速、简单、完全静态的网站生成器，专门用于构建项目文档。本文档介绍如何使用 MkDocs 构建和部署文档网站。

## 基本命令

### 本地开发
```bash
mkdocs serve
```
启动本地开发服务器，默认在 `http://127.0.0.1:8000`。文件修改会自动重新加载。

### 构建静态网站
```bash
mkdocs build
```
将文档构建为静态网站，输出到 `site/` 目录。

### 部署到 GitHub Pages
```bash
mkdocs gh-deploy
```
将构建好的网站部署到 GitHub Pages。需要预先配置 `mkdocs.yml` 中的 `site_url` 和 `repo_url`。

## 项目结构

- `mkdocs.yml` - 配置文件
- `docs/` - 文档源文件目录
  - `index.md` - 网站首页
  - 其他 Markdown 文件 - 网站内容

## 配置说明

编辑 `mkdocs.yml` 文件可以配置：
- 网站名称和描述
- 主题设置
- 导航菜单
- 插件配置

## 部署指南

1. 确保 `mkdocs.yml` 中配置了正确的 `repo_url`
2. 运行 `mkdocs gh-deploy` 命令
3. 访问 `https://[username].github.io/[repository]` 查看部署结果

## 更多资源

- [MkDocs 官方文档](https://www.mkdocs.org/)
- [Material for MkDocs 主题](https://squidfunk.github.io/mkdocs-material/)