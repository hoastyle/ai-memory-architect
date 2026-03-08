# 基于远程部署 QMD 的完整记忆系统构建指南

**版本**: 1.0  
**创建时间**: 2026-03-08  
**适用场景**: 从零开始构建基于远程 QMD 的记忆系统

---

## 概述

本文档提供完整的步骤，从零开始构建一个基于远程部署 QMD 的记忆系统。

### 系统架构

```
┌─────────────────────────────────────────────────┐
│  本地环境（OpenClaw）                            │
│  - Ubuntu/Linux                                  │
│  - OpenClaw Gateway                              │
│  - workspace 文件                                │
└─────────────────────────────────────────────────┘
                    ↓ SSH + rsync
┌─────────────────────────────────────────────────┐
│  远程服务器（QMD）                               │
│  - Ubuntu/Linux                                  │
│  - NVIDIA GPU (RTX 5080)                         │
│  - QMD 服务                                      │
│  - 索引数据库                                    │
└─────────────────────────────────────────────────┘
```

### 前置条件

**本地环境**：
- Ubuntu/Linux 系统
- OpenClaw 已安装
- SSH 客户端
- rsync 工具

**远程服务器**：
- Ubuntu/Linux 系统
- NVIDIA GPU（推荐 RTX 系列）
- CUDA 支持
- 至少 20GB 磁盘空间

---

## 第一部分：远程服务器准备

### 1.1 服务器基础配置

**登录远程服务器**：
```bash
ssh user@remote-server-ip
```

**更新系统**：
```bash
sudo apt update
sudo apt upgrade -y
```

**安装必要工具**：
```bash
sudo apt install -y curl git build-essential
```

### 1.2 安装 Bun（QMD 运行时）

```bash
# 安装 Bun
curl -fsSL https://bun.sh/install | bash

# 添加到 PATH
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 验证安装
bun --version
```

### 1.3 安装 QMD

```bash
# 全局安装 QMD
bun install -g @tobilu/qmd

# 验证安装
qmd --version
```

### 1.4 配置代理（如果需要）

如果需要访问 Hugging Face 下载模型：

```bash
# 设置代理
export HTTP_PROXY=http://proxy-server:port
export HTTPS_PROXY=http://proxy-server:port

# 添加到 ~/.bashrc
echo 'export HTTP_PROXY=http://proxy-server:port' >> ~/.bashrc
echo 'export HTTPS_PROXY=http://proxy-server:port' >> ~/.bashrc
```

### 1.5 创建工作目录

```bash
# 创建 QMD 工作目录
mkdir -p ~/Workspace/OpenClaw/qmd-docker
cd ~/Workspace/OpenClaw/qmd-docker

# 创建子目录
mkdir -p workspace memory
```

---

## 第二部分：本地环境配置

### 2.1 配置 SSH 免密登录

**生成 SSH 密钥**（如果没有）：
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**复制公钥到远程服务器**：
```bash
ssh-copy-id user@remote-server-ip
```

**测试免密登录**：
```bash
ssh user@remote-server-ip
# 应该无需密码直接登录
```

### 2.2 安装 rsync

```bash
# Ubuntu/Debian
sudo apt install rsync

# 验证安装
rsync --version
```

### 2.3 创建同步脚本

**文件位置**：`~/.openclaw/workspace/scripts/sync-to-qmd.sh`

