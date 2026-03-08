# 个人版记忆系统 - 完整部署指南

**版本**: v2.0
**日期**: 2026-03-08
**状态**: 生产就绪

---

## 📋 部署概览

本文档提供 supersystem 个人版记忆系统的完整部署指南，包括：
- 独立部署（纯命令行使用）
- OpenClaw 集成部署
- Slack 集成部署

### 推荐部署方式

| 场景 | 部署方式 | 复杂度 |
|------|---------|--------|
| 个人命令行使用 | 独立部署 | ⭐ |
| OpenClaw 用户 | OpenClaw 集成 | ⭐⭐ |
| 团队协作 | Slack 集成 | ⭐⭐⭐ |

---

## 🎯 方式一：独立部署（纯命令行）

### 前置要求

- Python 3.8+
- Git
- 2GB 可用内存
- 500MB 磁盘空间

### 安装步骤

#### 1. 克隆仓库

```bash
cd ~
git clone https://github.com/ktao732084-arch/openclaw_memory_supersystem-v1.0.git
cd openclaw_memory_supersystem-v1.0
```

#### 2. 安装依赖

```bash
pip3 install -r requirements.txt
```

#### 3. 初始化数据库

```bash
python3 src/memory.py init
```

**预期输出**：
```
✓ 数据库初始化成功
✓ 创建目录: memory/
✓ 创建数据库: memory/memory.db
```

#### 4. 验证安装

```bash
# 查看状态
python3 src/memory.py status

# 添加测试记忆
python3 src/memory.py capture "测试记忆内容" --type fact

# 搜索记忆
python3 src/memory.py search "测试"
```

### 常用命令

```bash
# 添加记忆
python3 src/memory.py capture "内容" --type fact|belief|summary

# 搜索记忆
python3 src/memory.py search "关键词" --limit 10

# 查看状态
python3 src/memory.py status

# 记忆整合（7阶段）
python3 src/memory.py consolidate

# 查看帮助
python3 src/memory.py --help
```

---

## 🎯 方式二：OpenClaw 集成部署

### 前置要求

- 已安装 OpenClaw
- SSH 访问目标机器
- 完成"方式一"的基础安装

### 集成步骤

#### 1. 创建桥接脚本

```bash
cd ~/openclaw_memory_supersystem-v1.0
cat > memory_bridge.sh << 'EOF'
#!/bin/bash
MEMORY_DIR="/home/admin/openclaw_memory_supersystem-v1.0"
cd "$MEMORY_DIR"
python3 src/memory.py "$@"
EOF

chmod +x memory_bridge.sh
```

#### 2. 创建 OpenClaw Skill

```bash
mkdir -p ~/.openclaw/skills
cat > ~/.openclaw/skills/supersystem-memory.md << 'EOF'
---
name: supersystem-memory
description: 长期记忆管理系统
version: 1.0.0
---

# Supersystem 记忆管理

## 命令

### /memory-add
添加新记忆到系统

**用法**：
```
/memory-add <内容> <类型>
```

**类型**：
- `fact` - 确定的事实
- `belief` - 不确定的推断
- `summary` - 总结性知识

**示例**：
```
/memory-add 用户喜欢喝咖啡 fact
/memory-add 用户可能住在北京 belief
```

**实现**：
```bash
~/openclaw_memory_supersystem-v1.0/memory_bridge.sh capture "$1" --type "$2"
```

### /memory-search
搜索历史记忆

**用法**：
```
/memory-search <关键词> [数量]
```

**示例**：
```
/memory-search 咖啡
/memory-search 北京 5
```

**实现**：
```bash
~/openclaw_memory_supersystem-v1.0/memory_bridge.sh search "$1" --limit "${2:-10}"
```

### /memory-status
查看记忆系统状态

**用法**：
```
/memory-status
```

**实现**：
```bash
~/openclaw_memory_supersystem-v1.0/memory_bridge.sh status
```

## 自动捕获

系统会自动捕获包含以下关键词的对话：
- "记住"、"记录"
- "喜欢"、"不喜欢"
- "重要"、"关键"

配置文件：`~/.openclaw/workspace/memory_auto_config.json`
EOF
```

#### 3. 测试集成

在 OpenClaw 中测试：

```
/memory-add 测试记忆 fact
/memory-search 测试
/memory-status
```

---

## 🎯 方式三：Slack 集成部署

### 前置要求

- 已完成"方式二"OpenClaw 集成
- Slack Workspace 管理员权限
- OpenClaw 已配置 Slack Socket Mode

### 集成步骤

#### 1. 配置 Slack App

1. 访问 https://api.slack.com/apps
2. 选择你的 OpenClaw App
3. 启用 Socket Mode
4. 添加 Bot Token Scopes：
   - `chat:write`
   - `commands`
   - `app_mentions:read`

#### 2. 使用 @mention 命令

**推荐方式**（无需额外配置）：

```
@bot /memory-add 用户喜欢喝咖啡 fact
@bot /memory-search 咖啡
@bot /memory-status
```

#### 3. 自动捕获配置

创建自动捕获配置：

