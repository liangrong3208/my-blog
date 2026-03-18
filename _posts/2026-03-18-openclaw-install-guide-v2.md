---
layout: post
title: "OpenClaw 完整安装命令手册第二版"
date: 2026-03-18 09:12:09
categories: [tutorial, openclaw, installation]
tags: [windows, wsl2, ubuntu, proxy, setup]
---

# OpenClaw 完整安装配置指南（Windows + WSL2）

## 第一部分：Windows 环境配置

### 1.1 在 Windows PowerShell 中设置代理

```powershell
# 打开 PowerShell（以管理员身份运行）

# 设置代理环境变量
$env:HTTP_PROXY = "http://127.0.0.1:10808"
$env:HTTPS_PROXY = "http://127.0.0.1:10808"

# 验证代理设置
Write-Host "HTTP_PROXY = $env:HTTP_PROXY" -ForegroundColor Green
Write-Host "HTTPS_PROXY = $env:HTTPS_PROXY" -ForegroundColor Green

# 测试代理是否工作
try {
 $response = Invoke-WebRequest -Uri "https://www.google.com" -UseBasicParsing -TimeoutSec 10
 Write-Host "✅ 成功通过代理访问 Google！" -ForegroundColor Green
 Write-Host "状态码: $($response.StatusCode)"
} catch {
 Write-Host "❌ 访问失败: $_" -ForegroundColor Red
}
```

### 1.2 获取 Windows 本机 IP 地址（备用）

```powershell
# 查看本机 IP 地址
ipconfig | findstr "IPv4"
# 记录下你的 IPv4 地址，例如：192.168.1.100
```

### 1.3 安装 WSL2 和 Ubuntu

```powershell
# 设置 WSL2 为默认版本
wsl --set-default-version 2

# 安装 Ubuntu 22.04 LTS
wsl --install -d Ubuntu-22.04

# 等待安装完成，系统会提示创建用户名和密码
# 按照提示设置：
# Enter new UNIX username: liangrong
# New password: [设置你的密码]
# Retype new password: [再次输入密码]
```

## 第二部分：WSL Ubuntu 基础配置

### 2.1 首次进入 WSL Ubuntu

```bash
# 安装完成后会自动进入 Ubuntu，或者手动启动：
# 在 PowerShell 中执行：wsl ~

# 确认当前目录
pwd
# 应该显示：/home/liangrong
```

### 2.2 配置代理（两种方法）

#### 方法 A：使用 host.docker.internal（推荐）

```bash
# 测试 host.docker.internal 是否可用
ping -c 1 host.docker.internal

# 设置代理环境变量
export http_proxy="http://host.docker.internal:10808"
export https_proxy="http://host.docker.internal:10808"
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$https_proxy
export NO_PROXY="localhost,127.0.0.1,.local,::1"
```

#### 方法 B：使用固定 IP 地址

```bash
# 获取 Windows 主机的 IP 地址
HOST_IP=$(ip route | grep default | awk '{print $3}')
echo "Windows 主机 IP: $HOST_IP"

# 设置代理环境变量
export http_proxy="http://${HOST_IP}:10808"
export https_proxy="http://${HOST_IP}:10808"
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$https_proxy
export NO_PROXY="localhost,127.0.0.1,.local,::1"
```

### 2.3 测试代理是否正常工作

```bash
# 测试 GitHub 连接
curl -I https://github.com

# 测试 Google 连接
curl -I https://www.google.com

# 查看当前公网 IP（确认代理生效）
curl https://api.ipify.org?format=text

# 应该能看到返回的 IP 地址（不是你的本地 IP）
```

## 第三部分：安装 OpenClaw

### 3.1 执行安装命令

```bash
# 安装 OpenClaw（可能会要求输入 sudo 密码）
curl -fsSL https://openclaw.ai/install.sh | bash

# 安装过程中会：
# 1. 检测操作系统（linux）
# 2. 安装 Node.js（如果没有）
# 3. 安装构建工具（make/g++/cmake/python3）
# 4. 安装 OpenClaw npm 包
```

### 3.2 安装后的提示

安装完成后，系统会显示：
```
! PATH missing npm global bin dir: /home/liangrong/.npm-global/bin
 This can make openclaw show as "command not found" in new terminals.
 Fix (bash: ~/.bashrc):
 export PATH="/home/liangrong/.npm-global/bin:$PATH"
```

## 第四部分：修复 PATH 环境变量

### 4.1 临时修复（当前会话有效）

```bash
# 添加 npm 全局 bin 目录到 PATH
export PATH="/home/liangrong/.npm-global/bin:$PATH"

# 验证安装
openclaw --version
# 应该显示：OpenClaw 2026.3.13 (61d171a)
```

### 4.2 永久修复（添加到 .bashrc）

```bash
# 将 PATH 设置添加到 .bashrc
echo 'export PATH="/home/liangrong/.npm-global/bin:$PATH"' >> ~/.bashrc

# 使配置生效
source ~/.bashrc

# 验证
openclaw --version
```

## 第五部分：配置 OpenClaw

### 5.1 运行配置向导

```bash
openclaw onboard
```

配置选项：
- 模式选择：QuickStart
- 模型提供商：OpenAI
- API Key：输入你的 OpenAI API Key
- 默认模型：保持当前
- 频道选择：Telegram
- Telegram Token：输入你的 Bot Token
- 其他配置：按需选择

### 5.2 修复配置文件错误

```bash
# 运行 doctor 自动修复
openclaw doctor --fix
```

### 5.3 启动服务

```bash
# 重启网关
openclaw gateway restart

# 查看状态
openclaw gateway status
```

## 第六部分：验证安装

### 6.1 访问 Web 控制台

打开浏览器访问：
```
http://127.0.0.1:18789/#token=...
```

### 6.2 在 Telegram 中使用

1. 找到你的 Bot（如 @Liangrongbot）
2. 发送 `/start` 开始对话
3. 发送 `/pair` 获取配对码
4. 在 Ubuntu 中执行配对命令

## 常见问题

### Q1: 代理设置后仍然无法访问外网？
- 检查代理软件是否正常运行
- 确认代理端口是否正确（默认10808）
- 尝试更换代理方法（host.docker.internal 或 固定IP）

### Q2: openclaw 命令找不到？
- 确认 PATH 环境变量已正确设置
- 执行 `source ~/.bashrc` 使配置生效
- 检查 `~/.npm-global/bin` 目录是否存在

### Q3: 安装过程中断？
- 重新执行安装命令
- 确保网络连接稳定
- 检查代理是否正常工作

---

**安装完成！** 你现在拥有了一个功能完整的 OpenClaw AI 助手。