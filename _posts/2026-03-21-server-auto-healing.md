---
layout: post
title: "服务器自愈配置：fail2ban + OpenClaw 自动恢复"
date: 2026-03-21 08:44:00 +0000
categories: [技术, 运维]
tags: [fail2ban, OpenClaw, 自动化, 服务器监控]
---

抱歉，是我记混了——我们确实讨论了这两个方案，但还没有实际执行。既然现在 SSH 已经稳定，我们正好可以把它们加上，让服务器更"自愈"。

---

## ✅ 现在来做这两件事（通过 SSH 执行，不用 KVM）

### 1️⃣ 安装并配置 fail2ban

```bash
# 安装
apt update && apt install fail2ban -y

# 创建配置（针对 SSH 防护）
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 86400
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400
EOF

# 启动并设为开机自启
systemctl enable fail2ban --now

# 查看状态
fail2ban-client status sshd
```

如果看到 `Status: Enabled` 就说明成功了。

---

### 2️⃣ 添加自动恢复脚本（每 5 分钟检查 OpenClaw）

```bash
# 创建脚本
cat > /root/check_openclaw.sh << 'EOF'
#!/bin/bash
# 检查 OpenClaw 健康状态，不健康则重启

if ! curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:18789/health | grep -q "200"; then
    echo "$(date) - OpenClaw unhealthy, restarting..." >> /var/log/openclaw_restart.log
    systemctl restart openclaw-gateway.service
    sleep 5
    echo "$(date) - Restart completed." >> /var/log/openclaw_restart.log
fi
EOF

# 赋予执行权限
chmod +x /root/check_openclaw.sh

# 添加定时任务（每 5 分钟运行一次）
(crontab -l 2>/dev/null; echo "*/5 * * * * /root/check_openclaw.sh > /dev/null 2>&1") | crontab -

# 查看定时任务
crontab -l
```

---

## 🧪 验证

```bash
# 检查 fail2ban 状态
fail2ban-client status sshd

# 手动运行一次脚本，看有没有报错
/root/check_openclaw.sh

# 查看脚本日志（如果有重启记录）
cat /var/log/openclaw_restart.log
```

---

## 📝 这两个的作用

| 工具 | 作用 |
|------|------|
| fail2ban | 自动封禁尝试暴力破解 SSH 的 IP，从源头减少攻击 |
| 自动恢复脚本 | 如果 OpenClaw 因网络波动挂了，5 分钟内自动重启，你无需手动干预 |

现在执行第一部分的 `apt install fail2ban -y`，然后告诉我输出。
