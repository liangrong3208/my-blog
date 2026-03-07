---
layout: post
title: "我重新配置好博客，可以用 OpenClaw 推送了"
date: 2026-03-07 22:00:00 +0800
categories: 随笔
---

今天完成了一件小事，但对我来说挺有意义的——**我的博客现在可以通过 OpenClaw 自动推送更新了**。

## 发生了什么

之前写博客的流程有点繁琐：
1. 本地写好 Markdown
2. `git add .`
3. `git commit -m "xxx"`
4. `git push origin main`

虽然不算复杂，但总是打断写作的思路。现在好了，我只需要告诉 OpenClaw "写一篇博客，内容是..."，它就会帮我完成剩下的工作。

## 怎么做到的

核心就两步：

**1. 配置 GitHub Token**
- 生成 Personal Access Token
- 配置 Git credential helper 存储凭证
- 设置 `GITHUB_TOKEN` 环境变量

**2. 让 OpenClaw 接管**
- 创建工作流，自动生成文章文件
- 提交并推送到 GitHub
- Jekyll 自动部署更新

## 第一篇自动推送的文章

没错，你现在看到的这篇，就是完全通过 OpenClaw 自动推送的。从生成文件到 `git push`，全程零手动操作。

## 后续计划

- 支持定时发布文章
- 自动配图和格式化
- 集成更多发布渠道

技术应该让创作更简单，而不是更复杂。
