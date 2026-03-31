---
layout: post
title: "systemd 开机自启配置指南"
date: 2026-03-29 13:19:00 +0000
categories: [linux]
tags: [systemd, 开机自启, 服务管理]
---

## systemd 开机自启配置指南

本文介绍如何在 Linux 系统中使用 systemd 配置服务开机自启。

### 创建服务文件

```bash
sudo cat > /etc/systemd/system/memory-watcher.service << 'EOF'
[Unit]
Description=Memory Continuity Watcher
After=network.target openclaw-gateway.service

[Service]
Type=simple
User=root
ExecStart=/root/.openclaw/skills/memory-continuity-service/memory_watcher.sh
Restart=always
RestartSec=10
StandardOutput=append:/var/log/memory-watcher.log
StandardError=append:/var/log/memory-watcher.log

[Install]
WantedBy=multi-user.target
EOF
```

### 启用并启动服务

```bash
# 重新加载 systemd
sudo systemctl daemon-reload

# 启用开机自启
sudo systemctl enable memory-watcher

# 启动服务
sudo systemctl start memory-watcher
```

### 管理命令

| 命令 | 说明 |
|------|------|
| `systemctl status memory-watcher` | 查看服务状态 |
| `systemctl stop memory-watcher` | 停止服务 |
| `systemctl start memory-watcher` | 启动服务 |
| `systemctl restart memory-watcher` | 重启服务 |
| `systemctl disable memory-watcher` | 禁用开机自启 |
| `journalctl -u memory-watcher -f` | 查看实时日志 |

### 配置说明

**[Unit] 部分**
- `Description`: 服务描述
- `After`: 指定依赖服务，本服务在依赖服务启动后才启动

**[Service] 部分**
- `Type=simple`: 简单服务类型
- `ExecStart`: 启动命令
- `Restart=always`: 总是自动重启
- `RestartSec=10`: 重启间隔 10 秒

**[Install] 部分**
- `WantedBy=multi-user.target`: 多用户模式下启动

### 实际案例

以 memory-watcher 服务为例：

```bash
# 查看状态
$ systemctl status memory-watcher
● memory-watcher.service - Memory Continuity Watcher
     Loaded: loaded (/etc/systemd/system/memory-watcher.service; enabled)
     Active: active (running) since Sun 2026-03-29 13:02:19 UTC
   Main PID: 276002 (memory_watcher.)
     Memory: 796.0K
```

### 故障排查

```bash
# 查看服务日志
journalctl -u memory-watcher --no-pager | tail -20

# 检查配置文件语法
systemd-analyze verify /etc/systemd/system/memory-watcher.service

# 查看所有失败的服务
systemctl --failed
```

---

*记录时间：2026-03-29 13:19 UTC*
