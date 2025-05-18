# Leila 个人博客

这是一个基于 [Hexo](https://hexo.io/) 框架的个人博客项目，使用 [Async](https://github.com/MaLuns/hexo-theme-async) 主题。

## 环境要求

- Node.js (建议 v16.0.0 或更高版本)
- Git

## 项目安装与启动

### 克隆项目

```bash
git clone https://github.com/yourusername/leila-blog.git
cd leila-blog
```

### 安装依赖

```bash
npm install
# 或者使用 yarn
yarn install
```

### 本地启动

```bash
npm run server
# 或者使用 yarn
yarn server
```

启动后，访问 `http://localhost:4000` 即可查看博客。

## 内容管理

### 创建新文章

```bash
hexo new "文章标题"
```

这将在 `source/_posts` 目录下创建一个新的 Markdown 文件。

### 创建草稿

```bash
hexo new draft "草稿标题"
```

草稿保存在 `source/_drafts` 目录下，默认不会显示在网站上。

### 将草稿转为正式文章

```bash
hexo publish "草稿标题"
```

### 创建新页面

```bash
hexo new page "页面名称"
```

这将在 `source` 目录下创建一个新的文件夹和 index.md 文件。

## 文章格式

文章使用 Markdown 格式编写，需要在文件开头添加 Front-matter：

```markdown
---
title: 文章标题
date: 2023-06-01 14:30:00
updated: 2023-06-02 12:00:00
tags:
  - 标签1
  - 标签2
categories:
  - 分类1
  - 分类2
description: 文章描述，会显示在首页
cover: /path/to/cover/image.jpg # 文章封面图
---

这里是文章内容...
```

### 常用 Markdown 语法

```markdown
# 一级标题

## 二级标题

**粗体文本**
_斜体文本_

> 引用文本

- 无序列表项
- 无序列表项

1. 有序列表项
2. 有序列表项

[链接文本](https://example.com)

![图片描述](/path/to/image.jpg)

\`\`\`javascript
// 代码块
console.log('Hello World');
\`\`\`
```

## 构建与部署

### 生成静态文件

```bash
hexo clean && hexo generate
# 或者使用简写
hexo clean && hexo g
```

### 部署到 GitHub Pages

1. 修改 `_config.yml` 文件中的 deploy 部分：

```yaml
deploy:
  type: git
  repo: https://github.com/yourusername/yourusername.github.io.git
  branch: main
```

2. 安装部署插件：

```bash
npm install hexo-deployer-git --save
```

3. 执行部署命令：

```bash
hexo deploy
# 或者使用简写
hexo d
```

也可以合并生成和部署命令：

```bash
hexo clean && hexo generate && hexo deploy
# 或者使用简写
hexo clean && hexo g && hexo d
```

## 主题配置

本博客使用 Async 主题，主题配置文件为 `_config.async.yml`。

### 修改个人信息

```yaml
user:
  name: 你的名字
  first_name: 名
  last_name: 姓
  email: your-email@example.com
  domain: https://yourdomain.com/
  avatar: /img/avatar.jpg # 头像路径
  describe: 个人简介
```

### 修改首页横幅

```yaml
banner:
  default:
    type: img
    bgurl: https://example.com/your-banner-image.jpg
    banner_title: 横幅标题
    banner_text: 横幅文字
```

### 添加社交媒体链接

```yaml
sidebar:
  social:
    github: https://github.com/yourusername
    twitter: https://twitter.com/yourusername
    # 更多社交媒体...
```

## 常见问题

如果遇到问题，可以参考 [Hexo 文档](https://hexo.io/zh-cn/docs/troubleshooting) 或者在 [GitHub Issues](https://github.com/hexojs/hexo/issues) 上提问。

## 参考资源

- [Hexo 官方文档](https://hexo.io/zh-cn/docs/)
- [Async 主题文档](https://hexo-theme-async.imalun.com/)
- [Markdown 基本语法](https://www.markdownguide.org/basic-syntax/)
