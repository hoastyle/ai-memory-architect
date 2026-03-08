# 变更日志

## [2.0.1] - 2026-03-08

### 新增参考方案

#### 📚 QMD 方案
- **添加位置**: `reference/qmd/`
- **方案类型**: 远程 GPU 加速的语义搜索系统
- **核心特点**:
  - 三种搜索模式（BM25/语义/混合）
  - RTX 5080 GPU 加速
  - 智能路由和降级策略
- **评估结论**: 不适合个人版（硬件成本高、需网络依赖），可作为企业版可选搜索增强模块
- **文档更新**:
  - ✅ 添加 `reference/qmd/README.md`
  - ✅ 更新主 `README.md` 参考方案列表
  - ✅ 更新 `docs/analysis/01-reference-analysis.md` 对比矩阵

---

## [2.0.0] - 2026-03-08

### 项目重组

#### 🎯 项目命名
- **新名称**: AI Memory Architect
- **原名称**: memory（临时名称）
- **命名理由**:
  - 强调架构设计和方案集成
  - 不限于 OpenClaw，适用于类似 AI 产品
  - 体现项目核心价值

#### 📂 目录结构优化

**重组前**:
```
docs/
├── 00-overview.md
├── 01-reference-analysis.md
├── 02-personal-solution.md
├── 03-enterprise-solution.md
├── 04-evaluation-report.md
├── 05-personal-solution-v2.md
├── 06-enterprise-solution-v2.md
├── 07-final-summary.md
└── integrated-solution.md
```

**重组后**:
```
docs/
├── PROJECT_OVERVIEW.md        # 新增：项目概览
├── README.md                  # 新增：目录说明
├── analysis/                  # 分析文档
│   ├── 00-overview.md
│   ├── 01-reference-analysis.md
│   └── 04-evaluation-report.md
├── solutions/                 # 方案文档
│   ├── v1/                   # 初版方案
│   │   ├── personal.md
│   │   └── enterprise.md
│   └── v2/                   # 优化方案
│       ├── personal.md
│       └── enterprise.md
├── reference/                 # 参考设计
│   └── integrated-solution.md
└── summary/                   # 总结文档
    └── 07-final-summary.md
```

#### 📝 文档更新
- ✅ 更新主 README.md
- ✅ 创建 PROJECT_OVERVIEW.md
- ✅ 创建 docs/README.md
- ✅ 创建 CHANGELOG.md

#### 🎯 改进点
1. **清晰分类**: 按文档类型分类（分析/方案/参考/总结）
2. **版本管理**: 方案文档分 v1/v2，明确演进过程
3. **易于导航**: 提供多个入口文档和阅读指南
4. **未来扩展**: 预留 implementation/ 目录用于代码实现

---

## [1.0.0] - 2026-03-08

### 初始版本

#### ✅ 完成内容
- 深度分析 4 个参考方案
- 设计个人版和企业版方案
- 评估初版方案并优化
- 集成参考方案源码（Git submodules）

#### 📊 方案总结
- **个人版**: 采用 supersystem
- **企业版**: WeKnora + supersystem 混合架构
