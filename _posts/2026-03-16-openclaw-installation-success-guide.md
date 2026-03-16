---
layout: post
title: "🎉 恭喜！OpenClaw 完美安装并运行成功！"
date: 2026-03-16 00:28:15
categories: [tutorial, openclaw, installation]
tags: [wsl, v2ray, telegram, ai-assistant]
---

## 🎉 恭喜！OpenClaw 完美安装并运行成功！
v2rayN，用的全局变量
让我帮你梳理一下这次成功的操作步骤，以后安装 OpenClaw 或者其他类似工具时，这就是一个完整的参考流程。

---

## 📋 完整的 OpenClaw 安装成功步骤总结

### 第一阶段：Windows 端准备（PowerShell）

#### 1. 确认网络环境
```powershell
# 测试国内网络（确认基础网络通）
Test-Connection baidu.com -Count 4

# 测试代理端口是否开放
Test-NetConnection -ComputerName "127.0.0.1" -Port 10808
```

#### 2. 正确设置代理（关键！）
```powershell
# 正确的代理格式（不要重复 http://）
$env:HTTP_PROXY = "http://127.0.0.1:10808"
$env:HTTPS_PROXY = "http://127.0.0.1:10808"

# 验证代理是否生效
Invoke-WebRequest -Uri "https://www.google.com" -UseBasicParsing
```

#### 3. 安装 WSL Ubuntu
```powershell
# 设置 WSL2 为默认版本
wsl --set-default-version 2

# 安装 Ubuntu 22.04
wsl --install -d Ubuntu-22.04
```

### 第二阶段：WSL Ubuntu 配置

#### 4. 配置 Ubuntu 代理
```bash
# 获取 Windows 主机 IP
ip route | grep default | awk '{print $3}'

# 设置代理（用获取到的 IP）
export http_proxy="http://172.28.224.1:10808"
export https_proxy="http://172.28.224.1:10808"

# 测试代理
curl -I https://github.com
curl https://api.ipify.org # 确认 IP 是代理出口
```

#### 5. 安装 OpenClaw
```bash
# 运行官方安装脚本（这次耐心等待，不要中断）
curl -fsSL https://openclaw.ai/install.sh | bash

# 安装脚本会自动：
# - 安装 Node.js 22
# - 安装编译工具
# - 通过 npm 安装 OpenClaw
```

### 第三阶段：OpenClaw 配置

#### 6. 修复 PATH 问题
```bash
# 安装完成后，需要将 npm 全局 bin 添加到 PATH
export PATH="/home/liangrong/.npm-global/bin:$PATH"
echo 'export PATH="/home/liangrong/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 验证命令可用
openclaw --version
```

#### 7. 运行配置向导
```bash
openclaw onboard
```

配置选项：
- ✅ 确认风险提示：`Yes`
- ✅ 模式选择：`QuickStart`
- ✅ 模型提供商：`OpenAI`
- ✅ API Key：`sk-sp-bad612b460904e8b948019269cb2f917`
- ✅ 默认模型：`Keep current`
- ✅ 选择频道：`Telegram`
- ✅ Telegram Token：`8224659764:AAFQK_6oSVLrTF8OJ2gNVXiftTBbQuMLDeg`
- ✅ Web搜索：`Skip for now`
- ✅ 配置技能：`Yes`（然后都选 No 跳过具体配置）
- ✅ Hooks：`Skip for now`

#### 8. 修复配置文件错误
```bash
# 运行 doctor 自动修复
openclaw doctor --fix
```

#### 9. 启动服务
```bash
# 重启网关
openclaw gateway restart

# 查看状态
openclaw gateway status

# 批准 Telegram 配对
openclaw pairing approve telegram 6YXQMNRQ
```

### 第四阶段：验证使用

#### 10. 访问 Web 控制台
http://127.0.0.1:18789/#token=958379670d316093a37232b145e0b7fec272df93688d8bc0

#### 11. 在 Telegram 中使用
- 找到你的 bot：`@Liangrongbot`
- 发送 /start 开始对话
- 发送 /pair 获取配对码
- 用 openclaw pairing approve telegram <配对码> 批准

---

## 🔑 成功的关键要点

### 1. 网络配置
- ✅ Ping 不通 Google 是正常的，要用 curl 或 Invoke-WebRequest
- ✅ 代理 URL 格式必须正确：`http://127.0.0.1:10808`（不要重复 http://）
- ✅ WSL 中要用 Windows 主机的 IP（172.28.224.1），而不是 127.0.0.1

### 2. 安装过程
- ✅ **耐心等待**：Node.js 安装和编译工具安装可能需要几分钟
- ✅ **不要中途中断**：这次成功是因为你让安装脚本完整运行
- ✅ **注意 sudo 密码提示**：安装 Node.js 时需要输入密码

### 3. PATH 问题
- ✅ 安装后记得将 `~/.npm-global/bin` 添加到 PATH
- ✅ 这是最常见的"command not found"原因

### 4. 配置修复
- ✅ `openclaw doctor --fix` 自动修复配置文件问题
- ✅ 这是解决配置错误的神器

---

## 🎯 最终成果

现在你拥有了：
1. ✅ Windows 端的 V2Ray 代理（端口 10808）
2. ✅ WSL Ubuntu 22.04 环境
3. ✅ OpenClaw 2026.3.13 安装成功
4. ✅ OpenAI GPT-5.1-codex 模型配置
5. ✅ Telegram Bot (@Liangrongbot) 集成
6. ✅ Web 控制台可访问
7. ✅ 系统服务自动运行

---

## 📝 一句话总结

这次成功的关键是： 在 WSL Ubuntu 中设置正确的代理（172.28.224.1:10808），然后**耐心让安装脚本完整运行**（不要中断），最后通过 `doctor --fix` 自动修复配置，再手动将 `~/.npm-global/bin` 添加到 PATH。

恭喜！你现在拥有了一个强大的 AI 助手！ 🦞✨