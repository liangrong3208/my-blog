---
layout: post
title: "OpenClaw 全量备份指南（手动版）"
date: 2026-03-21 14:04:00 +0000
categories: [技术, 运维]
tags: [openclaw, 备份, 运维指南]
---

## 📦 OpenClaw 全量备份指南（手动版）

### 一、核心文件清单

| 文件/目录 | 路径 | 内容 |
|----------|------|------|
| agent.md | `~/.openclaw/agents/main/agent.md` | 手动焊死的"人设"和硬规则 |
| MEMORY.md | `~/.openclaw/workspace/MEMORY.md` | 系统自动生成的长期记忆摘要 |
| sessions/ | `~/.openclaw/agents/main/sessions/` | 对话上下文 JSON 文件 |
| 配置文件 | `~/.openclaw/openclaw.json` | OpenClaw 核心配置 |
| 数据库 | `~/.openclaw/data/openclaw.db` | 向量记忆数据库（若存在） |

> ⚠️ 你的环境中 openclaw.db 不存在，说明使用的是 JSON 存储，sessions 文件夹就是核心。

---

### 二、全量备份命令（一键打包）

```bash
# 进入家目录
cd ~

# 创建备份包（排除日志和缓存）
tar -czvf openclaw_soul_full_$(date +%F).tar.gz \
  --exclude='.openclaw/cache' \
  --exclude='.openclaw/logs' \
  .openclaw
```

**生成的备份文件**：`/root/openclaw_soul_full_YYYY-MM-DD.tar.gz`

---

### 三、验证备份包内容

```bash
# 查看备份包大小
ls -lh ~/openclaw_soul_full_*.tar.gz

# 查看包内关键文件列表
tar -tvf ~/openclaw_soul_full_*.tar.gz | grep -E 'agent.md|MEMORY.md|sessions/'
```

---

### 四、整理备份文件（可选）

```bash
# 创建专用备份目录
mkdir -p ~/openclaw_backups

# 移动备份包到目录中
mv ~/openclaw_soul_full_*.tar.gz ~/openclaw_backups/

# 确认已入库
ls -lh ~/openclaw_backups/
```

---

### 五、下载到本地电脑（异地冷备份）

在**本地电脑**（PowerShell 或 Terminal）执行：

```bash
scp root@206.119.179.243:/root/openclaw_backups/openclaw_soul_full_2026-03-21.tar.gz .
```

> 将文件名中的日期替换为实际日期，或使用通配符：`openclaw_soul_full_*.tar.gz`

---

### 六、恢复备份（如果需要）

```bash
# 先备份当前目录（可选）
mv ~/.openclaw ~/.openclaw_old

# 解压备份包
tar -xzvf ~/openclaw_backups/openclaw_soul_full_2026-03-21.tar.gz -C ~

# 重启服务
systemctl --user restart openclaw-gateway.service
```

---

### 七、你的当前备份状态

从你的执行记录看：

- ✅ 备份文件已生成：`/root/openclaw_all_in_one_2026-03-21.tar.gz`
- ✅ 包含关键文件：
  - agent.md (1.3KB)
  - MEMORY.md (4.9KB)
  - `sessions/`（含 1.3MB 的对话上下文）
- ✅ 服务正常运行
- ✅ Telegram 通道正常

---

### 八、建议操作

1. **下载备份到本地**（执行上面的 scp 命令）
2. **清理服务器根目录**（可选）：
   ```bash
   rm /root/openclaw_all_in_one_2026-03-21.tar.gz
   ```
3. **定期手动备份**：在重大记忆更新后重复以上命令

---

如果你需要，可以写一个 **alias 缩写**，让你以后只需输入 `backup-soul` 就能一键完成打包和归档。
