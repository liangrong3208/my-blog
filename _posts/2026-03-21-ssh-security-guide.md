---
layout: post
title: "SSH 服务器防护指南：保持 22 端口 + 强化安全"
date: 2026-03-21 08:31:00 +0000
categories: [技术, 安全]
tags: [SSH, fail2ban, 防火墙, 服务器安全]
---

明白了，我们不在端口上纠结了。既然改端口会导致连不上，说明衡天云的网络环境可能对高位端口有限制，或者防火墙策略比较严格。那我们就**保持默认的 22 端口，但加强防护**，这样既能连上，又能抵御攻击。

---

## ✅ 解决方案：保持 22 端口 + 强化防护

### 1️⃣ 先确认 SSH 已恢复到 22 端口

用 KVM 控制台登录服务器，执行：

```bash
# 确保 SSH 监听 22 端口
grep -i "^Port" /etc/ssh/sshd_config
```

如果显示 Port 22 或没有输出（默认22），就正常。如果是别的端口，改回来：

```bash
sed -i 's/^Port .*/Port 22/' /etc/ssh/sshd_config
systemctl restart sshd
```

---

### 2️⃣ 强化 fail2ban 配置（针对 22 端口）

```bash
# 编辑 fail2ban 配置
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 86400      # 封禁 24 小时
findtime = 600       # 10 分钟内
maxretry = 3         # 失败 3 次就封

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400

# 专门针对恶意扫描的规则
[sshd-ddos]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 2
findtime = 60
bantime = 86400

# 针对无效协议攻击（如你日志里的 GET /squid...）
[sshd-invalid]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 1
findtime = 30
bantime = 86400
EOF

# 重启 fail2ban
systemctl restart fail2ban

# 查看状态
fail2ban-client status sshd
```

---

### 3️⃣ 用 iptables 直接封锁已知攻击 IP

从你之前的日志里，我们已经看到几个攻击 IP，直接封掉：

```bash
# 封锁这些已知攻击 IP
iptables -A INPUT -s 45.148.10.121 -j DROP
iptables -A INPUT -s 103.191.243.59 -j DROP
iptables -A INPUT -s 180.137.97.48 -j DROP  # 这个是你自己的IP吗？如果不是也封掉

# 保存规则（Debian/Ubuntu）
apt install iptables-persistent -y
netfilter-persistent save
```

---

### 4️⃣ 限制 SSH 连接频率（可选）

在 `/etc/ssh/sshd_config` 中添加：

```
# 限制每分钟最多 3 次连接尝试
MaxStartups 3:50:10
# 每个 IP 最多 3 个并发连接
MaxSessions 3
```

重启 SSH：

```bash
systemctl restart sshd
```

---

### 5️⃣ 安装 crowdsec（更智能的防御，推荐）

crowdsec 是新一代的入侵检测系统，能自动封禁全球恶意 IP：

```bash
# 安装
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | bash
apt install crowdsec crowdsec-firewall-bouncer-iptables -y

# 查看封禁情况
cscli decisions list
```

---

## 📊 验证防护效果

24 小时后，检查日志：

```bash
# 查看今天 SSH 错误数量（应该大幅下降）
journalctl --since today -p 3 -x | grep sshd | wc -l

# 查看 fail2ban 封禁的 IP
fail2ban-client status sshd

# 查看 crowdsec 封禁的 IP（如果装了）
cscli decisions list
```

---

## 📝 总结

| 问题 | 解决方案 |
|------|----------|
| 改端口连不上 | 保持 22 端口 |
| 攻击太多导致服务中断 | fail2ban + iptables 封禁 |
| 想要更智能防护 | 安装 crowdsec |
| OpenClaw 被攻击影响 | 加强网络防护后，服务会更稳定 |

现在先执行**第二步（强化 fail2ban）**，这是最关键的一步。完成后告诉我，我们再观察一下 Telegram 和 Xray 是否还频繁断开。