```bash
cat > ~/.openclaw/workspace/memory_auto_config.json << 'EOF'
{
  "enabled": true,
  "triggers": [
    "记住",
    "记录",
    "喜欢",
    "不喜欢",
    "重要",
    "关键"
  ],
  "default_type": "fact"
}
EOF
```

#### 4. 验证集成

在 Slack 中测试：

1. 发送：`@bot /memory-add 测试Slack集成 fact`
2. 发送：`@bot /memory-search 测试`
3. 发送：`记住：用户喜欢喝咖啡`（自动捕获）

---

## 🔧 故障排查

### 问题 1：命令未找到

**症状**：
```
python3: command not found
```

**解决**：
```bash
# 检查 Python 版本
python --version
python3 --version

# 如果没有 python3，创建软链接
sudo ln -s /usr/bin/python /usr/bin/python3
```

### 问题 2：数据库初始化失败

**症状**：
```
Error: Unable to create database
```

**解决**：
```bash
# 检查目录权限
ls -la memory/

# 修复权限
chmod 755 memory/
chmod 644 memory/memory.db
```

### 问题 3：搜索无结果

**症状**：
```
No memories found
```

**原因**：
- 记忆在 Layer 1（活跃池），搜索默认在 Layer 2（长期记忆）

**解决**：
```bash
# 方式 1：等待自动整合（每 24 小时）
# 方式 2：手动触发整合
python3 src/memory.py consolidate

# 方式 3：直接搜索 Layer 1
python3 src/memory.py search "关键词" --layer 1
```

### 问题 4：Slack 命令失败

**症状**：
```
dispatch_failed
```

**原因**：
- OpenClaw 不支持原生 Slack Slash Commands
- OpenClaw 不支持 Message Shortcuts

**解决**：
使用 @mention 命令代替：
```
@bot /memory-add 内容 fact
```

### 问题 5：Gateway 连接失败

**症状**：
```
Connection refused to localhost:18789
```

**解决**：
```bash
# 检查 OpenClaw 配置
cat ~/.openclaw/openclaw.json | grep -A 5 gateway

# 确认 bind 设置为 "loopback"
# 确认 port 为 18789

# 重启 OpenClaw
pkill -f openclaw
openclaw start
```

---

## 📊 性能优化

### 1. 定期整合

设置 cron 任务自动整合记忆：

```bash
# 每天凌晨 2 点执行整合
crontab -e

# 添加以下行
0 2 * * * cd ~/openclaw_memory_supersystem-v1.0 && python3 src/memory.py consolidate
```

### 2. 数据库优化

```bash
# 每周执行一次
python3 src/memory.py optimize
```

### 3. 清理旧记忆

```bash
# 删除 90 天前的低置信度记忆
python3 src/memory.py cleanup --days 90 --confidence 0.3
```

---

## 📈 监控和维护

### 日常检查

```bash
# 查看系统状态
python3 src/memory.py status

# 查看最近记忆
python3 src/memory.py list --limit 10

# 查看统计信息
python3 src/memory.py stats
```

### 备份

```bash
# 备份数据库
cp memory/memory.db memory/memory.db.backup.$(date +%Y%m%d)

# 自动备份脚本
cat > ~/backup_memory.sh << 'EOF'
#!/bin/bash
BACKUP_DIR=~/memory_backups
mkdir -p $BACKUP_DIR
cp ~/openclaw_memory_supersystem-v1.0/memory/memory.db \
   $BACKUP_DIR/memory.db.$(date +%Y%m%d_%H%M%S)
# 保留最近 7 天的备份
find $BACKUP_DIR -name "memory.db.*" -mtime +7 -delete
EOF

chmod +x ~/backup_memory.sh

# 添加到 crontab（每天备份）
0 3 * * * ~/backup_memory.sh
```

---

## 🎓 最佳实践

### 1. 记忆分类

- **Fact**：确定的事实（用户偏好、历史事件）
- **Belief**：不确定的推断（可能的意图、假设）
- **Summary**：总结性知识（项目总结、经验教训）

### 2. 搜索技巧

```bash
# 精确搜索
python3 src/memory.py search "咖啡" --exact

# 模糊搜索
python3 src/memory.py search "咖" --fuzzy

# 时间范围搜索
python3 src/memory.py search "项目" --after 2026-01-01

# 按类型搜索
python3 src/memory.py search "用户" --type fact
```

### 3. 记忆质量

- 添加记忆时包含上下文
- 使用清晰的描述
- 定期审查和更新记忆
- 删除过时或错误的记忆

---

## 📚 相关文档

- [个人版方案详细设计](personal.md)
- [supersystem 架构说明](../../reference/supersystem/README.md)
- [参考方案对比](../../analysis/01-reference-analysis.md)

---

## 🆘 获取帮助

### 常见问题

查看 [FAQ](../../reference/supersystem/FAQ.md)

### 社区支持

- GitHub Issues: https://github.com/ktao732084-arch/openclaw_memory_supersystem-v1.0/issues
- 文档反馈: 在本仓库提交 Issue

---

**部署完成时间**: 2026-03-08
**验证状态**: ✅ 生产环境验证通过
**维护者**: Howie
