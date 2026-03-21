---
layout: post
title: "高级防护：crowdsec 安装 + SSH 并发连接限制"
date: 2026-03-21 08:47:00 +0000
categories: [技术, 安全]
tags: [crowdsec, SSH, 入侵防御, 并发限制]
---

好的，这两个是更高级的防护措施，我们一步步来。**先用 SSH 安装 crowdsec**，然后用 KVM 控制台限制 SSH 并发连接数（因为改 SSH 配置后需要重启服务，可能会断开当前连接，用 KVM 更安全）。

---

## 📦 第一部分：安装 crowdsec（通过 SSH 执行）

### 1️⃣ 安装 crowdsec 和防火墙组件

```bash
# 添加官方仓库并安装
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | bash
apt install crowdsec crowdsec-firewall-bouncer-iptables -y
```

### 2️⃣ 查看 crowdsec 状态

```bash
# 查看服务状态
systemctl status crowdsec
systemctl status crowdsec-firewall-bouncer

# 查看当前封禁的 IP
cscli decisions list
```

### 3️⃣ 添加常用的防护场景（可选）

```bash
# 更新场景库
cscli hub update

# 添加 SSH 暴力破解防护
cscli collections install crowdsecurity/ssh

# 添加 HTTP 防护（如果你的服务器有 web 服务）
cscli collections install crowdsecurity/http-cve

# 查看已安装的集合
cscli collections list
```

### 4️⃣ 验证 iptables 规则是否生效

```bash
# 查看 crowdsec 添加的 iptables 链
iptables -L | grep -i crowdsec
```

---

## 🔧 第二部分：限制 SSH 并发连接数（用 KVM 控制台执行）

### 1️⃣ 登录衡天云后台 → 找到你的服务器 → 点击「KVM 控制台」

### 2️⃣ 编辑 SSH 配置文件

```bash
nano /etc/ssh/sshd_config
```

### 3️⃣ 添加或修改以下配置

```
# 限制未认证连接数，防止被占满
MaxStartups 3:50:10

# 限制每个 IP 的最大并发会话数
MaxSessions 3

# 登录超时时间（秒）
LoginGraceTime 30
```

**参数说明：**

| 参数 | 作用 |
|------|------|
| `MaxStartups 3:50:10` | 连接数 <3 时全接受；3-10 之间随机拒绝 50%；>10 时全部拒绝 |
| `MaxSessions 3` | 每个 IP 最多同时保持 3 个 SSH 会话 |
| `LoginGraceTime 30` | 登录超时 30 秒，防止僵尸连接占着不放 |

### 4️⃣ 保存并重启 SSH

```bash
# 保存退出（Ctrl+O, Enter, Ctrl+X）
# 检查配置是否正确
sshd -t

# 如果没有报错，重启 SSH
systemctl restart sshd
```

### 5️⃣ 验证配置生效

```bash
# 查看当前 SSH 连接数
ss -tn | grep :22 | wc -l

# 查看每个 IP 的连接情况
ss -tn | grep :22 | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn
```

---

## 🧪 测试防护效果

### 查看 crowdsec 封禁了哪些 IP

```bash
# 查看所有封禁决策
cscli decisions list

# 查看最近被攻击的日志
cscli alerts list

# 查看 crowdsec 日志
tail -50 /var/log/crowdsec.log
```

### 查看 SSH 连接限制是否生效

```bash
# 模拟多连接（从另一台机器快速多次 ssh，会看到部分连接被拒绝）
for i in {1..10}; do ssh -o ConnectTimeout=5 root@206.119.179.243 "echo test" & done
```

---

## 📊 两个工具的分工

| 工具 | 作用 | 特点 |
|------|------|------|
| **crowdsec** | 智能入侵防御，自动封禁全球恶意 IP | 社区共享威胁情报，能识别新型攻击 |
| **MaxStartups** | 限制 SSH 并发连接数 | 内核级别限制，防止连接数被占满 |

**两者互补**：crowdsec 负责识别和封禁攻击者，MaxStartups 负责防止瞬时大量连接挤爆 SSH。

---

## ✅ 完成检查清单

```bash
# 1. crowdsec 服务状态
systemctl status crowdsec --no-pager | head -5

# 2. 防火墙 bouncer 状态
systemctl status crowdsec-firewall-bouncer --no-pager | head -5

# 3. SSH 配置是否正确
sshd -t

# 4. 当前 SSH 连接数
ss -tn | grep :22 | wc -l
```

现在先执行第一部分的 curl 命令安装 crowdsec，告诉我结果。
