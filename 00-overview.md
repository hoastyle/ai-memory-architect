# AI 记忆系统方案设计总览

**版本**: v1.0
**日期**: 2026-03-08
**目标**: 为个人和企业场景设计两套开源 AI 记忆系统方案

---

## 📋 项目目标

设计两套记忆系统方案：

### 👤 个人版
- **定位**: 轻量化、低硬件需求、易部署
- **核心需求**:
  - 开源
  - 优先纯 CPU 方案
  - 硬件资源受限环境（OpenClaw 运行环境）
  - 备选远程服务支持
- **参考方案**: 三层架构 + 混合检索 + 记忆整合

### 🏢 企业版
- **定位**: 权限管理、知识分层、知识管控
- **核心需求**:
  - 开源
  - 多用户权限管理
  - 知识分层和隔离
  - 审计和管控能力
- **核心模块**: WeKnora（腾讯企业级 RAG 框架）

---

## 📚 参考方案分析

### 1. openclaw_memory_supersystem-v1.0
**特点**: 最成熟的神经科学启发方案
- 三层架构：工作记忆/长期记忆/事件日志
- 7阶段 Consolidation（记忆整合）
- Fact/Belief/Summary 分类
- SQLite 后端 + 混合检索
- 时序引擎 + 冲突消解

### 2. memory-lancedb-pro
**特点**: 检索能力最强
- LanceDB 向量数据库
- 混合检索：Vector + BM25
- Cross-Encoder Rerank（Jina/Cohere）
- 多阶段评分管线
- 多 Scope 隔离

### 3. openclaw-supermemory
**特点**: 云端托管方案
- 基于 Supermemory 云服务
- Auto-Recall + Auto-Capture
- 无需本地基础设施
- 持久化用户画像

### 4. another_optimized_solution.md
**特点**: 集大成者
- 三层物理记忆（Tier-0/1/2）
- 脱水打标法
- 时序防僵尸（Supersede）
- 人工审核闭环

---

## 🎯 方案定位

| 维度 | 个人版 | 企业版 |
|------|--------|--------|
| **架构基础** | 三层记忆 + 混合检索 | WeKnora RAG 框架 |
| **存储方案** | SQLite + LanceDB | 分布式向量数据库 |
| **检索策略** | 混合检索 + Rerank | 企业级语义检索 |
| **权限管理** | 单用户 | 多用户 + RBAC |
| **部署复杂度** | ⭐⭐ | ⭐⭐⭐⭐ |
| **硬件需求** | 低（纯 CPU 可选） | 中高 |
| **适用场景** | 个人助手、小团队 | 企业知识管理 |

---

## 📂 文档结构

```
memory_design/
├── 00-overview.md              # 本文件：总览
├── 01-reference-analysis.md    # 参考方案深度对比
├── 02-personal-solution.md     # 个人版详细方案
└── 03-enterprise-solution.md   # 企业版详细方案
```

---

## 🔄 下一步

1. 阅读 `01-reference-analysis.md` 了解参考方案的深度对比
2. 根据使用场景选择：
   - 个人使用 → `02-personal-solution.md`
   - 企业使用 → `03-enterprise-solution.md`
