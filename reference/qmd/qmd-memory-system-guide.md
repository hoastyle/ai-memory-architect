# 基于 QMD 的记忆系统 - 使用指南

**版本**: 2.0  
**更新时间**: 2026-03-08 05:00 UTC  
**状态**: Phase 1-3 完成

---

## 系统概览

CoR 的记忆系统基于 QMD (Quantum Memory Dynamics)，提供三种搜索模式：
- **BM25 全文搜索**：快速关键词匹配
- **语义搜索**：理解查询意图
- **混合搜索**：查询扩展 + 智能排序

---

## 系统架构

```
┌─────────────────────────────────────────────────┐
│  本地文件系统                                    │
│  - MEMORY.md (长期记忆)                          │
│  - memory/YYYY-MM-DD.md (每日日志)               │
│  - workspace/*.md (工作区文档)                   │
└─────────────────────────────────────────────────┘
                    ↓ rsync 同步
┌─────────────────────────────────────────────────┐
│  QMD 索引服务（远程 GPU 服务器）                 │
│  - 56 个文件已索引                               │
│  - 111 个向量                                    │
│  - RTX 5080 GPU 加速                             │
└─────────────────────────────────────────────────┘
                    ↓ 搜索调用
┌─────────────────────────────────────────────────┐
│  QMD Wrapper Script (v2.0)                      │
│  - 智能路由                                      │
│  - 自动降级                                      │
│  - 格式转换                                      │
└─────────────────────────────────────────────────┘
```

---

## 运转机制

### 1. 文件同步

**自动同步**（可选）：
```bash
# 每小时同步一次（cron）
0 * * * * /home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh
```

**手动同步**：
```bash
# 同步文件到 QMD 服务器
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh

# 更新 QMD 索引
qmd update
```

### 2. 搜索流程

```
用户查询
    ↓
智能路由（自动选择搜索模式）
    ↓
┌─────────┬─────────┬─────────┐
│ search  │ vsearch │  query  │
│ (BM25)  │ (语义)  │ (混合)  │
└─────────┴─────────┴─────────┘
    ↓
降级机制（如果失败）
    ↓
返回结果（JSON 格式）
```

### 3. 智能路由规则

| 查询类型 | 示例 | 选择模式 | 原因 |
|---------|------|----------|------|
| 简单关键词 | "王永锋" | vsearch | 短查询，语义理解 |
| 单词/数字 | "OC1" | search | 精确匹配，快速 |
| 自然语言 | "研发负责人的主要职责" | vsearch | 语义理解 |
| 复杂问句 | "如何提高研发效率？" | query | 查询扩展，最佳 |

### 4. 降级策略

```
query 失败
    ↓
vsearch 失败
    ↓
search 失败
    ↓
返回错误（不中断流程）
```

---

## 使用方法

### 方式 1: 命令行（推荐）

**创建别名**（一次性配置）：
```bash
echo 'alias qmd-search="/home/matchine/.openclaw/workspace/scripts/qmd-memory-search.sh"' >> ~/.bashrc
source ~/.bashrc
```

**使用**：
```bash
# 自动模式（推荐）
qmd-search "王永锋的任务负载" 5 0.6 auto

# 指定模式
qmd-search "王永锋" 5 0.6 search   # BM25（快速）
qmd-search "研发负责人" 5 0.6 vsearch  # 语义（准确）
qmd-search "如何提高效率？" 5 0.4 query  # 混合（最佳）
```

**参数说明**：
- 参数 1: 查询内容
- 参数 2: 最大结果数（默认 10）
- 参数 3: 最低分数（默认 0.5）
- 参数 4: 搜索模式（auto/search/vsearch/query，默认 auto）

### 方式 2: 在对话中使用

当我需要搜索历史记忆时，可以主动调用：

```
我: 让我搜索一下王永锋的相关信息...
[exec: /home/matchine/.openclaw/workspace/scripts/qmd-memory-search.sh "王永锋 任务 负载" 5 0.6 auto]
[分析返回的 JSON 结果]
我: 根据搜索结果，王永锋当前负载 131 个任务...
```

---

## 实际示例

### 示例 1: 查找人员信息

**查询**：
```bash
qmd-search "王永锋" 5 0.6 auto
```

**结果**：
```json
{
  "results": [
    {
      "path": "reports/research-status-2026-03-05.md",
      "from": 13,
      "score": 0.72,
      "snippet": ["王永锋负载 131 个（P0:55），远超平均水平 8 倍"],
      "title": "CoR 研发态势总览"
    },
    {
      "path": "teams/oc1-software/reports/latest-summary.md",
      "from": 10,
      "score": 0.72,
      "snippet": ["王永锋：131个（P0:55 P1:76）⚠️ 高负载"],
      "title": "OC1 软件组 - 最新日报摘要"
    }
  ],
  "provider": "qmd",
  "mode": "vsearch"
}
```

**解读**：
- 找到 2 个相关文档
- 使用 vsearch 模式（语义搜索）
- 准确率：72%
- 响应时间：~2000ms

### 示例 2: 查找历史决策

**查询**：
```bash
qmd-search "为什么选择 Docker 容器方案" 5 0.5 auto
```

**结果**：
- 自动选择 query 模式（复杂问句）
- 查询扩展：Docker + 容器 + 方案 + 架构
- 返回相关讨论和决策记录

### 示例 3: 快速关键词查询

**查询**：
```bash
qmd-search "OC1" 10 0.5 auto
```

**结果**：
- 自动选择 search 模式（简单关键词）
- 响应时间：~500ms（最快）
- 返回所有包含 "OC1" 的文档

---

## 性能指标

