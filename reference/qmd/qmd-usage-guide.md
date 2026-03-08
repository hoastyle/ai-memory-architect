# QMD 使用指南

## 当前可用功能

### ✅ 全文搜索（qmd search）
BM25 关键词搜索，速度快，无需模型。

**基本用法**：
```bash
# 搜索关键词
qmd search "研发负责人"

# 限制结果数量
qmd search "OC1 软件组" -n 5

# 多关键词搜索
qmd search "任务负载 王永锋"

# 在特定 collection 中搜索
qmd search "配置" -c workspace
```

**搜索结果包含**：
- 文档路径（qmd://workspace/...）
- 文档标题
- 相关性评分（Score: XX%）
- 匹配的文本片段

### ✅ 文件管理

**列出文件**：
```bash
# 列出 workspace collection 的所有文件
qmd ls workspace

# 列出 memory collection 的所有文件
qmd ls memory
```

**获取文档内容**：
```bash
# 获取完整文档
qmd get qmd://workspace/SOUL.md

# 获取特定行
qmd get qmd://workspace/SOUL.md:5 -l 10
```

**查看状态**：
```bash
# 查看 QMD 系统状态
qmd status
```

## ⏳ 即将可用功能

### 向量搜索（qmd vsearch）
语义搜索，理解查询意图。

**需要**：Reranker 模型（后台下载中）

**用法**：
```bash
qmd vsearch "研发负责人的主要职责"
```

### 混合搜索（qmd query）
最高质量搜索，结合关键词、语义和查询扩展。

**需要**：Generation 模型（后台下载中，预计 30 分钟）

**用法**：
```bash
qmd query "如何提高研发效率"
```

## 实用示例

### 查找特定主题
```bash
# 查找关于 OC1 软件组的信息
qmd search "OC1 软件组" -n 5

# 查找任务负载相关信息
qmd search "任务负载" -n 3

# 查找配置相关文档
qmd search "配置" -c workspace
```

### 查找特定人员
```bash
# 查找王永锋相关信息
qmd search "王永锋"

# 查找研发负责人相关信息
qmd search "研发负责人"
```

### 查找时间相关信息
```bash
# 查找 2026-03-05 的记录
qmd search "2026-03-05" -c memory

# 查找最近的工作日志
qmd ls memory
```

## 文件同步

当本地文件更新后，需要同步到远程服务器：

```bash
# 运行同步脚本
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh

# 更新 QMD 索引
qmd update
```

## 性能特点

- **搜索速度**：毫秒级响应
- **索引大小**：3.6 MB（56 个文件，111 个 vectors）
- **GPU 加速**：使用 RTX 5080（14.3 GB VRAM 空闲）
- **本地优先**：所有数据在本地，保护隐私

## 故障排查

### 搜索无结果
```bash
# 检查索引状态
qmd status

# 重新索引
qmd update
```

### 文件未找到
```bash
# 检查文件是否在 collection 中
qmd ls workspace | grep "文件名"

# 重新同步文件
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh
qmd update
```

### 模型下载状态
```bash
# 检查模型文件
ssh howie@192.168.110.205 "ls -lh ~/.cache/qmd/models/"

# 检查下载进程
ssh howie@192.168.110.205 "ps aux | grep qmd"
```

## 高级用法

### 组合搜索
```bash
# 先用 search 快速定位
qmd search "关键词" -n 10

# 然后用 get 查看完整内容
qmd get qmd://workspace/path/to/file.md
```

### 导出结果
```bash
# JSON 格式
qmd search "查询" --json

# CSV 格式
qmd search "查询" --csv

# 仅文件列表
qmd search "查询" --files
```

## 下一步

等待模型下载完成后（预计 30 分钟），将可以使用：
- `qmd vsearch` - 语义搜索
- `qmd query` - 混合搜索（最高质量）

届时会有更强大的搜索能力，能够理解自然语言查询。
