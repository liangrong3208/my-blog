---
layout: post
title: "OpenClaw 完整备份与恢复流程总结"
date: 2026-03-18 12:47:06
categories: [技术, OpenClaw]
tags: [OpenClaw, 备份, 恢复, 迁移, WSL]
---

## 📦 OpenClaw 完整备份与恢复流程总结

根据我们这整个对话，你成功地将一个 126MB 的 OpenClaw 备份从一台电脑迁移到了新电脑（WSL环境）。以下是整个流程的完整总结，供你以后参考：

---

### 🖥 第一阶段：旧电脑上的备份

**目标**：创建包含完整配置、记忆、技能的备份包

```bash
# 1. 停止OpenClaw服务（确保文件一致）
openclaw gateway stop

# 2. 创建完整备份（包含工作区、技能、记忆）
openclaw backup create

# 3. 找到生成的备份文件（通常在~/或指定目录）
ls -lh openclaw-backup-*.tar.gz
# 示例输出：openclaw-backup-20260318.tar.gz (126MB)
```

**备份包含内容**：
- ✅ 所有配置和状态数据 (`openclaw.json`)
- ✅ 工作区文件（配送费记录、脚本等）
- ✅ 技能模块（QVeris等）
- ✅ 记忆文件 (`MEMORY.md`)
- ✅ 博客仓库数据

---

### 📤 第二阶段：传输备份文件

**方法A：通过Telegram传输（适合远程迁移）**
- Telegram免费版支持 2GB 单文件，你的126MB完全没问题
- 发送到"已保存消息"或自己的另一个账号
- 注意：备份包含API密钥，传输后建议从聊天中删除

**方法B：通过USB/局域网传输**
- 直接复制到U盘或通过局域网共享

**方法C：分卷传输与合并**
- 如果Telegram自动分卷（如 `-part-aa`、`-part-ab`），在Windows合并：
```
copy /b openclaw-backup-part-aa.gz + openclaw-backup-part-ab + openclaw-backup-part-ac openclaw-backup-full.tar.gz
```

---

### 📱 第三阶段：特殊情况——恢复至手机 (Flutter App)

如果你需要恢复回手机版OpenClaw：

**关键：找到文件交换目录**

1. **授予存储权限**：手机设置 → 应用 → OpenClaw → 权限 → 允许"文件和媒体"
2. **查找交换目录**（按可能性排序）：
   ```
   /storage/emulated/0/Android/data/com.nxg.openclawproot/files/
   /storage/emulated/0/OpenClaw/
   /storage/emulated/0/Documents/OpenClaw/
   ```
3. **将备份文件放入交换目录**
4. 通过App内终端或Quick Actions恢复

---

### 🐧 第四阶段：恢复至新电脑 (WSL环境) —— 你刚完成的步骤

**第一步：准备WSL环境**

```bash
# 进入WSL终端
cd ~  # 进入用户主目录 (/home/liangrong)
pwd   # 确认路径
```

**第二步：放置备份文件**
- 通过Windows资源管理器访问：`\\wsl.localhost\Ubuntu\home\liangrong\`
- 将备份文件复制到此目录

**第三步：恢复备份**

```bash
# 1. 停止OpenClaw
openclaw gateway stop

# 2. 解压备份（直接覆盖）
tar -xzvf openclaw-backup-full.tar.gz -C ./

# 3. 修复文件所有权
sudo chown -R $USER:$USER ~/.openclaw

# 4. 运行诊断和自动修复（最关键步骤）
openclaw doctor --fix

# 5. 启动网关
openclaw gateway start

# 6. 验证状态
openclaw status
```

**第四步：验证恢复结果**

```bash
# 检查记忆是否完整
cat ~/.openclaw/workspace/MEMORY.md | head

# 列出技能
openclaw skill list

# 和AI对话验证
openclaw talk "请确认你的记忆是否完整，还记得我们的配送费记录吗？"
```

---

### ⚠️ 关键注意事项

| 注意事项 | 说明 |
|---------|------|
| 🔐 安全 | 备份包含API密钥，传输后及时删除聊天记录 |
| 👤 权限 | 跨环境迁移后务必运行 chown 和 doctor --fix |
| 📁 路径 | WSL中用户目录是 `/home/用户名/`，不是 `/用户名/` |
| 🔄 服务状态 | 恢复前确保OpenClaw已停止，恢复后重启 |
| 💾 备份完整性 | 126MB是合理大小，包含完整环境 |

---

### 🎯 最终成果

你的新电脑上的OpenClaw现在应该：
- ✅ 拥有完全相同的记忆 (MEMORY.md)
- ✅ 拥有所有技能（QVeris等）
- ✅ 保留配送费记录和工作数据
- ✅ 保持原有的身份设定
- ✅ 可以无缝继续之前的工作

恭喜！你的AI成功"搬家"了！ 🎉

如果以后需要再次迁移，这个流程可以作为标准操作手册。有任何问题随时回来问。
