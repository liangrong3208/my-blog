---
layout: post
title: "OpenClaw 阿里云 VPS 故障排查与修复全记录"
date: 2026-03-21 15:49:00 +0000
categories: [技术, 运维]
tags: [openclaw, vps, systemd, 故障排查, 阿里云]
---

## 🧩 OpenClaw 阿里云 VPS 故障排查与修复全记录

### 一、故障现象

- Telegram 机器人"已读不回"
- `openclaw status` 显示 `Gateway: unreachable (missing scope: operator.read)`
- 日志中出现 `Port 18789 is already in use` 和 `lock timeout after 5000ms`
- 服务不断重启（`Main process exited, code=exited, status=1/FAILURE`）

---

### 二、根因分析

通过 `sudo lsof -i :18789` 和 `ps -fj -p <PID>` 定位到：

- 存在一个 **root 用户级 systemd 服务**（`/root/.config/systemd/user/openclaw-gateway.service`）抢占 18789 端口。
- 该服务被设置为开机自启（`default.target.wants`），导致每次重启后 root 进程先行启动。
- 后续任何通过 `systemctl --user` 或 `systemctl start` 的启动尝试，都会因端口冲突而失败。
- 同时，`openclaw-gateway` 可执行文件的实际路径为 `/usr/bin/openclaw`，但服务文件中错误使用了 `openclaw-gateway`，导致 `203/EXEC` 错误。

---

### 三、解决步骤

#### 1. 定位并移除 root 私有服务

```bash
find /root/.config/systemd/user/ -name "openclaw*"
# 找到后停止并禁用
systemctl --user stop openclaw-gateway.service
systemctl --user disable openclaw-gateway.service
rm -f /root/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

#### 2. 清理残留进程与锁文件

```bash
pkill -9 openclaw
rm -rf /tmp/openclaw/*.lock
rm -rf /tmp/openclaw/gateway.pid
```

#### 3. 确认 openclaw 真实路径

```bash
type -a openclaw
# 输出：openclaw is /usr/bin/openclaw
```

#### 4. 创建全局 systemd 服务（`/etc/systemd/system/`）

```bash
cat <<'EOF' > /etc/systemd/system/openclaw-gateway.service
[Unit]
Description=OpenClaw Gateway (Global Root)
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
ExecStart=/usr/bin/openclaw gateway run
Restart=always
RestartSec=5
MemoryLimit=800M

[Install]
WantedBy=multi-user.target
EOF
```

#### 5. 启用并启动服务

```bash
systemctl daemon-reload
systemctl enable openclaw-gateway
systemctl start openclaw-gateway
systemctl status openclaw-gateway
```

---

### 四、最终状态验证

- ✅ 服务状态：`active (running)`
- ✅ 内存限制生效：`Memory: 88.9M (limit: 800.0M)`
- ✅ 端口监听正常：`lsof -i :18789` 显示为 root 进程
- ✅ Telegram 机器人恢复响应，`/status` 正常返回
- ✅ 上下文窗口恢复为 128k

---

### 五、关键经验总结

| 问题类型 | 原因 | 解决方案 |
|----------|------|----------|
| 端口冲突 | root 私有 systemd 服务抢占端口 | 移除 `/root/.config/systemd/user/` 中的服务文件 |
| 进程残留 | 手动运行或崩溃后未清理 | `pkill -9 openclaw` + 删除 `/tmp/openclaw/*.lock` |
| systemd 启动失败 | ExecStart 路径错误 | 使用 `type -a openclaw` 找到真实路径，写为绝对路径 |
| 配置隔离混乱 | 混淆用户级与系统级服务 | 统一使用 `/etc/systemd/system/` 全局服务 |
| 内存溢出风险 | 无限制 | 添加 `MemoryLimit=800M` 防止 OOM |

---

### 六、运维建议

- **统一服务管理**：阿里云 VPS 为单 root 环境，所有服务统一用 `systemctl`（不带 `--user`）管理。
- **配置备份**：将 `/etc/systemd/system/openclaw-gateway.service` 纳入版本控制或本地备份。
- **日志监控**：定期检查 `journalctl -u openclaw-gateway -n 50`，关注 `missing scope` 或 `lock timeout` 等关键词。
- **内存告警**：若未来任务加重，可适当调高 `MemoryLimit`，但需监控 VPS 整体负载。

---

*如果需要，可以将这套修复流程封装成一个 `fix-openclaw.sh` 脚本，方便以后出现类似问题时一键恢复。*
