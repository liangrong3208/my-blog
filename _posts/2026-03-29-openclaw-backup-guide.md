---
layout: post
title: "OpenClaw 完整备份指南"
date: 2026-03-29 10:50:00 +0000
categories: [openclaw]
tags: [备份, 教程, OpenClaw]
---

## 概述

本文介绍如何创建 OpenClaw 的完整备份，包括配置和工作目录。

---

## 📦 完整备份信息

| 项目 | 内容 |
|------|------|
| 备份文件 | `/root/2026-03-29T10-02-openclaw-backup-complete.tar.gz` |
| 文件大小 | 235 MB |
| 包含内容 | `~/.openclaw` + `~/.openclaw/workspace` |
| 说明 | 完整备份（配置 + 工作目录） |

---

## 💻 下载完整备份到本地

### 方法1：SCP 命令（推荐）

```bash
# 下载完整备份（包含 workspace）
scp root@206.119.179.243:/root/2026-03-29T10-02-openclaw-backup-complete.tar.gz ./

# 或者指定本地保存路径
scp root@206.119.179.243:/root/2026-03-29T10-02-openclaw-backup-complete.tar.gz ~/Downloads/
```

### 方法2：SFTP 命令

```bash
sftp root@206.119.179.243
get /root/2026-03-29T10-02-openclaw-backup-complete.tar.gz
bye
```

### 方法3：使用 rsync

```bash
rsync -avz --progress root@206.119.179.243:/root/2026-03-29T10-02-openclaw-backup-complete.tar.gz ./
```

---

## 📋 备份文件对比

| 备份文件 | 大小 | 包含内容 |
|----------|------|----------|
| `2026-03-29T09-57-08.350Z-openclaw-backup.tar.gz` | 235 MB | `~/.openclaw`（不含 workspace） |
| `2026-03-29T10-02-openclaw-backup-complete.tar.gz` | 235 MB | `~/.openclaw` + workspace ✅ |

---

## 🔐 恢复备份

```bash
# 解压完整备份
tar -xzf 2026-03-29T10-02-openclaw-backup-complete.tar.gz -C /

# 或者解压到指定目录
mkdir -p ~/openclaw-restore
tar -xzf 2026-03-29T10-02-openclaw-backup-complete.tar.gz -C ~/openclaw-restore
```

---

## 总结

完整备份已创建！现在包含 workspace 工作目录，请使用 SCP 命令下载到本地。

---

*发布时间：2026-03-29 10:50 UTC*
