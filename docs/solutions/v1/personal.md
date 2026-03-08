# 个人版 AI 记忆系统方案

**版本**: v1.0
**日期**: 2026-03-08
**定位**: 轻量化、低硬件需求、易部署的个人 AI 记忆系统

---

## 🎯 设计目标

### 核心约束
1. **开源**：所有组件必须开源
2. **低硬件需求**：优先纯 CPU 方案
3. **易部署**：最小化配置复杂度
4. **高质量**：检索精度 > 85%

### 硬件环境
- **CPU**: 2-4 核
- **内存**: 4-8GB
- **存储**: 10-50GB
- **网络**: 可选远程 Embedding API

---

## 🏗️ 架构设计

### 三层记忆架构

```
┌─────────────────────────────────────────┐
│  Tier-0: 热区（每次启动必读）            │
│  - IDENTITY.md: AI 身份认知              │
│  - USER.md: 用户档案                     │
│  - MEMORY.md: 核心长期记忆（<100条）     │
│  存储: 纯文本文件                        │
└──────────────┬──────────────────────────┘
               │ 按需检索
               ▼
┌─────────────────────────────────────────┐
│  Tier-1: 长期记忆库                      │
│  - SQLite: 结构化存储                    │
│  - LanceDB: 向量索引（可选）             │
│  - 混合检索: 关键词 + 语义               │
│  容量: 1000-10000 条                     │
└──────────────┬──────────────────────────┘
               │ 定期整合
               ▼
┌─────────────────────────────────────────┐
│  Tier-2: 暂存缓冲区                      │
│  - 每日对话日志: YYYY-MM-DD.md           │
│  - 待整合记忆: pending.jsonl             │
│  存储: 文本 + JSONL                      │
└─────────────────────────────────────────┘
```

---

## 🔧 技术方案

### 方案 A：纯 CPU 轻量版（推荐）

**适用场景**: 硬件资源受限，优先本地化

#### 核心组件
1. **存储层**
   - SQLite（WAL 模式）
   - 全文索引（FTS5）
   - 文件系统（Tier-0/2）

2. **检索层**
   - BM25 全文检索（纯 CPU）
   - TF-IDF 关键词匹配
   - 时间衰减 + 重要性加权

3. **记忆分类**
   - Fact（事实）
   - Decision（决策）
   - Preference（偏好）

4. **整合机制**
   - 每日轻量检查（Mini-Consolidate）
   - 每周深度整合（Full-Consolidate）

#### 技术栈
```python
# 核心依赖
sqlite3          # 内置，无需安装
jieba            # 中文分词
scikit-learn     # TF-IDF（可选）
```

#### 优势
- ✅ 零 GPU 依赖
- ✅ 内存占用 < 500MB
- ✅ 启动速度 < 1s
- ✅ 完全本地化

#### 劣势
- ❌ 无语义理解（纯关键词）
- ❌ 检索精度 ~75%

---

### 方案 B：混合检索增强版

**适用场景**: 可接受远程 API，追求高精度

#### 核心组件
1. **存储层**
   - SQLite + LanceDB
   - 向量索引（远程 Embedding）

2. **检索层**
   - Vector Search（语义）
   - BM25 Search（关键词）
   - RRF 融合

3. **Embedding 服务**
   - Jina AI（免费额度）
   - OpenAI（付费）
   - Ollama（本地，需 GPU）

4. **Rerank 服务**（可选）
   - Jina Reranker（免费额度）
   - 本地 Cross-Encoder（需 GPU）

#### 技术栈
```python
# 核心依赖
lancedb          # 向量数据库
openai           # API 客户端
requests         # HTTP 请求
```

#### 优势
- ✅ 语义理解能力
- ✅ 检索精度 > 85%
- ✅ 支持换种说法搜索

#### 劣势
- ❌ 依赖网络
- ❌ API 调用成本

---

## 📋 实现路线图

