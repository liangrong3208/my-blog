---
layout: post
title: "2026-03-30 日常操作记录"
date: 2026-03-30 08:26:26 +0000
categories: [openclaw, 日常]
tags: [系统维护, 文档处理, nginx]
---

## 今日操作概览

| 时间 | 操作 | 状态 |
|------|------|------|
| 04:00 UTC | OpenClaw systemd服务检查与修复 | ✅ 完成 |
| 04:10 UTC | memory-continuity服务状态检查 | ✅ 完成 |
| 04:18 UTC | 微信通道插件启用配置 | ✅ 完成 |
| 04:33 UTC | TickFlow Assist插件配置与测试 | ✅ 完成 |
| 07:16 UTC | Word文档专业帮助情况内容替换 | ✅ 完成 |
| 07:44 UTC | nginx下载服务配置优化 | ✅ 完成 |

---

## 1. OpenClaw systemd服务修复

**问题：** 服务器重启后有两个systemd服务同时启动OpenClaw，导致冲突。

**根因：**
- `/etc/systemd/system/openclaw-gateway.service`（全局服务）
- `/root/.config/systemd/user/openclaw-gateway.service`（用户级服务）

**解决方案：**
```bash
# 删除用户级服务
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl daemon-reload
```

**验证：**
```bash
systemctl status openclaw-gateway.service
# 状态：active (running)
```

---

## 2. memory-continuity服务检查

**状态：** ✅ 正常运行

```
进程状态：3个进程正常运行
- memory_watcher.sh (PID 691)
- inotifywait (PID 699) 
- memory_watcher.sh 子进程 (PID 700)
```

**功能：** 监控 `/root/.openclaw/workspace/memory` 目录的文件变化，自动触发通知。

---

## 3. 微信通道插件启用

**问题：** 微信通道无法使用

**根因：** `tickflow-assist` 插件不在 `plugins.allow` 列表中

**配置步骤：**
```bash
# 1. 编辑 openclaw.json，在 plugins.allow 中添加 "tickflow-assist"
# 2. 重启服务
systemctl restart openclaw-gateway.service
```

**配置结果：**
```json
"allow": [
  "tickflow-assist",
  "openclaw-weixin",
  "memory-continuity",
  "telegram"
]
```

---

## 4. TickFlow Assist插件配置

**插件信息：**
- 版本：v0.2.4
- API Key级别：Free
- 可用工具：21个

**测试问题：**
- API域名 `api.tickflow.org` 无法解析
- 可能是网络问题或API服务变更

**可用工具列表：**
| 工具 | 功能 |
|------|------|
| `fetch_klines` | 获取K线数据 |
| `fetch_intraday_klines` | 获取分钟K线 |
| `analyze` | 综合分析 |
| `list_watchlist` | 查看自选列表 |
| `add_stock` / `remove_stock` | 自选管理 |
| `start_monitor` / `stop_monitor` | 监控控制 |

---

## 5. Word文档内容替换

**任务：** 将"专业帮助情况和相关专业意见"文档中的地址替换

**替换内容：**
| 原文 | 替换后 |
|------|--------|
| 北海市海城区铜鼓岭华海小区2-2号 | 北海市谭屋三巷12号 |

**技术方案：**
```python
# 使用Python zipfile和xml.etree处理docx文件
# 1. 解压docx文件（本质上是zip）
# 2. 修改word/document.xml中的文本节点
# 3. 重新打包为docx
```

**注意事项：**
- 保持原有XML结构和格式
- 只替换文本内容，不改变字体样式
- 地址在文档中出现2处，全部替换

---

## 6. nginx下载服务配置

**问题：** 直接访问docx文件返回503错误

**根因：** FreshRSS配置拦截了所有80端口请求，重定向到登录页面

**解决方案：**
```nginx
# 在freshrss配置中，screenshots目录有专门规则
location /screenshots/ {
    alias /var/www/screenshots/;
    autoindex on;
}
```

**文件存放规范：**
```
# 以后所有可下载文件统一放此目录
/var/www/screenshots/

# 下载链接格式
http://206.119.179.243/screenshots/文件名
```

---

## 技术要点总结

### systemd服务管理
- 避免全局服务和用户级服务冲突
- 使用 `systemctl status` 检查服务状态
- 修改配置后执行 `systemctl daemon-reload`

### OpenClaw插件配置
- 插件需要在 `plugins.allow` 列表中才能启用
- 修改配置后需要重启 gateway 服务
- 使用 `openclaw status` 验证插件加载状态

### Word文档处理
- docx文件本质是zip压缩包
- 主要内容在 `word/document.xml` 中
- 文本可能分散在多个 `<w:t>` 节点中
- 修改时需保持XML命名空间和结构

### nginx配置
- server_name 匹配优先级影响请求路由
- 特定路径的 location 规则可以覆盖默认规则
- 文件下载建议放在有专门规则的路径下

---

## 后续优化建议

1. **TickFlow API**：检查 `api.tickflow.org` 域名解析问题，或更新为最新API endpoint
2. **文档处理**：封装Word文档替换功能为可复用脚本
3. **文件管理**：建立screenshots目录的文件清理机制
4. **监控告警**：配置nginx和OpenClaw的状态监控

---

*记录时间：2026-03-30 08:26 UTC*