```bash
#!/bin/bash
# 同步本地文件到远程 QMD 服务器

set -euo pipefail

# 配置
REMOTE_USER="user"
REMOTE_HOST="remote-server-ip"
REMOTE_BASE="/home/$REMOTE_USER/Workspace/OpenClaw/qmd-docker"
LOCAL_WORKSPACE="$HOME/.openclaw/workspace"

echo "=== 同步文件到 QMD 服务器 ==="

# 1. 创建远程目录结构
ssh "$REMOTE_USER@$REMOTE_HOST" "mkdir -p $REMOTE_BASE/workspace $REMOTE_BASE/memory"

# 2. 同步 workspace 文件（排除敏感信息）
echo "同步 workspace..."
rsync -avz --delete \
  --exclude='credentials/' \
  --exclude='.git/' \
  --exclude='node_modules/' \
  --exclude='*.log' \
  "$LOCAL_WORKSPACE/" \
  "$REMOTE_USER@$REMOTE_HOST:$REMOTE_BASE/workspace/"

# 3. 同步 memory 文件
echo "同步 memory..."
rsync -avz --delete \
  "$LOCAL_WORKSPACE/memory/" \
  "$REMOTE_USER@$REMOTE_HOST:$REMOTE_BASE/memory/"

# 4. 统计同步结果
WORKSPACE_COUNT=$(ssh "$REMOTE_USER@$REMOTE_HOST" "find $REMOTE_BASE/workspace -name '*.md' | wc -l")
MEMORY_COUNT=$(ssh "$REMOTE_USER@$REMOTE_HOST" "find $REMOTE_BASE/memory -name '*.md' | wc -l")

echo "✅ 同步完成"
echo "  Workspace: $WORKSPACE_COUNT 个 .md 文件"
echo "  Memory: $MEMORY_COUNT 个 .md 文件"
echo "  总计: $((WORKSPACE_COUNT + MEMORY_COUNT)) 个文件"
```

**添加执行权限**：
```bash
chmod +x ~/.openclaw/workspace/scripts/sync-to-qmd.sh
```

**测试同步**：
```bash
~/.openclaw/workspace/scripts/sync-to-qmd.sh
```

---

## 第三部分：QMD 配置

### 3.1 创建 Collections

**登录远程服务器**：
```bash
ssh user@remote-server-ip
cd ~/Workspace/OpenClaw/qmd-docker
```

**创建 workspace collection**：
```bash
qmd collection add workspace workspace
```

**创建 memory collection**：
```bash
qmd collection add memory memory
```

**验证 collections**：
```bash
qmd ls workspace
qmd ls memory
```

### 3.2 生成 Embeddings

**开始索引**：
```bash
qmd embed
```

这个过程会：
1. 下载 Embedding 模型（314MB）
2. 下载 Reranker 模型（328MB）
3. 下载 Generation 模型（1.28GB）
4. 生成向量索引

**预计时间**：30-60 分钟（取决于网络速度和文件数量）

**检查状态**：
```bash
qmd status
```

### 3.3 验证功能

**测试全文搜索**：
```bash
qmd search "测试" -n 3
```

**测试语义搜索**（等待模型下载完成）：
```bash
qmd vsearch "测试查询" -n 3
```

**测试混合搜索**（等待模型下载完成）：
```bash
qmd query "如何测试" -n 3
```

---

## 第四部分：OpenClaw 集成

### 4.1 创建 QMD Wrapper Script

**文件位置**：`~/.openclaw/workspace/scripts/qmd-memory-search.sh`

**内容**：见 [qmd-integration-guide.md](qmd-integration-guide.md) 中的完整脚本

**关键功能**：
- 接收查询参数
- 调用远程 QMD（通过 SSH）
- 转换输出格式为 OpenClaw 兼容
- 智能路由（自动选择搜索模式）
- 降级机制

**添加执行权限**：
```bash
chmod +x ~/.openclaw/workspace/scripts/qmd-memory-search.sh
```

### 4.2 创建别名

**添加到 ~/.bashrc**：
```bash
echo 'alias qmd-search="$HOME/.openclaw/workspace/scripts/qmd-memory-search.sh"' >> ~/.bashrc
source ~/.bashrc
```

### 4.3 测试集成

**基本测试**：
```bash
qmd-search "测试" 3 0.5 auto
```

**完整测试**：
```bash
~/.openclaw/workspace/scripts/test-qmd-complete.sh
```

---

## 第五部分：自动化配置

### 5.1 配置定时同步

**编辑 crontab**：
```bash
crontab -e
```

**添加定时任务**：
```bash
# 每天 8:00 UTC 同步文件
0 8 * * * /home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh >> /tmp/qmd-sync.log 2>&1

# 每小时同步一次（可选）
0 * * * * /home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh >> /tmp/qmd-sync.log 2>&1
```