### Phase 1: 基础架构（1-2 天）
1. 创建三层目录结构
2. 实现 SQLite 存储层
3. 实现 Tier-0 文件读写
4. 基础 CLI 命令

### Phase 2: 检索能力（2-3 天）
**方案 A**:
- SQLite FTS5 全文索引
- BM25 检索实现
- 时间衰减算法

**方案 B**:
- LanceDB 集成
- 远程 Embedding API
- 混合检索 + RRF 融合

### Phase 3: 记忆整合（2-3 天）
1. Mini-Consolidate（每日）
2. Full-Consolidate（每周）
3. Supersede 机制（防僵尸）

### Phase 4: 工具集成（1-2 天）
1. OpenClaw 插件适配
2. CLI 管理工具
3. 导入导出功能

---

## 🔄 工作流程

### 1. 启动流程
```
OpenClaw 启动
    ↓
读取 Tier-0（IDENTITY + USER + MEMORY）
    ↓
加载到 Prompt Context
    ↓
准备就绪
```

### 2. 对话流程
```
用户输入
    ↓
自适应检索判断（跳过问候/命令）
    ↓
混合检索 Tier-1（Top-5）
    ↓
注入 <relevant-memories>
    ↓
AI 生成回复
    ↓
记录到 Tier-2（YYYY-MM-DD.md）
```

### 3. 整合流程
```
每日 23:00 触发 Mini-Consolidate
    ↓
扫描 Tier-2 当日日志
    ↓
提取候选记忆 → pending.jsonl
    ↓
（可选）人工审核
    ↓
脱水打标 → Tier-1
    ↓
更新 Tier-0（核心记忆）
```

---

## 📊 性能指标

| 指标 | 方案 A（纯 CPU） | 方案 B（混合） |
|------|-----------------|---------------|
| **启动时间** | < 1s | < 2s |
| **检索延迟** | < 50ms | < 200ms |
| **内存占用** | < 500MB | < 1GB |
| **检索精度** | ~75% | > 85% |
| **Token 消耗** | < 1500/次 | < 1500/次 |
| **网络依赖** | 无 | 有（Embedding） |

---

## 🛠️ 部署指南

### 方案 A 部署
```bash
# 1. 克隆仓库
git clone <repo-url>
cd memory-system

# 2. 安装依赖
pip install -r requirements-lite.txt

# 3. 初始化
python memory.py init

# 4. 配置 Tier-0
vim memory/IDENTITY.md
vim memory/USER.md

# 5. 启动
python memory.py serve
```

### 方案 B 部署
```bash
# 1-4 同方案 A

# 5. 配置 API
export JINA_API_KEY="your-key"

# 6. 启动（启用向量检索）
python memory.py serve --enable-vector
```

---

## 💡 最佳实践

### 1. Tier-0 管理
- IDENTITY.md: 50-100 行
- USER.md: 30-50 行
- MEMORY.md: < 100 条核心记忆

### 2. 记忆分类
```markdown
[Fact] Docker compose 端口映射格式：HOST:CONTAINER
[Decision] 遇到 ECONNREFUSED 先检查端口映射
[Preference] 用户偏好使用 TypeScript 而非 JavaScript
```

### 3. Supersede 示例
```python
# 存入新记忆前
old_memories = memory_recall("Docker 端口映射")
if old_memories:
    memory_update(old_id, new_content)  # 覆盖旧版本
else:
    memory_store(new_content)  # 新增
```

---

## 🔮 未来扩展

### 短期（1-3 个月）
- [ ] 支持本地 Ollama Embedding
- [ ] 图形化管理界面
- [ ] 多 Agent 隔离

### 中期（3-6 个月）
- [ ] 移动端同步
- [ ] 知识图谱可视化
- [ ] 自动技能提取

### 长期（6-12 个月）
- [ ] 联邦学习（多设备）
- [ ] 主动记忆推荐
- [ ] 记忆质量评分
