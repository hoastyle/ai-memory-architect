# 企业版方案（优化版）

**版本**: v2.0
**日期**: 2026-03-08
**架构**: WeKnora + supersystem 混合架构

---

## 🎯 方案定位

**核心决策**: WeKnora（检索层）+ supersystem（管理层）混合架构

**理由**:
1. ✅ WeKnora 提供企业级 RAG 和权限管理
2. ✅ supersystem 提供记忆分类和质量管理
3. ✅ 两者互补，避免重复开发
4. ✅ 完全开源（WeKnora + supersystem 均为开源）

---

## 🏗️ 混合架构设计

### 整体架构

```
┌─────────────────────────────────────────────────┐
│           用户层（多租户）                       │
│  部门A用户 | 部门B用户 | 项目组用户             │
└────────┬────────────────────────────────────────┘
         │
    ┌────┴────────────────────────────────────┐
    │      WeKnora RAG Framework              │
    │  - 权限管理（RBAC）                      │
    │  - 知识库分层                            │
    │  - 语义检索                              │
    │  - 审计日志                              │
    └────────┬────────────────────────────────┘
             │ API 桥接
    ┌────────▼────────────────────────────────┐
    │   supersystem 记忆管理层                 │
    │  - Fact/Belief/Summary 分类             │
    │  - 置信度标注                            │
    │  - 记忆整合（Consolidation）             │
    │  - 质量评分                              │
    └────────┬────────────────────────────────┘
             │
    ┌────────▼────────────────────────────────┐
    │         存储层                           │
    │  PostgreSQL | Milvus | Elasticsearch    │
    └─────────────────────────────────────────┘
```

### 四层知识分层

```
L0: 企业知识库（全局）
├─ 公司政策、规范
├─ 权限: 所有人只读，管理员读写
└─ 存储: WeKnora Document KB

L1: 部门知识库（部门隔离）
├─ 部门专属知识
├─ 权限: 部门成员只读，部门管理员读写
└─ 存储: WeKnora Document KB + Scope

L2: 项目知识库（项目隔离）
├─ 项目文档、决策记录
├─ 权限: 项目成员读写
└─ 存储: WeKnora Document KB + Scope

L3: 个人记忆（用户隔离）
├─ 个人笔记、对话历史
├─ 权限: 仅本人可见
└─ 存储: supersystem (per-user instance)
```

---

## 🔧 核心组件

### 1. WeKnora 检索层

**功能**:
- 文档理解和语义检索
- 多知识库类型（Document, FAQ）
- Parent-child chunking
- 权限过滤

**配置**:
```yaml
# weknora-config.yml
knowledge_bases:
  - name: "enterprise"
    type: "document"
    scope: "global"
    permissions:
      read: ["all"]
      write: ["admin"]

  - name: "dept_engineering"
    type: "document"
    scope: "department:engineering"
    permissions:
      read: ["dept:engineering"]
      write: ["dept_admin:engineering"]
```

### 2. supersystem 管理层

**功能**:
- 记忆分类（Fact/Belief/Summary）
- 置信度标注
- 质量评分
- 记忆整合

**集成方式**:
```python
# memory_bridge.py
class MemoryBridge:
    def __init__(self, weknora_client, supersystem_engine):
        self.weknora = weknora_client
        self.supersystem = supersystem_engine

    def store_knowledge(self, content, level, user_id):
        # 1. supersystem 分类和标注
        classified = self.supersystem.classify(content)

        # 2. WeKnora 存储和索引
        doc_id = self.weknora.store(
            content=classified['content'],
            metadata={
                'type': classified['type'],  # Fact/Belief/Summary
                'confidence': classified['confidence'],
                'level': level,
                'user_id': user_id
            }
        )

        return doc_id

    def search_knowledge(self, query, user_id, level):
        # 1. WeKnora 权限过滤 + 检索
        results = self.weknora.search(
            query=query,
            filters={'level': level, 'user_id': user_id}
        )

        # 2. supersystem 质量评分
        scored = self.supersystem.score_results(results)

        return scored
```

---

## 📋 实施路线图

### Phase 1: 基础设施（2-3 周）

