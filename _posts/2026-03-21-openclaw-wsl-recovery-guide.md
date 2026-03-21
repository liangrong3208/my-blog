---
layout: post
title: "OpenClaw 本地 WSL 恢复与记忆同步全记录"
date: 2026-03-21 14:42:00 +0000
categories: [技术, 运维]
tags: [openclaw, wsl, 备份恢复, 代理配置]
---

## 🧠 OpenClaw 本地 WSL 恢复与记忆同步全记录

### 一、问题背景

服务器上的 OpenClaw 已稳定运行，但希望在本地 WSL 中复刻完全相同的环境（包括 agent.md 规则、MEMORY.md 记忆、128k 上下文、会话历史），以便本地调试和开发。

---

### 二、核心操作步骤

#### 1. 备份从服务器下载到本地

```bash
# 在本地 PowerShell 执行（下载备份包）
scp root@206.119.179.243:/root/openclaw_all_in_one_2026-03-21.tar.gz .
```

#### 2. 解压并修正路径（/root → /home/liangrong）

```bash
# 解压到 WSL 家目录
tar -xzvf openclaw_all_in_one_2026-03-21.tar.gz -C ~

# 批量替换所有文件中的路径
sed -i "s/\/root\//\/home\/liangrong\//g" ~/.openclaw/openclaw.json
find ~/.openclaw -type f -name "*.json*" -exec sed -i "s/\/root\//\/home\/liangrong\//g" {} +

# 修正文件所有权（避免 permission denied）
sudo chown -R $USER:$USER ~/.openclaw
chmod -R 755 ~/.openclaw/agents/main
```

#### 3. 扩大上下文窗口（16k → 128k）

```bash
sed -i 's/16000/128000/g' ~/.openclaw/openclaw.json
```

#### 4. 解决 Token 认证问题（__OPENCLAW_REDACTED__ → 真实 Token）

```bash
# 手动写入一个简单 Token
sed -i 's/"token": "__OPENCLAW_REDACTED__"/"token": "1234567890abcdef"/g' ~/.openclaw/openclaw.json

# 同步 CLI 配置
openclaw config set gateway.auth.token "1234567890abcdef"
```

#### 5. 配置 WSL 代理穿透（解决 fetch failed）

```bash
# 获取 Windows 宿主机 IP
HOST_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}')
PROXY_URL="http://${HOST_IP}:10808"

# 写入 openclaw.json 的模型配置中（如已有 proxy 字段）
sed -i "s#\"proxy\": \".*\"#\"proxy\": \"$PROXY_URL\"#g" ~/.openclaw/openclaw.json

# 若无 proxy 字段，手动添加：
nano ~/.openclaw/openclaw.json
# 在 custom-coding-dashscope-aliyuncs-com 段内添加：
# "proxy": "http://172.28.224.1:10808"

# 测试代理是否通
curl -I -x $PROXY_URL https://www.google.com
```

#### 6. 重启服务并重建记忆索引

```bash
# 停止旧服务
openclaw gateway stop

# 前台启动（观察日志）
openclaw gateway run --force

# 另开终端，重建向量索引（让服务器记忆生效）
openclaw memory index --force
```

#### 7. 验证最终状态

```bash
openclaw status
```

应显示：
- ✅ Gateway reachable
- ✅ 1 bootstrap file present
- ✅ kimi-k2.5 (128k ctx)
- ✅ Telegram ON / OK

---

### 三、关键问题与解决

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| missing scope: operator.read | Token 被脱敏为 __OPENCLAW_REDACTED__ | 手动写入真实 Token 并同步 CLI |
| EACCES: permission denied | 文件所有者仍为 root | chown -R $USER:$USER ~/.openclaw |
| fetch failed | WSL 无法访问 Windows 代理 | 设置 proxy 为宿主机 IP（如 172.x.x.x:10808） |
| unknown command 'rebuild' | 2026.3 版本命令变更 | 改用 openclaw memory index --force |
| 服务器记忆未加载 | 向量索引未重建 | 执行 openclaw memory index --force |

---

### 四、最终效果

在 Telegram 上测试，本地机器人已能：
- ✅ 正确读取 MEMORY.md 中的本地记录
- ✅ 通过代理调用阿里云模型
- ✅ 成功重建索引，可检索服务器端历史对话内容
- ✅ 完全复刻服务器的"灵魂"与"记忆"

---

### 五、后续维护建议

1. **定期备份**（本地 + 远程）：
   ```bash
   tar -czvf ~/openclaw_soul_full_$(date +%F).tar.gz --exclude='.openclaw/cache' --exclude='.openclaw/logs' -C ~ .openclaw
   ```

2. **同步服务器与本地记忆**：可考虑用 Git 管理 workspace/ 和 agents/main/sessions/ 中的关键文件。

3. **编写一键维护脚本**：将路径修正、Token 对齐、代理配置整合为 alias，方便后续快速重建。

---

*如果需要将这套流程整理成 Shell 脚本，或者进一步优化代理自动检测逻辑，随时告诉我。*
