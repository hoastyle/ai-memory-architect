# AI Memory Architect

**版本**: v2.0
**日期**: 2026-03-08
**状态**: 方案设计完成

---

## 📋 项目简介

AI Memory Architect 是一个为 OpenClaw 及类似 AI 产品构建完整记忆方案的架构设计项目。基于对主流开源方案的深度分析，提供个人版和企业版两套解决方案。

### 核心目标

- 🎯 **个人版**: 轻量化、低硬件需求、纯本地化
- 🏢 **企业版**: 权限管理、知识分层、审计管控
- 🔓 **完全开源**: 所有方案基于开源组件

### 推荐方案

| 场景 | 方案 | 核心特点 |
|------|------|----------|
| **个人版** | supersystem | 7阶段整合、Fact/Belief区分、纯本地 |
| **企业版** | WeKnora + supersystem | 企业级RAG、权限管理、知识分层 |

---

## 📂 项目结构

```
ai-memory-architect/
├── docs/                       # 文档目录
│   ├── analysis/              # 分析文档
│   │   ├── 00-overview.md
│   │   ├── 01-reference-analysis.md
│   │   └── 04-evaluation-report.md
│   ├── solutions/             # 方案文档
│   │   ├── v1/               # 初版方案
│   │   │   ├── personal.md
│   │   │   └── enterprise.md
│   │   └── v2/               # 优化方案 ⭐
│   │       ├── personal.md
│   │       └── enterprise.md
│   ├── reference/             # 参考设计
│   │   └── integrated-solution.md
│   └── summary/               # 总结
│       └── 07-final-summary.md
├── reference/                  # 参考实现（Git submodules）
│   ├── supersystem/
│   ├── memory-lancedb-pro/
│   └── supermemory/
└── implementation/             # 未来实现代码（待开发）
```

---

## 📖 文档导航

### 快速开始

**推荐阅读顺序**:
```
analysis/00-overview.md           # 项目总览 →
analysis/01-reference-analysis.md # 参考方案对比 →
analysis/04-evaluation-report.md  # 方案评估 →
solutions/v2/personal.md          # 个人版方案 ⭐ →
solutions/v2/enterprise.md        # 企业版方案 ⭐ →
summary/07-final-summary.md       # 最终总结
```

### 文档分类

#### 📊 分析文档 (`docs/analysis/`)
- `00-overview.md` - 项目目标和方案定位
- `01-reference-analysis.md` - 4个参考方案深度对比
- `04-evaluation-report.md` - 初版方案评估与问题分析

#### 💡 方案文档 (`docs/solutions/`)
- `v1/` - 初版方案（已优化，仅供参考）
- `v2/` - **优化方案**（推荐使用）
  - `personal.md` - 个人版详细方案
  - `enterprise.md` - 企业版详细方案

#### 📚 参考文档 (`docs/reference/`)
- `integrated-solution.md` - 三层物理隔离集成方案（社区方案）

#### 📝 总结文档 (`docs/summary/`)
- `07-final-summary.md` - 最终总结和行动指南

---

## 🔗 参考方案

项目集成了 4 个优秀的开源方案作为参考：

| 方案 | 特点 | 仓库 |
|------|------|------|
| **supersystem** | 7阶段整合、神经科学启发 | `reference/supersystem/` |
| **memory-lancedb-pro** | 混合检索+Rerank | `reference/memory-lancedb-pro/` |
| **supermemory** | 云端托管、自动捕获 | `reference/supermemory/` |
| **QMD** | 远程GPU语义搜索 | `reference/qmd/` |

### 克隆项目（含 submodules）

```bash
git clone --recurse-submodules https://github.com/[your-username]/ai-memory-architect.git
```

如果已克隆，拉取 submodules：
```bash
git submodule update --init --recursive
```

---

## 🚀 快速部署

### 个人版（推荐）

```bash
cd reference/supersystem
pip install -r requirements.txt
python src/memory.py init
python src/memory.py serve
```

详见：`docs/solutions/v2/personal.md`

### 企业版

企业版需要 6-8 周实施，包含 4 个阶段：
1. 部署 WeKnora 框架
2. 集成 supersystem 记忆管理
3. 配置知识分层
4. 实现管控能力

详见：`docs/solutions/v2/enterprise.md`

---

## 📊 方案对比

| 维度 | 个人版 | 企业版 |
|------|--------|--------|
| **架构** | 三层记忆 | WeKnora + supersystem |
| **存储** | SQLite | PostgreSQL + Milvus |
| **检索** | 混合检索 | 企业级语义检索 |
| **权限** | 单用户 | RBAC 多用户 |
| **硬件** | 2核/2GB | 中高配置 |
| **部署** | ⭐⭐ | ⭐⭐⭐⭐ |

---

## 🎯 核心特性

### 个人版特性
- ✅ 7阶段 Consolidation（记忆整合）
- ✅ Fact/Belief 区分（元认知）
- ✅ 自动衰减机制
- ✅ 纯本地化（SQLite）
- ✅ 低硬件需求

### 企业版特性
- ✅ 四层知识分层（L0-L3）
- ✅ RBAC 权限管理
- ✅ 审计日志
- ✅ 知识质量管理
- ✅ 完全开源

---

## 🛠️ 未来规划

- [ ] 实现个人版记忆系统
- [ ] 实现企业版记忆系统
- [ ] 提供 Docker 部署方案
- [ ] 开发管理界面
- [ ] 性能优化和测试

---

## 📝 License

MIT

---

## 🙏 致谢

本项目基于以下优秀开源方案：
- [openclaw_memory_supersystem](https://github.com/ktao732084-arch/openclaw_memory_supersystem-v1.0)
- [memory-lancedb-pro](https://github.com/win4r/memory-lancedb-pro)
- [openclaw-supermemory](https://github.com/supermemoryai/openclaw-supermemory)
- [WeKnora](https://github.com/Tencent/WeKnora)

---

**项目完成时间**: 2026-03-08
**维护者**: Howie