**任务**:
1. 部署 WeKnora 框架
2. 配置 PostgreSQL + Milvus
3. 搭建用户认证（OAuth2/LDAP）
4. 实现 RBAC 权限模型

**交付物**:
- WeKnora 运行环境
- 用户认证系统
- 权限管理后台

### Phase 2: 记忆管理集成（2-3 周）

**任务**:
1. 部署 supersystem（多实例模式）
2. 开发 API 桥接层（memory_bridge.py）
3. 实现记忆分类和标注
4. 集成质量评分

**交付物**:
- supersystem 集成
- API 桥接服务
- 记忆分类系统

### Phase 3: 知识分层（1-2 周）

**任务**:
1. 配置四层知识库
2. 实现权限过滤
3. 跨层级检索
4. 知识审核工作流

**交付物**:
- 四层知识库
- 权限过滤系统
- 审核工作流

### Phase 4: 管控能力（1-2 周）

**任务**:
1. 审计日志（Elasticsearch）
2. 合规检查
3. 质量监控
4. 导出管控

**交付物**:
- 审计日志系统
- 合规检查工具
- 质量监控面板

---

## 🔄 工作流程

### 知识创建流程

```
用户创建知识
    ↓
选择层级（L0/L1/L2/L3）
    ↓
WeKnora 权限检查
    ↓
supersystem 分类标注
    ├─ Fact（高置信度）
    ├─ Belief（中置信度）
    └─ Summary（总结）
    ↓
（L0/L1）管理员审批
    ↓
WeKnora 存储 + 索引
    ↓
记录审计日志
```

### 知识检索流程

```
用户查询
    ↓
WeKnora 权限过滤
    ↓
语义检索（Vector + BM25）
    ↓
supersystem 质量评分
    ├─ 置信度加权
    ├─ 时间衰减
    └─ 重要性排序
    ↓
返回结果
```

---

## 🛠️ 部署架构

### Docker Compose 配置

```yaml
version: '3.8'

services:
  # WeKnora API
  weknora-api:
    image: weknora/api:latest
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=postgres
      - VECTOR_DB_HOST=milvus
      - AUTH_PROVIDER=ldap
    depends_on:
      - postgres
      - milvus

  # supersystem 桥接服务
  memory-bridge:
    build: ./memory-bridge
    ports:
      - "8081:8081"
    environment:
      - WEKNORA_URL=http://weknora-api:8080
      - SUPERSYSTEM_PATH=/data/supersystem
    volumes:
      - supersystem_data:/data/supersystem

  # PostgreSQL
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=weknora
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Milvus 向量数据库
  milvus:
    image: milvusdb/milvus:latest
    ports:
      - "19530:19530"
    volumes:
      - milvus_data:/var/lib/milvus

  # Keycloak 认证
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=${KEYCLOAK_PASSWORD}
    ports:
      - "8180:8080"

  # Elasticsearch 审计日志
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
    volumes:
      - es_data:/usr/share/elasticsearch/data

volumes:
  supersystem_data:
  postgres_data:
  milvus_data:
  es_data:
```

---

## 📊 性能指标

| 指标 | 目标值 |
|------|--------|
| 并发用户 | 100-500 |
| 检索延迟 | < 500ms (P95) |
| 知识容量 | 100 万+ |
| 可用性 | 99.9% |
| 数据备份 | 每日 |

---

## 💡 最佳实践

### 1. 知识分层原则
- L0: 稳定、权威、全局适用（如公司政策）
- L1: 部门特色、中等变化（如部门流程）
- L2: 项目专属、频繁更新（如项目文档）
- L3: 个人笔记、高度动态（如个人偏好）

### 2. 记忆质量管理
- Fact 优先：确定的事实优先级最高
- Belief 验证：定期验证推断的准确性
- 低置信度清理：自动清理低置信度的 Belief

### 3. 权限设计
- 最小权限原则
- 定期权限审查
- 敏感知识加密

---

## 📝 总结

**企业版方案 = WeKnora + supersystem**

**核心优势**:
1. WeKnora 提供企业级能力（权限、分层、审计）
2. supersystem 提供记忆质量管理（分类、置信度、整合）
3. 完全开源
4. 互补架构，避免重复开发

**实施周期**: 6-8 周

**推荐配置**: 中等规模企业（100-500 用户）
