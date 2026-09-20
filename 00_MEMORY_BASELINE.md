# 面试冲刺外部记忆基线

> 目的：用于未来对话快速恢复这次 5 天面试冲刺的事实、课程边界和进度。  
> 使用规则：后续开始课程前，应先阅读本文件和 PROGRESS.md，再决定从哪里继续。

# 1. 总目标

用户只有 5 天准备时间，目标是 AI 全栈开发实习面试。

5 天结束时要求：

- 简历核心项目能够自然讲解
- 能承受项目三层以上追问
- 简历核心技术栈具备实习面试所需理解
- 回答必须优先结合真实项目，不采用脱离项目的八股堆砌
- 不追求五天“精通所有技术”，追求高频、高价值、可解释、可验证

# 2. 核心学习方法

固定训练循环：

~~~
知识点
 ↓
真实项目中的落点
 ↓
面试主问题
 ↓
用户自己回答
 ↓
三层追问
 ↓
纠错
 ↓
用户重新回答
 ↓
通过标准
~~~

原则：

- 不能只看课件
- 不能只背标准答案
- 要求用户输出
- 项目事实必须与 GitHub 对齐
- 不了解的项目实现必须先查仓库，禁止凭想象补齐

# 3. 简历核心项目优先级

第一梯队：

1. 渔芯 AI 水产养殖一体化系统
2. 云邻 AI 智脑
3. Enterprise Smart Assistant

第二梯队：

4. Codex Project Factory
5. KnowFlow AI

防守型项目：

6. StudyAgent
7. HarmonyOS 上架应用

# 4. 渔芯项目事实基线

仓库：

Changxin-YR/yuxin

简历中的旧 FPA 链接已经重定向到该仓库。

真实技术栈：

- Python
- Flask
- PyMySQL
- MySQL
- Vue 3
- TypeScript
- Vite
- DeepSeek Harness
- Agent Gateway
- pytest
- Vitest
- Playwright

明确不属于这个项目：

- SQLAlchemy
- Alembic
- Redis
- FastAPI
- ORM

数据库层特点：

- PyMySQL
- 手写 SQL
- UnitOfWork 显式事务边界
- autocommit=False
- 需要时支持 SAVEPOINT

# 5. 渔芯核心业务域

- master_data：基地、区域、塘口、物料、往来单位
- production：批次、投喂、捕捞
- warehouse：库存、流水、仓储
- purchase：采购、收货、应付、付款
- sales：销售、发货、应收、收款
- cost：成本
- identity：身份
- access：权限与 DataScope
- audit：审计
- agent：Agent Gateway

# 6. 渔芯核心架构事实

Capability 是一个业务能力的唯一声明来源。

一条 Capability 会包含/驱动：

- REST method/path
- handler/service
- required permission
- DataScope
- risk
- confirmation
- AgentExposure
- idempotency
- audit
- invariants
- Agent Tool schema
- 部分前端元数据

典型人工调用链：

~~~
Vue
 ↓
Flask
 ↓
Capability Route
 ↓
CapabilityRunner
 ↓
权限 / DataScope / 校验 / 不变量
 ↓
Domain Service
 ↓
UnitOfWork
 ↓
PyMySQL
 ↓
MySQL
~~~

Agent 基础调用链：

~~~
DeepSeek Harness
 ↓
Agent Tool
 ↓
Agent Gateway
 ↓
Capability / Business Service
 ↓
UnitOfWork
 ↓
MySQL
~~~

# 7. 第一项目的重要面试亮点

1. Capability 单一事实来源，避免页面/API/Agent 多处定义漂移
2. Web / Agent 不直接写 SQL
3. 页面与 Agent 最终复用业务规则
4. UnitOfWork 控制显式事务
5. 幂等、权限、DataScope、确认、审计属于系统控制，不信任模型
6. Agent 写成功需要真实数据库事实，不只信任模型自然语言
7. 生产域跨域副作用通过目标业务域提供的受控入口实现，不应随意直接写其他域表

# 8. 5 天课程计划

Day 1：
- 渔芯整体业务与架构
- Python 项目必备基础
- HTTP / Flask / REST
- MySQL / SQL
- 事务 / UnitOfWork
- 真实代码走读
- 项目 + 后端模拟面试

Day 2：
- MySQL 并发
- 事务隔离
- 悲观锁
- 乐观锁
- row_version
- 幂等
- 状态机
- RBAC
- DataScope
- 审计
- 高风险操作确认
- 用渔芯 + 云邻进行追问

Day 3：
- LLM / Agent
- Tool Calling
- LangChain
- LangGraph
- MCP
- RAG
- Chroma / Qdrant
- Hybrid RAG
- Rerank
- Human-in-the-loop
- Enterprise Smart Assistant 深挖

Day 4：
- Vue3 / TypeScript
- 前后端联调
- Redis
- SSE
- Docker
- Linux
- Nginx
- pytest / Vitest / Playwright
- 部署与测试
- 其他项目防守

Day 5：
- 不新增大块知识
- 三档项目介绍
- 全简历高压追问
- 场景设计
- AI Agent 深挖
- 后端基础深挖
- 模拟完整面试
- 只补暴露出的薄弱点

# 9. 掌握等级

L1：知道定义  
L2：能解释原理  
L3：能结合自己的项目说明  
L4：能回答取舍、异常场景和连续追问

核心技术最低要求：

- Capability：L4
- 事务/UnitOfWork：L4
- RBAC/DataScope：L4
- Tool Calling/Agent：L4
- RAG：L4
- Python：高频点 L3
- Flask/FastAPI：核心请求链 L3～L4
- MySQL：事务/并发 L4，其他 L3
- Vue/TS：L3
- Docker/Linux/Nginx：L2～L3
- HarmonyOS/Flutter：防守型 L2～L3

# 10. 教学要求

后续课程必须：

- 以真实企业面试为标准
- 主动纠正用户错误，不顺着错误回答
- 标明哪些必须秒答，哪些只需理解
- 每个重要技术必须绑定简历项目
- 重要项目实现需要查真实 GitHub
- 不允许把不同项目技术栈混为一谈
- 不使用无意义的大量八股挤占五天时间
- 每日必须有综合验收
- 未通过时只补薄弱模块，不整天推倒重学



# 11. Day 1 终审新增约束

根据 2026 年近期 AI 应用 / Agent / AI 全栈岗位与公开面经，Day 1 必须额外覆盖：

- Python 装饰器
- iterator / generator / yield
- 深拷贝 / 浅拷贝
- Flask Application Context / Request Context
- CORS 基础
- SQL NULL / COUNT / WHERE vs HAVING
- 至少 1 道限时 Python/SQL Coding
- 至少 1 道接口故障排查题
- 项目真实性、设计取舍、重构思路、AI 生成代码如何验证

真实面试更偏：
项目连续深挖 → 基础抽查 → Coding/SQL → 场景排障，
而不是大段背定义。

## 渔芯写路径重要事实修正

当前运行时代码 kernel/runner.py 的真实顺序：

~~~
业务写
→ invariants
→ 同事务 reload after
→ 同事务写 success audit
→ 退出 UnitOfWork
→ commit
→ commit 成功后返回 executed
~~~

对于幂等写：

~~~
业务 commit
→ mark_completed
→ 返回 executed
~~~

如果业务可能已提交但幂等状态收口失败，需要进入 COMMIT_UNKNOWN 处理，不能自动重复执行。

注意：
docs/WRITE_CONTRACT.md 中仍存在“commit 后回读”的旧描述；面试与课程以当前运行时代码为事实源，不背漂移文档。
