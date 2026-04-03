---
layout: post
title: "PM2 配置 OpenClaw 开机自启完整指南"
date: 2026-04-02 16:08:00 +0000
categories: [技术]
tags: [pm2, openclaw, 运维, 开机自启]
---

完美！PM2 开机自启已经配置成功。现在执行最后一步保存进程列表：

```bash
# 保存当前进程列表，确保重启后自动恢复
pm2 save
```

应该会看到类似输出：

```
[PM2] Saving current process list...
[PM2] Successfully saved in /root/.pm2/dump.pm2
```

## 验证服务状态

```bash
# 1. 查看完整日志，确认没有错误
pm2 logs openclaw --lines 50 --nostream
```

特别注意查看日志中是否有这些关键信息：

- Gateway started on port 18789
- Server listening
- 任何 ERROR 或 FATAL 信息

```bash
# 2. 确认端口监听状态
ss -tlnp | grep 18789
```

```bash
# 3. 查看 PM2 详细进程信息
pm2 show openclaw
```

## 测试重启功能（可选）

```bash
# 测试重启是否正常
pm2 restart openclaw
pm2 status
```

## 最终检查清单

- ✅ PM2 服务状态：online
- ✅ 开机自启：已配置 pm2-root.service
- ✅ 进程列表：已保存

## 如果一切正常

你现在可以：

1. **测试外部访问**：从另一台机器访问 `http://你的服务器IP:18789`
2. **查看实时日志**：`pm2 logs openclaw`（按 Ctrl+C 退出）
3. **监控资源**：`pm2 monit`

## 如果遇到连接问题

检查防火墙：

```bash
# 检查防火墙状态
ufw status

# 如果防火墙开启，需要开放 18789 端口
ufw allow 18789/tcp
```

---

把 `pm2 logs openclaw --lines 30 --nostream` 的输出发给我，我帮你确认服务是否完全正常启动了！
