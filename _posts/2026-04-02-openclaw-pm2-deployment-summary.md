---
layout: post
title: "OpenClaw PM2 部署完整总结"
date: 2026-04-02 16:09:00 +0000
categories: [技术]
tags: [openclaw, pm2, 部署, 运维, systemd]
---

🎉 完美！所有步骤都成功了！

## ✅ 最终状态确认

从输出可以看到：

- ✅ 服务正常运行在 **端口 18790**（PID 21306）
- ✅ PM2 配置已保存（`pm2 save`）
- ✅ 开机自启已配置（`pm2 startup`）
- ✅ 没有端口冲突
- ✅ 只有一个进程在运行

## 📋 服务访问信息

### Dashboard 访问地址

从之前的 onboarding 输出，你的 Dashboard 地址是：

```
http://你的服务器IP:18790/#token=437baf2144da397fe452577fcbac52c8b4f75f1585341d7a
```

### 获取服务器 IP

```bash
curl ifconfig.me
```

然后用浏览器访问：`http://<服务器IP>:18790/#token=<你的token>`

## 🔧 日常管理命令速查

```bash
# 查看服务状态
pm2 status

# 查看实时日志
pm2 logs openclaw-main

# 重启服务
pm2 restart openclaw-main

# 停止服务
pm2 stop openclaw-main

# 监控资源使用
pm2 monit

# 查看详细信息
pm2 show openclaw-main
```

## 🔥 防火墙配置（如需外部访问）

```bash
# 开放 18790 端口
ufw allow 18790/tcp

# 查看防火墙状态
ufw status
```

## 🧹 清理配置警告（可选）

```bash
# 删除 memory-continuity 配置警告
openclaw --profile main config delete plugins.entries.memory-continuity
```

## 🎯 验证多实例问题已解决

现在你可以在 Telegram 发送测试消息，然后监控进程数：

```bash
# 实时监控进程数量（应该始终是 1）
watch -n 2 'ps aux | grep "openclaw-gateway" | grep -v grep | wc -l'
```

---

## 📝 总结

### 问题根源

systemd user 服务在后台自动启动，与 PM2 实例冲突。

### 解决方案

1. 清理所有 systemd 服务文件
2. 改用 PM2 管理进程
3. 使用 `--profile main` 隔离配置
4. 更换端口到 18790 彻底避免冲突

### 当前状态

✅ 稳定运行，开机自启，不会有多实例问题

现在你的 OpenClaw 应该可以稳定运行了！有任何问题随时告诉我。
