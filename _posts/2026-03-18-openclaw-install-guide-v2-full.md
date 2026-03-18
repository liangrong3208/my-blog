---
layout: post
title: "OpenClaw 完整安装命令手册第二版"
date: 2026-03-18 09:14:46
categories: [tutorial, openclaw, installation]
tags: [windows, wsl2, ubuntu, proxy, setup, guide]
---

# OpenClaw 完整安装配置指南（Windows + WSL2）

## 第四部分：配置 .bashrc 环境

### 4.1 编辑 .bashrc 文件

```bash
# 编辑 .bashrc 文件
nano ~/.bashrc

# 在文件末尾添加以下内容：
# ===== OpenClaw 配置开始 =====

# 添加 OpenClaw 到 PATH
export PATH="/home/liangrong/.npm-global/bin:$PATH"

# 代理配置函数
setup_proxy() {
 # 尝试使用 host.docker.internal（WSL2 推荐）
 if ping -c 1 host.docker.internal &> /dev/null; then
 export http_proxy="http://host.docker.internal:10808"
 export https_proxy="http://host.docker.internal:10808"
 export HTTP_PROXY=$http_proxy
 export HTTPS_PROXY=$https_proxy
 export NO_PROXY="localhost,127.0.0.1,.local,::1"
 echo "✓ 使用 host.docker.internal 代理"
 else
 # 备用方案：使用默认网关 IP
 HOST_IP=$(ip route | grep default | awk '{print $3}')
 if [ ! -z "$HOST_IP" ]; then
 export http_proxy="http://${HOST_IP}:10808"
 export https_proxy="http://${HOST_IP}:10808"
 export HTTP_PROXY=$http_proxy
 export HTTPS_PROXY=$https_proxy
 export NO_PROXY="localhost,127.0.0.1,.local,::1"
 echo "✓ 使用 ${HOST_IP} 代理"
 else
 echo "⚠️ 无法检测到代理地址"
 fi
 fi
}

# 取消代理的函数
unset_proxy() {
 unset http_proxy https_proxy HTTP_PROXY HTTPS_PROXY
 echo "✗ 代理已取消"
}

# 显示当前代理状态
show_proxy() {
 echo "当前代理设置："
 echo "http_proxy = $http_proxy"
 echo "https_proxy = $https_proxy"
}

# 自动设置代理
setup_proxy

# ===== OpenClaw 配置结束 =====

# 保存文件：按 Ctrl+O，然后 Enter
# 退出 nano：按 Ctrl+X

# 重新加载 .bashrc
source ~/.bashrc

# 验证 PATH 设置
echo $PATH | grep npm-global
# 应该显示包含 /home/liangrong/.npm-global/bin
```

## 第五部分：OpenClaw 交互式配置

### 5.1 启动配置向导

```bash
# 运行 OpenClaw（会自动启动配置向导）
openclaw
```

### 5.2 配置步骤详解

#### 步骤 1：安全确认
- Security warning – please read. [阅读安全警告]
- I understand this is personal-by-default... Continue? → **Yes**

#### 步骤 2：选择 onboarding 模式
- Onboarding mode → **QuickStart**

#### 步骤 3：选择模型提供商
- Model/auth provider → **OpenAI**
- OpenAI auth method → **OpenAI API key**
- How do you want to provide this API key? → **Paste API key now**
- Enter OpenAI API key → `sk-sp-bad612b460904e8b948019269cb2f917`
- Default model → **Keep current** (openai/gpt-5.1-codex)

#### 步骤 4：选择通信渠道
- Select channel (QuickStart) → **Telegram (Bot API)**

#### 步骤 5：配置 Telegram 机器人
- How do you want to provide this Telegram bot token? → **Enter Telegram bot token**
- Enter Telegram bot token → `8224659764:AAFQK_6oSVLrTF8OJ2gNVXiftTBbQuMLDeg`

#### 步骤 6：配置 Web 搜索
- Web search → **Skip for now**

#### 步骤 7：配置技能
- Configure skills now? (recommended) → **Yes**
- Install missing skill dependencies → **Skip for now**
- Set GOOGLE_PLACES_API_KEY for goplaces? → **No**
- Set GEMINI_API_KEY for nano-banana-pro? → **No**
- Set NOTION_API_KEY for notion? → **No**
- Set OPENAI_API_KEY for openai-image-gen? → **No**
- Set OPENAI_API_KEY for openai-whisper-api? → **No**
- Set ELEVENLABS_API_KEY for sag? → **No**

#### 步骤 8：配置 Hooks
- Enable hooks? → **Skip for now**

#### 步骤 9：配置完成
- How do you want to hatch your bot? → **Open the Web UI**

## 第六部分：修复配置文件问题

### 6.1 运行 Doctor 修复

```bash
# 第一次尝试运行命令可能遇到配置错误
openclaw gateway restart
# 可能显示：Failed to read config... invalid character ',' at 133:4

# 运行 doctor 修复配置文件
openclaw doctor --fix

# doctor 会修复的问题：
# - 移除无效的配置键（如 agents.workspace）
# - 创建缺失的目录（如 credentials）
# - 备份原配置文件
```

