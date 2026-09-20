# 课件自检记录

## Day 1 第一轮：项目事实与技术准确性

已核对来源：

- 用户当前简历
- Changxin-YR/yuxin/README.md
- docs/ARCHITECTURE.md
- docs/WRITE_CONTRACT.md
- backend/yuxin/kernel/capability.py
- backend/yuxin/kernel/uow.py
- backend/yuxin/web/app.py
- backend/yuxin/domains/master_data/capabilities.py
- backend/yuxin/domains/production/capabilities.py
- backend/yuxin/domains/production/write.py

检查结果：

- 渔芯真实后端栈确认为 Flask + PyMySQL + MySQL + 手写 SQL
- 已明确区分 SQLAlchemy / FastAPI / Redis 等其他项目技术，不把它们写成渔芯实现
- Capability 的职责与真实字段一致
- Flask Route 确实由 Capability 注册逻辑派生
- UnitOfWork 的 commit / rollback / savepoint 描述与源码一致
- production 写路径关于权限、范围、业务校验和跨域入口的描述与源码方向一致
- “Agent 成功需要数据库事实”与 WRITE_CONTRACT 的设计目标一致

## Day 1 第二轮：面试价值与教学节奏

检查目标：

- 5 天冲刺下是否有过度展开
- 是否能绑定真实项目
- 是否有明确掌握标准
- 是否有真实企业追问
- 是否能形成每日验收
- 是否存在长期记忆失真

修正：

- 将“明天”改成具体日期 2026-09-21 或 Day 2，避免未来读取歧义
- 将 Session 描述改为更严谨表述：Session 是登录会话概念，具体存储实现并不唯一；渔芯通过会话令牌在服务端解析可信身份
- 保留 GIL、MVCC 深层实现、B+Tree 页结构等为非 Day 1 重点，避免挤占项目与事务训练
- Day 1 重点固定为 Capability、请求链、UnitOfWork、基础 Python/Flask/MySQL
- 综合通过线固定为 78/100，未通过只补最弱模块

结论：

Day 1 课件可作为 2026-09-21 的正式学习版本。


## Day 1 第三轮终审：企业真实面试对齐

检查依据：

- 2026 年 AI 应用开发 / Agent / AI 全栈公开岗位要求
- 2026 年公开 AI/Agent 实习面经
- 当前渔芯运行时代码，而不是只依赖设计文档

发现并修正的问题：

1. **Python 高频点不够全**
   - 原版缺少装饰器、生成器/迭代器、yield、深浅拷贝。
   - 近期 Python/AI 开发实习真实面经中仍会抽查这些基础。
   - 已补入，但控制在 L2～L3，不扩展到 CPython 底层。

2. **Flask 全栈面试防守不足**
   - 原版只讲 current_app，缺 Application Context / Request Context。
   - 缺少 CORS 与浏览器调通/接口排障场景。
   - 已补。

3. **SQL 基础存在低级风险**
   - 原版 JOIN/GROUP BY 有了，但缺 NULL、COUNT、WHERE vs HAVING。
   - 这些属于真实一面可能快速筛人的基础题。
   - 已补，并加入“查无投喂记录塘口”业务 SQL。

4. **模拟面试过于偏定义题**
   - 原版容易训练成“会背答案”。
   - 已新增：项目真实性、最难 Bug、重构、技术取舍、AI 生成代码验证、接口 500 排障。

5. **缺少限时 Coding**
   - 当前 AI 应用/全栈岗位仍可能出现 Python/SQL/简单算法题。
   - 已加入 15 分钟 Python Coding，并调整终测评分。

6. **发现一处重要项目事实错误**
   - 原课件根据 WRITE_CONTRACT 写成“commit 后回读”。
   - 当前 kernel/runner.py 实际是：事务内写入 → invariants → reload after → audit → commit → 返回 executed。
   - 幂等路径在业务 commit 后再 mark_completed。
   - 已按运行时代码修正课件，并在记忆基线记录“文档漂移”。

7. **Day 1 深度重新裁剪**
   - 不把 MVCC、Next-Key Lock、GIL、RAG、LangGraph 提前塞进 Day 1。
   - 这些保留后续日程。
   - Day 1 重点仍然是项目真实性 + Python/Flask/MySQL/事务。

终审结论：

Day 1 现在更接近真实 AI 全栈 / AI 应用开发一面结构：

~~~
自我介绍/项目
→ 项目连续追问
→ Python/Flask/MySQL 基础抽查
→ 一道 Coding/SQL
→ 一个故障场景
→ 继续项目取舍追问
~~~

不再把“答出定义”视为真正掌握。


## Day 1 第四轮终审：项目语义 + 2026 面试结构

本轮不再扩充大块知识，只处理高风险缺口。

### 项目语义修正

对照当前 production/capabilities.py 与 production/write.py：

- feeding.create：创建 draft 投喂单
- feeding.verify：真正执行物料库存扣减、库存流水与成本归集
- feeding.verify 不减少鱼的塘内存塘；出塘核验才是减少存塘的重要能力

课件已修正，禁止再用“创建投喂就立即扣库存记成本”的说法。

### 当前岗位/面经对齐结论

2026 年公开 AI 应用 / Agent 岗位与面经显示：

- Python 仍是硬基础
- FastAPI / Flask 属于常见 Web 工程要求
- RAG / Agent / MCP / Tool Calling / 向量库是核心方向
- Docker / API 封装 / 工程部署常作为基本或加分能力
- 一面明显偏项目深挖，而不是单纯八股
- AI Coding 真实性会被直接追问：哪些由 AI 生成、本人真正负责什么
- 仍可能出现简单 Coding、SQL 多表查询、慢查询/数据库优化
- 部分 Agent 岗位明确要求 asyncio / 异步编程

因此 Day 1 已补：

- async/await 防守级
- Application Factory / Flask Context / CORS
- SQL 参数化与注入防护
- SQL NULL / HAVING
- 15 分钟 Coding
- 故障排查
- AI Coding 如实回答边界

### 时间控制

新增内容全部分 A/B/C 优先级，不增加 Day 1 总时长。

若时间不足：
1. 保项目
2. 保 Flask 请求链
3. 保 MySQL/事务
4. 保 UnitOfWork
5. 再做 Python 防守题
6. 最后才做备用算法题

### 结论

Day 1 当前版本达到“真实项目事实 + 实习一面高频 + 五天时间约束”三者平衡。
后续不再对 Day 1 无限制扩知识点；正式学习中只根据用户答题暴露的弱项动态补充。
