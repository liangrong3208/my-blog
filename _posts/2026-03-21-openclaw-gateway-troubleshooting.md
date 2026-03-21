---
layout: post
title: "OpenClaw Gateway 故障排查指南"
date: 2026-03-21 13:36:00 +0000
categories: [技术, 运维]
tags: [openclaw, gateway, systemd, 故障排查]
---

## 问题分析

从日志和 `openclaw status` 输出来看，核心问题是 **OpenClaw Gateway 服务虽然被 systemd 管理，但当前处于停止状态（inactive）**，且之前的启动尝试因"端口占用"或"残留进程"而失败。

---

## 逐步解决方案

### 1. 彻底清理残留进程

日志提示 `Gateway already running locally`，说明旧进程未完全退出。

```bash
pkill -f openclaw
pkill -f gateway
```

### 2. 确认服务文件位置

`systemctl status openclaw-gateway.service` 提示找不到 Unit，但 journalctl 却有记录。说明服务可能安装在 **用户级别（`--user`）**，而非系统级别。

查看用户级服务状态：

```bash
systemctl --user status openclaw-gateway.service
```

### 3. 正确启动服务

**步骤 A**：先用 OpenClaw 自带命令停止：

```bash
openclaw gateway stop
```

**步骤 B**：启动服务

- 如果是系统级服务：
  ```bash
  systemctl start openclaw-gateway.service
  ```

- 如果是用户级服务（日志曾提示）：
  ```bash
  systemctl --user start openclaw-gateway.service
  ```

### 4. 检查端口占用

若启动仍失败，检查 18789 端口是否被占用：

```bash
netstat -tunlp | grep 18789
# 或
ss -tunlp | grep 18789
```

如有输出，记下 PID，确认是否为 OpenClaw 进程。

### 5. 修复可达性（Reachability）

状态图显示 `Gateway: unreachable (connect failed: ECONNREFUSED)`。若服务启动后仍无法连接，运行诊断命令：

```bash
openclaw gateway probe
```

---

## 总结建议

| 步骤 | 操作 |
|------|------|
| 1️⃣ | `pkill -f openclaw` 清理残留 |
| 2️⃣ | `systemctl --user start openclaw-gateway.service` 启动服务 |
| 3️⃣ | 启动后立即查看日志：`journalctl -u openclaw-gateway.service -f -n 50` |
| 4️⃣ | 若仍有报错，运行 `openclaw gateway probe` 并提供输出 |

> ✅ Telegram 测试已通过（HTTP 200），网络无问题，只需 Gateway 进程正常运行即可恢复。

---

*如需进一步分析 `openclaw gateway probe` 的输出，可以将其发送给我。*
