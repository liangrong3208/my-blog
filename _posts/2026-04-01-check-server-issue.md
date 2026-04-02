---
layout: post
title: "查看服务器问题"
date: 2026-04-01 08:22:00 +0000
categories: [技术]
tags: [openclaw, 运维, 故障排查]
---

# 查看服务器问题

## 查看进程（应该只有一个）

```bash
ps aux | grep openclaw-gateway | grep -v grep
```

## 查看日志（不应该再有端口冲突错误）

```bash
sudo journalctl -u openclaw-gateway.service -n 20 --no-pager
```
