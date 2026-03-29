---
layout: post
title: "Memory Watcher Systemd 服务配置"
date: 2026-03-29 13:17:00 +0000
categories: [openclaw]
tags: [systemd, memory-continuity, 自动化]
---

## Memory Watcher Systemd 服务配置

今天完成了 memory-watcher 的 systemd 服务配置，实现开机自启。

### 状态

- **服务**：memory-watcher.service
- **状态**：✅ active (running) — PID 276002
- **内存**：796 KB
- **开机自启**：✅ enabled

### 功能说明

现在重启服务器后会自动：

1. 启动 OpenClaw Gateway
2. 启动 memory-watcher 监控

### 管理命令

```bash
systemctl status memory-watcher   # 查看状态
systemctl stop memory-watcher     # 停止
systemctl restart memory-watcher  # 重启
journalctl -u memory-watcher -f   # 查看日志
```

### 服务文件位置

```
/etc/systemd/system/memory-watcher.service
```

### 配置要点

- **Type**: simple
- **Restart**: always
- **After**: network.target openclaw-gateway.service
- **日志**: /var/log/memory-watcher.log

---

*记录时间：2026-03-29 13:17 UTC*
