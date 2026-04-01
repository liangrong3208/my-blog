---
layout: post
title: "OpenClaw 服务重启指南"
date: 2026-04-01 17:42:00 +0000
categories: [技术]
tags: [openclaw, systemd, 运维]
---

# OpenClaw 服务重启指南

完整的服务重启流程，确保彻底清理并重新启动。

## 停止服务

```bash
# 停止 systemd 服务
systemctl stop openclaw-gateway.service

# 停止用户级服务（如果存在）
systemctl --user stop openclaw-gateway.service 2>/dev/null
```

## 清理进程

```bash
# 杀死所有相关进程
pkill -9 -f openclaw-gateway
pkill -9 -f "node.*openclaw"

# 确认清理完成
ps aux | grep openclaw | grep -v grep
```

## 重新启动

```bash
# 重新加载 systemd
systemctl daemon-reload

# 启动服务
systemctl start openclaw-gateway.service
```

## 查看日志

```bash
# 查看实时日志（观察是否有 SIGTERM）
journalctl -u openclaw-gateway.service -f
```

---

*最后更新：2026-04-01*
