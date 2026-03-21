---
layout: post
title: "服务器运维手册"
date: 2026-03-21 16:00:00 +0000
categories: [技术, 运维]
tags: [openclaw, 运维手册, systemd, 故障排查, 内存管理]
---

## 📝 MEMORY.md | 运维技术栈：OpenClaw Root 版

### 🏗 环境架构

- **运行用户**：`root`
- **服务路径**：`/etc/systemd/system/openclaw-gateway.service`
- **执行文件**：`/usr/bin/openclaw`
- **配置目录**：`/root/.openclaw/`
- **工作空间**：`/root/` (存储 Weekly Review 等 Markdown 文件)

---

### 🛠 核心管理指令 (已加入 `.bashrc` 别名)

| 任务 | 原始命令 | 别名 (Alias) |
| :--- | :--- | :--- |
| **查看实时状态** | `systemctl status openclaw-gateway` | `claw` |
| **重启服务** | `systemctl restart openclaw-gateway` | `reclaw` |
| **查看滚动日志** | `journalctl -u openclaw-gateway -f -n 50` | `logs` |
| **停止服务** | `systemctl stop openclaw-gateway` | - |

---

### 🚨 常见故障排查 (Troubleshooting)

#### 1. 机器人"已读不回" (死锁排查)

如果 status 显示 active 但 Telegram 没反应，执行：

```bash
# 强制杀掉残留的 Node 僵尸进程并重启
pkill -9 openclaw && systemctl restart openclaw-gateway
```

#### 2. 内存溢出 (OOM) 导致服务重启

如果你在日志里看到 `JavaScript heap out of memory`：

- **现状**：当前已在 `.service` 中限制了 `800M` 内存。
- **对策**：如果频繁崩溃，修改 `/etc/systemd/system/openclaw-gateway.service`，将 `MemoryLimit` 提升至 `1G`，然后执行 `systemctl daemon-reload && reclaw`。

#### 3. 配置文件修改未生效

OpenClaw 启动时会锁定配置。修改 `openclaw.json` 后，**必须**执行 `reclaw` 才能加载新的 `128k ctx` 或 API Key。

---

### 📅 定期维护建议

- **清理日志**：`journalctl --vacuum-time=7d` (仅保留最近 7 天日志，防止撑爆磁盘)。
- **备份记忆**：定期手动备份 `/root/.openclaw/workspace/` 下的重要 Markdown 文件。

---

**更新完毕！** 既然服务器已经成了"正规军"，你可以尝试给 Telegram 机器人发一句：

> "新一，帮我把刚才这段运维指令整理成一个 SERVER_GUIDE.md 存在你的 workspace 里。"

看看它现在的处理速度是不是比之前那次"打架"的时候快多了？