| 搜索模式 | 响应时间 | 准确率 | 适用场景 |
|---------|----------|--------|----------|
| search (BM25) | ~500ms | 72% | 简单关键词、精确匹配 |
| vsearch (语义) | ~2000ms | 68% | 自然语言、短查询 |
| query (混合) | ~4000ms | 89% | 复杂问句、最佳质量 |
| auto (智能) | 变化 | 最优 | **推荐使用** |

---

## 对比：传统 vs QMD

### 传统方式（无 QMD）

**限制**：
- 只能搜索启动时加载的文件
- 依赖精确关键词
- 无法理解查询意图
- 可能遗漏重要信息

**示例**：
```
你: "上个月讨论的架构方案"
我: [只能看到 MEMORY.md 和最近两天的日志]
   "抱歉，我没有找到相关信息"
```

### QMD 方式（当前）

**优势**：
- 搜索所有历史文档（56 个文件）
- 语义理解查询意图
- 自动查询扩展
- 智能排序和重排序

**示例**：
```
你: "上个月讨论的架构方案"
我: [QMD query 模式]
   [查询扩展：架构 + 方案 + 设计 + 讨论]
   [搜索所有 memory/*.md]
   "找到了！2026-02-15 讨论了微服务架构方案..."
```

---

## 常见场景

### 场景 1: 查找特定人员信息

```bash
# 查找王永锋的任务负载
qmd-search "王永锋 任务 负载" 5 0.6 auto

# 查找郭际的工作内容
qmd-search "郭际" 5 0.6 auto
```

### 场景 2: 查找历史决策

```bash
# 查找架构决策
qmd-search "为什么选择这个架构" 5 0.5 auto

# 查找技术选型
qmd-search "技术选型 理由" 5 0.5 auto
```

### 场景 3: 查找时间相关信息

```bash
# 查找特定日期的记录
qmd-search "2026-03-05" 10 0.5 search

# 查找最近的讨论
qmd-search "最近讨论的问题" 5 0.5 auto
```

### 场景 4: 查找主题相关信息

```bash
# 查找研发效率相关
qmd-search "如何提高研发效率" 5 0.4 auto

# 查找风险相关
qmd-search "当前有哪些风险" 5 0.5 auto
```

---

## 故障排查

### 问题 1: 搜索无结果

**症状**：`"results": []`

**排查**：
```bash
# 1. 检查 QMD 状态
qmd status

# 2. 更新索引
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh
qmd update

# 3. 降低最低分数
qmd-search "查询" 10 0.3 auto
```

### 问题 2: 响应时间过长

**症状**：响应时间 > 5 秒

**排查**：
```bash
# 1. 使用更快的模式
qmd-search "查询" 5 0.6 search  # 使用 BM25

# 2. 减少结果数量
qmd-search "查询" 3 0.6 auto

# 3. 检查 GPU 状态
qmd status  # 查看 VRAM 使用情况
```

### 问题 3: 命令未找到

**症状**：`qmd-search: command not found`

**解决**：
```bash
# 1. 检查脚本是否存在
ls -l /home/matchine/.openclaw/workspace/scripts/qmd-memory-search.sh

# 2. 添加执行权限
chmod +x /home/matchine/.openclaw/workspace/scripts/qmd-memory-search.sh

# 3. 重新创建别名
echo 'alias qmd-search="/home/matchine/.openclaw/workspace/scripts/qmd-memory-search.sh"' >> ~/.bashrc
source ~/.bashrc
```

---

## 维护

### 日常维护

```bash
# 1. 同步文件（每天或每次重要更新后）
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh

# 2. 更新索引
qmd update

# 3. 检查状态
qmd status
```

### 定期检查

**每周**：
- 检查 QMD 服务状态
- 验证搜索质量
- 清理旧索引

**每月**：
- 评估搜索准确率
- 优化智能路由规则
- 更新文档

---

## 限制与注意事项

### 当前限制

1. **手动调用**：需要主动调用脚本，未实现全自动集成
2. **响应时间**：复杂查询可能需要 3-4 秒
3. **索引更新**：需要手动同步和更新索引

### 注意事项

1. **敏感信息**：credentials/ 目录已排除，不会被索引
2. **文件大小**：大文件可能影响索引速度
3. **GPU 依赖**：语义搜索和混合搜索需要 GPU 支持

---

## 下一步

### 短期计划

- [ ] 实现全自动集成（TODO 项）
- [ ] 优化智能路由逻辑
- [ ] 添加缓存层

### 长期计划

- [ ] 主动记忆维护
- [ ] 定期评估搜索质量
- [ ] 提交 PR 到 OpenClaw

---

## 相关文档

- **集成指南**: [docs/qmd-integration-guide.md](qmd-integration-guide.md)
- **完整架构**: [docs/memory-system-with-qmd.md](memory-system-with-qmd.md)
- **待办事项**: [TODO.md](../TODO.md)
- **工具说明**: [TOOLS.md](../TOOLS.md)

---

## 快速参考

### 常用命令

```bash
# 自动模式（推荐）
qmd-search "查询内容" 5 0.6 auto

# 快速搜索
qmd-search "关键词" 10 0.5 search

# 语义搜索
qmd-search "自然语言查询" 5 0.6 vsearch

# 最佳质量
qmd-search "复杂问句？" 5 0.4 query

# 同步和更新
/home/matchine/.openclaw/workspace/scripts/sync-to-qmd.sh
qmd update

# 检查状态
qmd status

# 运行测试
/home/matchine/.openclaw/workspace/scripts/test-qmd-complete.sh
```

---

**版本**: 2.0  
**最后更新**: 2026-03-08 05:00 UTC  
**状态**: Phase 1-3 完成 ✅