### 6.2 再次启动网关

```bash
# 重启网关服务
openclaw gateway restart

# 查看网关状态
openclaw gateway status
```

## 第七部分：Telegram 机器人配对

### 7.1 在 Telegram 中启动机器人

1. 打开 Telegram App
2. 搜索 `@Liangrongbot`（你创建的机器人用户名）
3. 点击 Start 或发送任意消息
4. 机器人会回复一个配对码，例如：`6YXQMNRQ`

### 7.2 在 WSL 中批准配对

```bash
# 批准 Telegram 配对
openclaw pairing approve telegram 6YXQMNRQ
# 输出：Approved telegram sender 1751810419.
```

## 第八部分：访问 OpenClaw 控制面板

### 8.1 查看网关状态获取访问地址

```bash
# 查看网关状态
openclaw gateway status

# 输出中会显示：
# Dashboard: http://127.0.0.1:18789/
# Dashboard: http://127.0.0.1:18789/#token=958379670d316093a37232b145e0b7fec272df93688d8bc0
```

### 8.2 从 Windows 访问 WSL 中的控制面板

#### 方法 A：直接在 WSL 中使用浏览器（如果有 GUI）

```bash
# 如果在 WSL 中安装了浏览器，可以直接打开
# 但通常 WSL 没有 GUI，所以使用方法 B
```

#### 方法 B：通过 SSH 端口转发（推荐）

在 Windows PowerShell 中执行：

```powershell
# 设置 SSH 端口转发
ssh -N -L 18789:127.0.0.1:18789 liangrong@localhost

# 保持此窗口运行，不要关闭
# 打开浏览器访问：
# http://localhost:18789/
# 或使用带 token 的链接：
# http://localhost:18789/#token=958379670d316093a37232b145e0b7fec272df93688d8bc0
```

## 第九部分：OpenClaw 常用命令

### 9.1 基本命令

```bash
# 查看版本
openclaw --version

# 查看状态
openclaw status

# 查看帮助
openclaw --help
```

### 9.2 网关管理

```bash
# 重启网关
openclaw gateway restart

# 停止网关
openclaw gateway stop

# 查看网关状态
openclaw gateway status

# 查看网关日志
openclaw gateway logs
```

### 9.3 配置管理

```bash
# 查看配置
openclaw config get gateway.auth.token

# 运行 doctor 检查和修复
openclaw doctor --fix

# 安全审计
openclaw security audit --deep
openclaw security audit --fix
```

### 9.4 代理和 PATH 管理（自定义函数）

```bash
# 显示当前代理设置
show_proxy

# 重新设置代理
setup_proxy

# 取消代理
unset_proxy

# 测试代理是否工作
curl -s --connect-timeout 3 https://github.com > /dev/null && echo "✓ 代理工作正常" || echo "✗ 代理可能有问题"
```

## 第十部分：故障排除

### 10.1 代理不工作

```bash
# 检查 host.docker.internal 是否可 ping
ping -c 1 host.docker.internal

# 检查 Windows 防火墙设置
# 确保代理软件（如 clash、v2ray）开启了"允许局域网连接"

# 测试直接连接
telnet host.docker.internal 10808
# 或
nc -zv host.docker.internal 10808
```

### 10.2 openclaw 命令找不到

```bash
# 检查 PATH
echo $PATH | grep npm-global

# 手动添加 PATH
export PATH="/home/liangrong/.npm-global/bin:$PATH"

# 永久修复
echo 'export PATH="/home/liangrong/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 10.3 配置文件错误

```bash
# 运行 doctor 修复
openclaw doctor --fix

# 查看配置文件内容
cat ~/.openclaw/openclaw.json

# 备份后手动编辑（如果必要）
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
nano ~/.openclaw/openclaw.json
```

### 10.4 Telegram 机器人无响应

```bash
# 检查网关状态
openclaw gateway status

# 重新批准配对
openclaw pairing list # 查看待批准的配对
openclaw pairing approve telegram [配对码]

# 检查 Telegram 配置
openclaw config get channels.telegram
```

## 附录：环境检查清单

### Windows 端检查
- [ ] 代理软件已开启
- [ ] 代理软件允许局域网连接
- [ ] 代理端口正确（10808）
- [ ] PowerShell 中代理测试成功

### WSL 端检查
- [ ] Ubuntu 22.04 已安装
- [ ] 代理环境变量已设置
- [ ] curl 测试 GitHub 成功
- [ ] npm 全局 bin 在 PATH 中
- [ ] openclaw 命令可用

### OpenClaw 检查
- [ ] OpenAI API key 已配置
- [ ] Telegram bot token 已配置
- [ ] 网关运行正常
- [ ] Telegram 配对已完成
- [ ] 控制面板可访问

---

**🎉 安装完成！**

现在你可以：
1. 通过 Telegram 与你的 AI 助手对话
2. 访问 Web 控制面板管理 OpenClaw
3. 根据需要配置更多功能和渠道