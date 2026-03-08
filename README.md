# AI Memory System Design

**版本**: v2.0
**日期**: 2026-03-08
**状态**: 方案优化完成

---

## 📋 项目简介

本项目为个人和企业场景设计两套开源 AI 记忆系统方案，基于对 4 个主流参考方案的深度分析。

### 核心结论

| 场景 | 推荐方案 | 核心特点 |
|------|----------|----------|
| **个人版** | supersystem | 7阶段整合、轻量化、纯本地 |
| **企业版** | WeKnora + supersystem | 权限管理、知识分层、审计管控 |

---

## 📂 文档导航

### 阅读顺序

```
00-overview.md                    # 项目总览 →
01-reference-analysis.md          # 参考方案对比 →
04-evaluation-report.md           # 方案评估 →
05-personal-solution-v2.md        # 个人版详细方案 →
06-enterprise-solution-v2.md      # 企业版详细方案 →
07-final-summary.md               # 最终总结
```

### 文档说明

| 文档 | 内容 |
|------|------|
| `00-overview.md` | 项目目标和方案定位 |
| `01-reference-analysis.md` | 4 个参考方案深度对比 |
| `02-personal-solution.md` | 个人版初版方案 |
| `03-enterprise-solution.md` | 企业版初版方案 |
| `04-evaluation-report.md` | 方案评估与问题分析 |
| `05-personal-solution-v2.md` | 个人版优化方案 ⭐ |
| `06-enterprise-solution-v2.md` | 企业版优化方案 ⭐ |
| `07-final-summary.md` | 最终总结和行动指南 |
| `docs/integrated-solution.md` | 三层物理隔离集成方案 |

---

## 🔗 参考仓库

项目包含 3 个 Git submodule，供参考：

| 仓库 | 特点 | 链接 |
|------|------|------|
| **supersystem** | 7阶段整合、神经科学启发 | [GitHub](https://github.com/ktao732084-arch/openclaw_memory_supersystem-v1.0) |
| **memory-lancedb-pro** | 混合检索+Rerank | [GitHub](https://github.com/win4r/memory-lancedb-pro) |
| **supermemory** | 云端托管、自动捕获 | [GitHub](https://github.com/supermemoryai/openclaw-supermemory) |

### 克隆带 submodule 的仓库

```bash
git clone --recurse-submodules <repo-url>
```

如果已克隆，拉取 submodule：
```bash
git submodule update --init --recursive
```

---

## 🚀 快速开始

### 个人版部署

```bash
cd reference/supersystem
pip install -r requirements.txt
python src/memory.py init
python src/memory.py serve
```

### 企业版部署

详见 `docs/06-enterprise-solution-v2.md` 的 4 阶段实施路线图。

---

## 📊 方案对比

| 维度 | 个人版 (supersystem) | 企业版 (WeKnora+) |
|------|---------------------|-------------------|
| 架构 | 三层记忆 | 混合架构 |
| 存储 | SQLite | PostgreSQL + Milvus |
| 检索 | 混合检索 | 企业级语义检索 |
| 权限 | 单用户 | RBAC 多用户 |
| 硬件需求 | 低 (2核/2GB) | 中高 |
| 部署复杂度 | ⭐⭐ | ⭐⭐⭐⭐ |

---

## 📝 License

MIT

---

**项目完成时间**: 2026-03-08