### 5.2 创建每日同步脚本

**文件位置**：`~/.openclaw/workspace/scripts/daily-sync.sh`

```bash
#!/bin/bash
# 每日信息收集和同步

set -euo pipefail

TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
echo "=== CoR 信息收集 - $TIMESTAMP ==="

# 1. 同步文件到 QMD
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh

# 2. 更新 QMD 索引
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd update"

echo "✅ 同步成功"
```

**添加执行权限**：
```bash
chmod +x ~/.openclaw/workspace/scripts/daily-sync.sh
```

---

## 第六部分：验证和测试

### 6.1 端到端测试

**测试 1：文件同步**
```bash
# 创建测试文件
echo "# 测试文档" > ~/.openclaw/workspace/test.md

# 同步到远程
~/.openclaw/workspace/scripts/sync-to-qmd.sh

# 验证远程文件
ssh user@remote-server-ip "cat ~/Workspace/OpenClaw/qmd-docker/workspace/test.md"
```

**测试 2：QMD 搜索**
```bash
# 更新索引
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd update"

# 搜索测试文档
qmd-search "测试文档" 3 0.5 auto
```

**测试 3：完整流程**
```bash
# 运行完整测试套件
~/.openclaw/workspace/scripts/test-qmd-complete.sh
```

### 6.2 性能测试

**响应时间测试**：
```bash
time qmd-search "测试查询" 5 0.6 search   # 应该 < 1s
time qmd-search "测试查询" 5 0.6 vsearch  # 应该 < 3s
time qmd-search "测试查询" 5 0.4 query    # 应该 < 5s
```

**准确率测试**：
```bash
# 搜索已知信息
qmd-search "王永锋" 5 0.6 auto
# 验证结果是否包含正确信息
```

---

## 第七部分：维护和监控

### 7.1 日常维护

**每日检查**：
```bash
# 检查同步日志
tail -f /tmp/qmd-sync.log

# 检查 QMD 状态
ssh user@remote-server-ip "qmd status"
```

**每周维护**：
```bash
# 清理旧日志
rm -f /tmp/qmd-sync.log

# 重新索引（如果需要）
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd update --force"
```

### 7.2 监控指标

**关键指标**：
- 同步成功率：应该 > 99%
- 搜索响应时间：< 5s
- 索引文件数：应该与本地一致
- GPU 使用率：正常情况下 < 50%

**检查命令**：
```bash
# 检查索引状态
ssh user@remote-server-ip "qmd status"

# 检查 GPU 状态
ssh user@remote-server-ip "nvidia-smi"

# 检查磁盘空间
ssh user@remote-server-ip "df -h"
```

### 7.3 故障排查

**问题 1：同步失败**
```bash
# 检查 SSH 连接
ssh user@remote-server-ip "echo 'SSH OK'"

# 检查 rsync
rsync --version

# 手动同步测试
rsync -avz --dry-run ~/.openclaw/workspace/ user@remote-server-ip:~/test/
```

**问题 2：搜索无结果**
```bash
# 检查索引状态
ssh user@remote-server-ip "qmd status"

# 重新索引
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd update --force"

# 检查文件是否存在
ssh user@remote-server-ip "ls -la ~/Workspace/OpenClaw/qmd-docker/workspace/"
```

**问题 3：响应时间过长**
```bash
# 检查 GPU 状态
ssh user@remote-server-ip "nvidia-smi"

# 检查网络延迟
ping remote-server-ip

# 检查 QMD 进程
ssh user@remote-server-ip "ps aux | grep qmd"
```

---

## 第八部分：安全配置

### 8.1 敏感信息保护

**排除敏感目录**：
```bash
# 在 sync-to-qmd.sh 中已配置
--exclude='credentials/'
--exclude='.git/'
--exclude='node_modules/'
--exclude='*.log'
```

**验证排除**：
```bash
# 检查远程是否有敏感文件
ssh user@remote-server-ip "find ~/Workspace/OpenClaw/qmd-docker -name 'credentials' -o -name '.git'"
# 应该返回空
```

### 8.2 访问控制

