---
layout: post
title: "📋 OpenClaw 完整安装命令手册"
date: 2026-03-16 00:33:44
categories: [tutorial, openclaw, installation, commands]
tags: [wsl, v2ray, telegram, ai-assistant, command-line]
---

## 📋 OpenClaw 完整安装命令手册

这是一个从零开始到成功安装 OpenClaw 的完整命令集，按阶段整理，每一步都是经过验证的正确操作。

---

## 第一阶段：Windows PowerShell 准备工作

### 1.1 测试网络环境
```powershell
# 测试国内网络（确认基础网络通）
Test-Connection baidu.com -Count 4

# 测试代理端口是否开放（确认 V2Ray 运行）
Test-NetConnection -ComputerName "127.0.0.1" -Port 10808
```

### 1.2 设置 Windows 代理（关键！）
```powershell
# 正确格式：http://IP:端口 （不要重复 http://）
$env:HTTP_PROXY = "http://127.0.0.1:10808"
$env:HTTPS_PROXY = "http://127.0.0.1:10808"

# 验证代理是否生效
try {
 $response = Invoke-WebRequest -Uri "https://www.google.com" -UseBasicParsing -TimeoutSec 10
 Write-Host "✅ 代理正常，状态码: $($response.StatusCode)" -ForegroundColor Green
} catch {
 Write-Host "❌ 代理失败: $_" -ForegroundColor Red
}

# 查看当前出口 IP
$response = Invoke-WebRequest -Uri "https://api.ipify.org" -UseBasicParsing
Write-Host "当前出口 IP: $($response.Content)" -ForegroundColor Yellow
```

### 1.3 安装 WSL Ubuntu
```powershell
# 设置 WSL2 为默认版本
wsl --set-default-version 2

# 安装 Ubuntu 22.04 LTS
wsl --install -d Ubuntu-22.04
```
> 注意：安装过程中会提示创建用户名和密码，输入 `liangrong` 和密码即可。

---

## 第二阶段：WSL Ubuntu 基础配置

### 2.1 进入 Ubuntu
```bash
# 在 PowerShell 中输入
wsl
```
命令行提示符变成：`liangrong@DESKTOP-5PE7T00:~$`

### 2.2 配置 Ubuntu 代理
```bash
# 获取 Windows 主机 IP（关键！）
host_ip=$(ip route | grep default | awk '{print $3}')
echo "Windows 主机 IP: $host_ip"

# 设置代理（用获取到的 IP）
export http_proxy="http://$host_ip:10808"
export https_proxy="http://$host_ip:10808"

# 验证代理设置
echo "HTTP_PROXY: $http_proxy"
echo "HTTPS_PROXY: $https_proxy"

# 测试代理是否生效
curl -I https://www.google.com
curl https://api.ipify.org # 应该显示代理 IP
```

### 2.3 永久保存代理配置（可选但推荐）
```bash
# 编辑 .bashrc 文件
nano ~/.bashrc
```
在文件末尾添加：
```bash
# WSL 代理自动配置
export host_ip=$(ip route | grep default | awk '{print $3}')
export http_proxy="http://$host_ip:10808"
export https_proxy="http://$host_ip:10808"
```
保存：`Ctrl+O` → 回车 → Ctrl+X 
```bash
# 使配置生效
source ~/.bashrc
```

---

## 第三阶段：安装 OpenClaw

### 3.1 运行官方安装脚本
```bash
# 执行安装（关键：不要中途中断！）
curl -fsSL https://openclaw.ai/install.sh | bash
```
> 耐心等待：这个过程会自动安装 Node.js、编译工具，可能需要 3-5 分钟。 
> 当提示 `[sudo] password for liangrong:` 时，输入你的 Ubuntu 密码。

### 3.2 安装成功后的提示
安装成功会看到：
```
🦞 OpenClaw installed successfully (OpenClaw 2026.3.13)!
! PATH missing npm global bin dir: /home/liangrong/.npm-global/bin
```

### 3.3 修复 PATH 问题（必须做！）
```bash
# 临时添加到当前会话
export PATH="/home/liangrong/.npm-global/bin:$PATH"

# 永久添加到 .bashrc
echo 'export PATH="/home/liangrong/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 验证命令可用
openclaw --version
```
应该显示：`OpenClaw 2026.3.13 (61d171a)`

---

## 第四阶段：OpenClaw 配置向导

### 4.1 启动配置向导
```bash
openclaw onboard
```

### 4.2 配置选项（按顺序回答）

