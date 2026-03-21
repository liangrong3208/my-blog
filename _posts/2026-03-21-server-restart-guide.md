---
layout: post
title: "服务器重启需要的操作"
date: 2026-03-21 15:56:00 +0000
categories: [技术, 运维]
tags: [openclaw, systemd, 服务器维护, 运维指南]
---

由于我们刚刚把 OpenClaw 升级为了**系统级全局服务（System-wide Service）**，现在的操作已经变得非常标准且"一劳永逸"了。

无论是平时修改配置后的"软重启"，还是整个阿里云 VPS 物理宕机后的"冷启动"，你只需要记住以下几条指令：

---

### 1. 平时修改配置后的重启

如果你改了 `openclaw.json` 里的模型参数或上下文长度，执行这一条即可：

```bash
systemctl restart openclaw-gateway
```

---

### 2. 物理重启服务器后（自动运行）

你不需要手动操作。因为我们刚才执行了 `systemctl enable openclaw-gateway`，这会在系统内核里写下一个"预约单"。

- 当你执行 `reboot` 重启阿里云 VPS 后，系统只要加载完网络驱动，就会**自动**拉起 OpenClaw。
- 你上线后只需要用 `systemctl status openclaw-gateway` 确认一下它是否已经"绿了"即可。

---

### 3. 如果重启后发现"不回话"的应急预案

万一重启后发现 Telegram 没反应，按这个顺序点检：

| 检查项 | 命令 | 预期结果 |
| :--- | :--- | :--- |
| 进程状态 | `systemctl status openclaw-gateway` | 看到 `active (running)` |
| 端口占用 | `ss -tlnp \| grep 18789` | 看到 openclaw 正在监听 |
| 实时日志 | `journalctl -u openclaw-gateway -f` | 看到 `[telegram] listening` |

---

### 🛠 建议：存一个"救命"别名

为了方便你以后在 root 下一键维护，建议你把这行代码贴进终端（执行一次即可）：

```bash
echo "alias claw='systemctl status openclaw-gateway'" >> ~/.bashrc
echo "alias reclaw='systemctl restart openclaw-gateway'" >> ~/.bashrc
source ~/.bashrc
```

这样以后：
- 输入 `claw`：直接看它活没活着。
- 输入 `reclaw`：直接重启服务。

---

现在服务器状态稳如老狗。