**SSH 密钥管理**：
```bash
# 使用强密钥
ssh-keygen -t ed25519 -C "your_email@example.com"

# 限制密钥权限
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

**防火墙配置**（远程服务器）：
```bash
# 只允许特定 IP 访问
sudo ufw allow from your-local-ip to any port 22
sudo ufw enable
```

### 8.3 数据备份

**备份 QMD 索引**：
```bash
# 在远程服务器上
cd ~/Workspace/OpenClaw/qmd-docker
tar -czf qmd-backup-$(date +%Y%m%d).tar.gz workspace memory ~/.cache/qmd/

# 下载到本地
scp user@remote-server-ip:~/Workspace/OpenClaw/qmd-docker/qmd-backup-*.tar.gz ~/backups/
```

---

## 第九部分：性能优化

### 9.1 网络优化

**使用压缩传输**：
```bash
# rsync 已启用压缩（-z 参数）
rsync -avz ...
```

**增量同步**：
```bash
# rsync 自动使用增量同步
# 只传输变化的文件
```

### 9.2 索引优化

**定期清理**：
```bash
# 清理旧索引
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd clean"

# 重新索引
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd update --force"
```

**批量索引**：
```bash
# 一次性索引多个文件
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd embed --batch"
```

### 9.3 缓存优化

**本地缓存**（可选）：
```bash
# 缓存最近的搜索结果
mkdir -p ~/.cache/qmd-search/

# 在 wrapper script 中添加缓存逻辑
# 缓存 TTL: 5 分钟
```

---

## 第十部分：升级和迁移

### 10.1 升级 QMD

```bash
# 在远程服务器上
ssh user@remote-server-ip

# 升级 QMD
bun update -g @tobilu/qmd

# 验证版本
qmd --version

# 重新索引（如果需要）
cd ~/Workspace/OpenClaw/qmd-docker
qmd update --force
```

### 10.2 迁移到新服务器

**步骤**：
1. 在新服务器上重复"第一部分"的配置
2. 复制索引数据：
   ```bash
   # 从旧服务器
   ssh old-server "tar -czf qmd-data.tar.gz ~/Workspace/OpenClaw/qmd-docker ~/.cache/qmd/"
   
   # 传输到新服务器
   scp old-server:~/qmd-data.tar.gz new-server:~/
   
   # 在新服务器上解压
   ssh new-server "tar -xzf qmd-data.tar.gz"
   ```
3. 更新本地配置（sync-to-qmd.sh 中的服务器地址）
4. 测试验证

---

## 附录

### A. 完整配置清单

**远程服务器**：
- [ ] Ubuntu/Linux 系统
- [ ] NVIDIA GPU + CUDA
- [ ] Bun 运行时
- [ ] QMD 安装
- [ ] 工作目录创建
- [ ] Collections 配置
- [ ] Embeddings 生成

**本地环境**：
- [ ] SSH 免密登录
- [ ] rsync 安装
- [ ] 同步脚本创建
- [ ] Wrapper 脚本创建
- [ ] 别名配置
- [ ] 定时任务配置

**验证**：
- [ ] 文件同步测试
- [ ] QMD 搜索测试
- [ ] 完整流程测试
- [ ] 性能测试

### B. 常用命令速查

```bash
# 同步文件
~/.openclaw/workspace/scripts/sync-to-qmd.sh

# 更新索引
ssh user@remote-server-ip "cd ~/Workspace/OpenClaw/qmd-docker && qmd update"

# 检查状态
ssh user@remote-server-ip "qmd status"

# 搜索
qmd-search "查询" 5 0.6 auto

# 测试
~/.openclaw/workspace/scripts/test-qmd-complete.sh

# 查看日志
tail -f /tmp/qmd-sync.log
```

### C. 参考文档

- **使用指南**: [qmd-memory-system-guide.md](qmd-memory-system-guide.md)
- **集成指南**: [qmd-integration-guide.md](qmd-integration-guide.md)
- **架构说明**: [memory-system-with-qmd.md](memory-system-with-qmd.md)

---

**版本**: 1.0  
**最后更新**: 2026-03-08  
**状态**: 生产就绪 ✅