| 提示 | 输入/选择 | 说明 |
|------|----------|------|
| I understand this is personal-by-default... | Yes | 确认风险 |
| Onboarding mode | 选择 QuickStart | 快速启动模式 |
| Model/auth provider | 选择 OpenAI | 选择 AI 模型提供商 |
| OpenAI auth method | OpenAI API key | 使用 API Key |
| How do you want to provide this API key? | Paste API key now | 粘贴 API Key |
| Enter OpenAI API key | sk-sp-bad612b460904e8b948019269cb2f917 | 输入你的 API Key |
| Default model | Keep current | 保持默认模型 |
| Select channel (QuickStart) | Telegram (Bot API) | 选择 Telegram 频道 |
| How do you want to provide this Telegram bot token? | Enter Telegram bot token | 输入 Bot Token |
| Enter Telegram bot token | 8224659764:AAFQK_6oSVLrTF8OJ2gNVXiftTBbQuMLDeg | 输入你的 Bot Token |
| Search provider | Skip for now | 暂时跳过 |
| Configure skills now? | Yes | 配置技能 |
| Install missing skill dependencies | Skip for now | 跳过依赖安装 |
| Set GOOGLE_PLACES_API_KEY? | No | 跳过 |
| Set GEMINI_API_KEY? | No | 跳过 |
| Set NOTION_API_KEY? | No | 跳过 |
| Set OPENAI_API_KEY for openai-image-gen? | No | 跳过 |
| Set OPENAI_API_KEY for openai-whisper-api? | No | 跳过 |
| Set ELEVENLABS_API_KEY for sag? | No | 跳过 |
| Enable hooks? | Skip for now | 跳过钩子 |
| How do you want to hatch your bot? | Open the Web UI | 打开 Web 界面 |

### 4.3 配置完成
配置完成后会看到 Dashboard 链接：
```
http://127.0.0.1:18789/#token=958379670d316093a37232b145e0b7fec272df93688d8bc0
```

---

## 第五阶段：启动和修复服务

### 5.1 首次启动可能遇到的错误
```bash
# 尝试启动（可能报配置文件错误）
openclaw gateway restart
```
如果看到类似错误：
```
Failed to read config... SyntaxError: JSON5: invalid character ','
agents: Unrecognized key: "workspace"
```

### 5.2 运行 doctor 自动修复
```bash
# 自动修复配置文件
openclaw doctor --fix
```

### 5.3 正常启动服务
```bash
# 重启网关
openclaw gateway restart

# 查看状态
openclaw gateway status
```
# 应该看到：Runtime: running (pid xxxx)

### 5.4 Telegram 配对（如果需要）
```bash
# 在 Telegram 中向你的 Bot 发送 /pair
# 会收到一个配对码，比如 6YXQMNRQ

# 在 Ubuntu 中批准
openclaw pairing approve telegram 6YXQMNRQ
```
应该看到：`Approved telegram sender 1751810419.`

---

## 第六阶段：验证和使用

### 6.1 打开 Web 控制台
```bash
# 方式1：直接打开（会自动在浏览器中打开）
openclaw dashboard

# 方式2：手动访问（复制上面的链接）
http://127.0.0.1:18789/#token=958379670d316093a37232b145e0b7fec272df93688d8bc0
```

### 6.2 在 Telegram 中使用
1. 在 Telegram 中找到 @Liangrongbot
2. 发送 /start 开始对话
3. 直接发送消息即可与 AI 对话

### 6.3 常用管理命令
```bash
# 查看状态
openclaw gateway status

# 查看实时日志
openclaw logs follow

# 停止服务
openclaw gateway stop

# 启动服务
openclaw gateway start

# 重启服务
openclaw gateway restart

# 查看配置
openclaw config list

# 再次运行 doctor（定期检查）
openclaw doctor
```

---

## 第七阶段：Windows 退出后的再次进入

### 7.1 重新进入 Ubuntu
```bash
# 在 PowerShell 中
wsl
```

### 7.2 检查代理是否生效
```bash
# 检查 PATH
echo $PATH | grep .npm-global

# 检查 openclaw 是否可用
openclaw --version

# 检查服务状态
openclaw gateway status
```

### 7.3 如果 PATH 丢失
```bash
# 临时修复
export PATH="/home/liangrong/.npm-global/bin:$PATH"

# 检查 .bashrc 是否有配置
cat ~/.bashrc | grep PATH
```

---

## 🔑 关键要点总结

### ✅ 网络配置要点
1. Windows 代理：`http://127.0.0.1:10808`（不要重复 http://）
2. WSL 代理：用 172.28.224.1 而不是 127.0.0.1
3. 测试用 curl 而不是 ping

### ✅ 安装要点
1. **不要中途中断**安装脚本（这是上次失败的原因）
2. 安装后**必须**将 `~/.npm-global/bin` 添加到 PATH
3. 配置错误用 `openclaw doctor --fix` 自动修复

### ✅ 成功标志
- openclaw --version 显示版本号
- openclaw gateway status 显示 Runtime: running
- Telegram Bot 能回复消息
- Web 控制台能打开

---

## 📝 一句话总结

先配好代理（Windows 用 127.0.0.1，WSL 用 172.28.224.1），然后耐心跑完安装脚本，最后加 PATH 跑 doctor，就能得到一只活蹦乱跳的 OpenClaw！ 🦞

有任何问题，随时用这个手册对照检查！