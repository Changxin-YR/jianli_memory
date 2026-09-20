# Day 1｜渔芯项目 + Python / Flask / MySQL 面试基础

> 适用目标：AI 全栈开发实习面试冲刺  
> 学习周期：5 天中的第 1 天  
> 今日主项目：渔芯 AI 水产养殖一体化系统（GitHub：Changxin-YR/yuxin）  
> 今日原则：不追求知识百科全书，追求“项目能讲、原理能解释、连续追问不崩”。

---

# 0. 今天到底要达到什么程度

第一天结束以后，不要求你成为 Python、Flask 或 MySQL 专家。

你必须达到的状态是：

1. 面试官让你介绍渔芯项目时，可以不看资料连续讲 2～3 分钟。
2. 面试官沿着项目追问 Flask、HTTP、Python、SQL、事务时，你能继续回答。
3. 你知道这些技术在“自己的项目哪一层、哪个文件、解决什么问题”。
4. 不把别的项目技术栈串进渔芯。
5. 碰到 Day 2 才系统学习的并发、幂等、RBAC、DataScope，可以先说清基本概念，但不硬装深度。

今天统一采用四级掌握标准：

| 等级 | 要求 |
|---|---|
| L1 知道 | 能说出是什么 |
| L2 理解 | 能解释为什么需要 |
| L3 项目 | 能结合渔芯说明怎么使用 |
| L4 追问 | 能回答为什么这样做、替代方案和失败场景 |

今日核心内容全部至少达到 L3；Capability、请求链、事务、UnitOfWork 要达到 L4。

---

# 1. 今日时间表

推荐 8～10 小时有效学习时间。

| 阶段 | 建议时长 | 内容 | 合格标准 |
|---|---:|---|---|
| A | 1.5h | 渔芯业务与架构 | 能画业务图和系统图 |
| B | 2h | Python 项目必备基础 | 能看懂核心 Python 代码 |
| C | 1.5h | HTTP + Flask + REST | 能讲完整请求链 |
| D | 1.5h | MySQL + SQL | 能写基础查询并解释约束/索引 |
| E | 1.5h | 事务 + UnitOfWork | 能讲 commit / rollback / 边界 |
| F | 0.5～1h | 真实代码走读 | 能定位核心文件 |
| G | 1h | 模拟面试 + 复盘 | 总分达到 78/100 |

学习过程中不要一边看一边觉得“懂了”。每完成一个模块，都必须关掉课件，用自己的话重新说一遍。

---

# 2. 项目事实基线：先避免面试串栈

渔芯当前真实仓库是 Changxin-YR/yuxin。

当前项目核心技术栈：

- 后端：Python 3.11+ / Flask 3.1 / PyMySQL / MySQL 8+
- 前端：Vue 3 / TypeScript / Vite / vue-router
- Agent：DeepSeek Harness + Agent Gateway + Capability 派生的 Tool
- 测试：pytest / Vitest / Playwright
- 数据库访问：PyMySQL + 手写 SQL
- 事务：UnitOfWork 显式事务边界

这个项目明确没有使用：

- SQLAlchemy
- Alembic
- Redis
- FastAPI
- ORM

面试时如果问“你这个鱼塘项目为什么不用 ORM”，可以回答设计取舍；但不能说“我们这里用 SQLAlchemy”。

---

# 3. 第一模块：先把渔芯讲明白

## 3.1 项目一句话定义

渔芯不是“一个 Flask CRUD 项目”。

更准确的定义是：

渔芯是一套面向水产养殖企业的生产经营管理系统，把基地、塘口、养殖批次、投喂、仓储、采购、销售、成本、权限与审计串成完整业务链，并把同一套业务能力同时提供给人工页面和 AI Agent。

你要先讲业务，再讲技术。

面试官通常不关心你第一句话用了几个框架，他先判断你知不知道自己做的是什么。

---

## 3.2 业务主链必须会画

先把业务理解成下面这条线：

~~~
基地
 ↓
区域
 ↓
塘口
 ↓
养殖批次
 ↓
投苗 / 投喂 / 捕捞
 ↓
仓储库存
 ↓
成本归集
 ↓
销售 / 收款
~~~

外围支撑：

~~~
用户
 ↓
角色 / 权限
 ↓
DataScope
 ↓
审计
~~~

采购侧：

~~~
供应商
 ↓
采购单
 ↓
收货
 ↓
库存增加
 ↓
应付 / 付款
~~~

生产侧：

~~~
投喂
 ↓
消耗饲料库存
 ↓
产生库存流水
 ↓
归集养殖成本
~~~

销售侧：

~~~
销售单
 ↓
发货
 ↓
应收
 ↓
收款
~~~

今天不要求记住所有表名，但必须知道业务为什么彼此关联。

### 今日练习

不看课件，用 90 秒回答：

“如果一个养殖场今天采购 1 吨饲料，之后给 3 号塘投喂 50kg，这套系统大致会发生哪些业务变化？”

合格答案至少应该出现：

采购 / 入库 → 库存增加 → 投喂记录 → 库存扣减 → 库存流水 → 成本变化。

---

# 4. 第二模块：系统架构

## 4.1 人工页面调用链

第一天必须默写：

~~~
Vue 3
  ↓ HTTP
Flask Web 层
  ↓
Capability Route
  ↓
CapabilityRunner
  ↓
权限 / DataScope / 参数 / 不变量
  ↓
Domain Service
  ↓
UnitOfWork
  ↓
PyMySQL
  ↓
MySQL
~~~

这里最重要的不是背图，而是理解每一层职责。

### Vue

负责界面、用户交互、发 HTTP 请求。

### Flask Web 层

负责把 HTTP 世界翻译成应用内部调用，例如：

- 读取 path/query/body
- 读取 Cookie
- 处理 CSRF
- 生成 Response
- HTTP 错误转换

### Capability

描述“系统有什么业务能力”。

例如：

- pond.list
- pond.create
- batch.create
- feeding.create

### CapabilityRunner

执行公共业务控制流程。

### Domain Service

真正处理某个领域的业务。

### UnitOfWork

控制一次业务操作使用的数据库事务。

### PyMySQL

Python 和 MySQL 之间的数据库驱动。

### MySQL

持久化业务事实。

---

# 5. Capability Registry：Day 1 第一重点

## 5.1 为什么会有 Capability

普通项目可能把一个“创建塘口”操作写很多遍：

- Flask Route 写一份
- 权限配置写一份
- Agent Tool 写一份
- 前端字段写一份
- DataScope 写一份
- 幂等策略写一份

问题不是“重复代码不好看”。

真正的问题是：这些定义会漂移。

例如：

- 页面显示按钮，但 API 权限不同
- API 已经新增，Agent Tool 忘了新增
- 高风险操作后端要求确认，Agent 却没有确认
- 前端字段与后端字段不一致

渔芯的设计原则是：

> 一个业务能力只声明一次，其余内容从这一份声明机械派生。

---

## 5.2 Capability 中实际描述什么

真实 Capability 对象中会出现的关键信息包括：

- name：能力名
- title：展示名称
- domain：所属领域
- handler：业务处理器
- method：HTTP 方法
- path：HTTP 路径
- kind：read/create/update/delete/action
- required_permission：需要什么权限
- scope：数据范围策略
- risk：风险等级
- confirmation：是否人工确认
- agent_exposure：是否向 Agent 开放
- idempotent：是否要求幂等
- audit：审计策略
- invariants：业务不变量

例如 pond.create 的思想可以理解为：

~~~
pond.create
├─ POST /api/v1/ponds
├─ pond.create 权限
├─ DataScope
├─ 普通写风险
├─ 是否暴露为 Agent Tool
├─ 幂等
├─ 审计
└─ 业务不变量
~~~

---

## 5.3 面试回答模板

问题：

“你简历里的 Capability Registry 是什么？”

参考回答：

“我把系统中的业务操作抽象成 Capability，比如创建塘口、建立养殖批次、登记投喂。每个 Capability 不只是一个接口名字，它同时描述 HTTP 方法和路径、所需权限、数据范围、幂等、确认策略、审计以及是否允许 Agent 调用。之后 REST Route、权限码、Agent Tool Schema 和部分前端元数据都从这份声明派生。这样页面、API 和 Agent 不需要分别维护一套业务能力定义，可以降低规则漂移。”

追问：

“为什么不直接写 Flask Route？”

参考回答：

“如果系统只有人工页面，普通 Route 完全可以。但这个项目还有 Agent 入口，而且权限、DataScope、确认和幂等都要求两个入口一致。如果这些规则散落在 Route 和 Agent Tool 里，新能力很容易漏配，所以我把 Capability 当成能力层的单一事实来源，Route 只是它的 HTTP 适配结果。”

追问：

“这是不是过度设计？”

参考回答：

“如果项目只有十几个简单 CRUD，我不会专门做这一层。这个项目业务域比较多，而且同一能力同时服务页面和 Agent，所以重复维护的成本和安全风险更高。这里引入 Registry 是为了保证多个入口的一致性，而不是为了抽象本身。”

最后一句很重要：不要把所有架构模式都说成“越复杂越高级”。

---

# 6. 第三模块：Python，只学项目今天需要的

今天 Python 的目标不是刷语言八股，而是达到：

- 能读懂核心代码
- 能解释常见语法
- 面试官问基础时不掉分
- 能把 Python 概念映射到项目

---

# 7. list / tuple / dict / set

## list

有序、可修改，适合动态集合。

示例：

~~~python
permissions = ["pond.view", "pond.create"]
permissions.append("pond.update")
~~~

## tuple

有序，通常用来表达相对固定的一组值。

~~~python
fields = ("code", "name", "status")
~~~

## dict

key-value 映射。

项目里非常重要：

- API JSON
- 数据库查询行
- payload
- 配置
- query 参数整理

~~~python
pond = {
    "id": 1,
    "name": "3号塘",
    "status": "active"
}
~~~

## set

不重复集合，成员测试通常很快。

权限集合很适合 set / frozenset：

~~~python
permissions = {"pond.view", "pond.create"}
if "pond.create" in permissions:
    ...
~~~

### 掌握要求

达到 L3：

不仅能说区别，还能回答“为什么权限集合适合 set，而不是 list”。

答案核心：

权限判断主要是成员测试；set 不允许重复，平均情况下成员查询接近 O(1)。

---

# 8. 可变与不可变

常见可变对象：

- list
- dict
- set

常见不可变对象：

- int
- float
- str
- tuple（元素本身仍可能包含可变对象）
- frozenset

面试高频：

“为什么不要用可变对象做默认参数？”

危险写法：

~~~python
def add_item(item, items=[]):
    items.append(item)
    return items
~~~

默认对象在函数定义时创建，后续调用可能复用同一个 list。

正确思路：

~~~python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
~~~

今日掌握到 L2 即可。

---

# 9. is 和 ==

== 比较“值是否相等”。

is 比较“是不是同一个对象”。

典型：

~~~python
if value is None:
    ...
~~~

不要因为某些小整数/字符串存在缓存就把 is 当成值比较。

面试一句话回答：

“== 走值相等语义，is 比较对象身份；判断 None 通常使用 is None。”

---

# 10. 函数、参数、*args、**kwargs

普通函数：

~~~python
def create_pond(name: str, area_id: int) -> dict:
    ...
~~~

*args：

收集额外位置参数，得到 tuple。

**kwargs：

收集额外关键字参数，得到 dict。

项目代码里业务 handler 经常存在 **_ 或 **kwargs 一类写法，用于接收统一 Runner 传来的公共参数。

面试问题：

“*args 和 **kwargs 有什么区别？”

回答：

“*args 接收任意数量的位置参数，内部是 tuple；**kwargs 接收关键字参数，内部是 dict。它们适合做通用包装器或统一调用协议，但业务接口中如果参数固定，显式参数和类型标注通常更可读。”

---

# 11. 类、继承、classmethod、staticmethod

## 普通实例方法

第一个参数是 self。

## classmethod

第一个参数是 cls，更适合与类本身相关的构造或类级操作。

## staticmethod

没有自动 self / cls，本质上是放在类命名空间中的函数。

项目中读/写 Service 之间会共享部分实现。

面试问题：

“为什么 BatchWriteService 继承 BatchService？”

项目化回答：

“主要为了让读写服务复用相同的行级范围检查和数据装饰逻辑。相比单纯为了少写几行代码，我更关心的是读写路径使用同一套判定，避免两个实现慢慢漂移。”

---

# 12. dataclass

项目中的 ConnectionConfig 就使用 dataclass。

用途：

减少纯数据对象的样板代码，并让字段、类型、不可变性等意图更清楚。

~~~python
@dataclass(frozen=True)
class ConnectionConfig:
    host: str
    port: int
    user: str
~~~

你需要能解释：

- dataclass 适合“主要保存数据”的类
- frozen=True 表示不允许普通字段赋值修改
- slots=True 可限制动态属性并降低部分对象开销

slots 的底层细节今天不用深入。

---

# 13. 类型注解

常见：

- dict[str, Any]
- list[dict[str, Any]]
- Connection | None
- tuple[str, ...]
- frozenset[str]

面试问题：

“Python 是动态语言，为什么还写类型？”

回答：

“类型注解不会把 Python 变成静态语言，但它能提高 IDE 提示、静态检查、接口可读性和跨模块维护能力。这个项目业务域和统一调用接口比较多，所以明确参数和返回类型能降低接错接口的概率。”

---

# 14. try / except / finally / raise

理解执行关系：

~~~python
try:
    do_work()
except SomeError:
    handle_error()
finally:
    cleanup()
~~~

raise：

把异常继续抛给上层处理。

项目里不要每个地方都把异常吃掉。

一个异常如果代表业务失败，应该沿着统一异常协议上抛，在 Web 层转换成 HTTP Response。

面试问题：

“为什么不要 except Exception 然后什么都不做？”

回答：

“因为这会吞掉真实失败，让上层误以为操作成功，还会丢失排障信息。数据库和业务写操作尤其危险。”

---

# 15. with 与 context manager：今天 Python 最重要的一块

项目真实使用：

~~~python
with uow.begin() as tx:
    ...
~~~

理解 with：

“进入一个受控资源生命周期，退出时统一完成清理逻辑。”

文件：

~~~python
with open(...) as f:
    ...
~~~

数据库事务：

~~~
进入 with
  ↓
创建连接 / 开始事务

正常退出
  ↓
commit
  ↓
close

出现异常
  ↓
rollback
  ↓
close
  ↓
继续抛异常
~~~

项目中的 UnitOfWork.begin 使用 contextmanager 实现这一行为。

面试问题：

“with uow.begin() 为什么发生异常能够 rollback？”

回答核心：

“begin 是上下文管理器。进入时建立事务资源，yield 把 tx 交给业务代码；业务代码抛异常后，控制流回到 contextmanager 的 except 分支执行 rollback，再重新抛出；如果正常结束则在 else 分支 commit，最后关闭连接。”

这题要达到 L4。

---

# 16. Protocol

项目中 Cursor 使用 Protocol 描述最小接口。

你只需要理解：

Protocol 更强调“对象具备什么能力”，不要求必须继承某个具体基类。

这样业务代码可以依赖：

- execute
- fetchone
- fetchall

而不是依赖一个具体 Cursor 实现。

对测试尤其友好，可以做替身。

今天达到 L2～L3 即可。

---

# 17. Decimal

金额、成本等不应随意使用二进制 float。

~~~python
0.1 + 0.2
~~~

可能不是精确十进制 0.3。

Decimal 可以基于十进制字符串构造：

~~~python
Decimal("0.1")
~~~

面试回答重点：

“涉及金额和精确计量时，需要避免二进制浮点带来的表示误差，并且统一舍入规则，因此项目中使用 Decimal 一类精确十进制表示更合适。”

---

# 18. Python 今日面试题清单

必须秒答：

1. list / tuple / dict / set 区别是什么？
2. set 为什么适合权限集合？
3. is 和 == 的区别？
4. *args / **kwargs 是什么？
5. 什么是可变对象？
6. 为什么避免可变默认参数？
7. with 是什么？
8. context manager 是什么？
9. try / except / finally 怎么执行？
10. raise 的作用是什么？
11. classmethod / staticmethod 区别？
12. dataclass 用来解决什么问题？
13. 类型注解有什么价值？
14. Decimal 为什么比 float 更适合金额？
15. Protocol 大致解决什么问题？

今日不要花时间深挖：

- GIL
- CPython 对象模型
- GC 细节
- descriptor
- metaclass
- MRO 高级细节

除非实际面试岗位明确要求纯 Python 深度。

---

# 19. 第四模块：HTTP 与 Flask

## 19.1 一个 HTTP 请求里有什么

至少知道：

- Method
- URL
- Path
- Query String
- Headers
- Cookie
- Body

示例：

~~~
GET /api/v1/ponds?page=1&page_size=20
~~~

Method：GET  
Path：/api/v1/ponds  
Query：page=1&page_size=20

---

# 20. HTTP Method

常用语义：

- GET：读取
- POST：创建或执行动作
- PUT：整体替换式更新
- PATCH：部分更新
- DELETE：删除

注意：这只是语义约定，不是数学定律。

例如审批动作完全可能是：

~~~
POST /orders/123/approve
~~~

---

# 21. HTTP 状态码

第一天必须掌握：

| 状态码 | 含义 |
|---|---|
| 200 | 请求成功 |
| 201 | 创建成功 |
| 400 | 请求格式错误 |
| 401 | 未认证或认证无效 |
| 403 | 已知道是谁，但无权操作 |
| 404 | 资源不存在 |
| 409 | 业务状态/并发冲突 |
| 422 | 数据校验不通过 |
| 429 | 请求过于频繁 |
| 500 | 未处理的服务器错误 |

面试高频：

“401 和 403 区别？”

回答：

“401 主要是身份认证问题，例如没有有效登录凭据；403 是身份已经确认，但当前用户没有访问该资源或执行该操作的权限。”

---

# 22. Flask 是什么

合格回答：

“Flask 是 Python Web 框架，负责把 HTTP 请求映射到 Python 处理逻辑，并提供路由、request/response、应用上下文、请求钩子、错误处理等 Web 基础能力。我的渔芯项目里 Flask 主要处在 HTTP 适配层，核心业务规则不会直接写进 Route。”

---

# 23. 渔芯里的 Route 不是手写一堆装饰器

真实设计是：

~~~
Capability Registry
     ↓
遍历 capability
     ↓
app.add_url_rule(...)
     ↓
生成 Flask Route
~~~

Route 收到请求以后：

1. 取得当前用户 Actor
2. 读取 body / query / path params
3. 构造 Invocation
4. 调 CapabilityRunner.invoke
5. 把结果转成统一 JSON Response

所以如果面试官问：

“你 Flask 项目里的路由是怎么组织的？”

不要只回答“用了 Blueprint”。

这个项目更准确的特点是：

“业务路由由 Capability Registry 动态注册，Web 层保持较薄。”

---

# 24. request 中常用信息

要看懂：

- request.method
- request.args
- request.get_json()
- request.headers
- request.cookies
- request.path

分别对应：

- HTTP 方法
- Query 参数
- JSON Body
- Header
- Cookie
- 请求路径

---

# 25. before_request

项目中 before_request 做公共前置逻辑，例如：

- 绑定 request context
- 对非安全方法执行 CSRF 前置校验

为什么好？

因为如果 CSRF 检查散落到每个写接口，新增加一个接口时很容易忘记。

面试追问：

“所有逻辑都应该放 before_request 吗？”

不是。

公共的 HTTP 级横切逻辑适合放这里；真正业务规则仍应放业务/执行层。

---

# 26. after_request

项目用于统一给响应加信息，例如：

- X-Request-ID
- API Cache-Control

Request ID 的价值：

出现线上问题时，可以用同一个 ID 串起客户端报错和服务端日志。

---

# 27. 统一异常处理

项目存在 DomainError，以及 Flask errorhandler。

思想：

~~~
业务代码
  ↓
raise DomainError
  ↓
Flask error handler
  ↓
统一 JSON 错误响应
~~~

好处：

- 不需要每个 Route 重复 try/except
- 错误码和格式一致
- 业务层不关心 HTTP Response 如何构造
- 更容易统一日志与 request_id

面试问题：

“为什么不每个接口自己 try/except？”

回答时必须提到“职责分离”和“一致性”，不要只说“少写代码”。

---

# 28. Cookie / Session

Cookie：

浏览器保存并随请求携带的小段数据。

Session：

它表示一段登录会话。Session 的具体存储方式并不只有一种；在渔芯里，客户端携带会话令牌，服务端通过该令牌解析可信的用户、角色与权限信息。

渔芯的身份来源不能是前端传：

~~~
user_id=1
role=admin
~~~

因为客户端可以伪造。

更合理：

~~~
Cookie 中会话令牌
   ↓
服务端解析 Session
   ↓
得到可信 user
   ↓
得到 permissions / role / session
~~~

---

# 29. CSRF

CSRF = Cross-Site Request Forgery。

核心问题：

浏览器在向目标网站发送请求时可能自动携带 Cookie。

如果用户登录了站点 A，又打开恶意站点 B，B 可能诱导浏览器向 A 发起写请求。

因此使用 Cookie 会话的 Web 应用通常需要 CSRF 防御。

渔芯在 Flask 请求前置层对 POST / PUT / PATCH / DELETE 等非安全方法执行 CSRF 检查，并对少数有明确理由的路径做豁免。

今天达到 L2～L3，不深入 SameSite 等浏览器策略细节。

---

# 30. RESTful API

今天的回答标准：

“REST 更像一种资源化 HTTP 接口设计风格，常见做法是用 URL 表示资源、HTTP Method 表示操作，并合理使用状态码。它不是要求所有业务动作都只能 CRUD；审批、核验这类领域动作可以设计成明确的 action endpoint。”

不要回答成“REST 就是 GET 查、POST 增、PUT 改、DELETE 删”。

---

# 31. Flask 高频面试题

必须能回答：

1. Flask 是什么？
2. 一个请求从浏览器到业务代码经历了什么？
3. request 和 response 是什么？
4. GET / POST / PUT / PATCH / DELETE 常见语义？
5. 401 和 403 区别？
6. before_request 用在哪里？
7. errorhandler 为什么有价值？
8. Cookie 和 Session 区别？
9. CSRF 是什么？
10. 为什么业务逻辑不直接写 Route？
11. 为什么渔芯路由通过 Capability 注册？
12. Flask 和 FastAPI 有什么区别？

第 12 题今天达到基础回答：

“Flask 更轻量自由，生态成熟；FastAPI 原生围绕类型注解、Pydantic、OpenAPI 和 async 体验做得更完整。选哪个取决于项目，而不是谁绝对更高级。渔芯当前基线就是 Flask + PyMySQL，另一些项目才使用 FastAPI。”

---

# 32. 第五模块：MySQL 与 SQL

今天目标不是 DBA。

必须做到：

- 知道表和关系怎么表达业务
- 会写基本 SQL
- 知道主键/唯一约束/外键/索引的作用
- 能解释为什么数据库约束不是多余
- 为明天并发和锁打基础

---

# 33. 主键

主键用于唯一标识一行。

例如：

~~~
ponds.id
~~~

典型：

~~~sql
id BIGINT PRIMARY KEY
~~~

业务编码 code 可以有唯一约束，但通常不建议把“人类业务规则频繁变化的字段”随意当数据库主键。

---

# 34. 唯一约束

例如企业内塘口编号不能重复。

为什么不能只：

~~~
SELECT 是否存在
不存在就 INSERT
~~~

因为并发下：

~~~
事务 A 查询：不存在
事务 B 查询：不存在
A 插入
B 也插入
~~~

所以数据库 UNIQUE 是最终正确性防线。

应用层可以提前校验并生成友好错误，但不能替代数据库唯一约束。

这已经是非常典型的企业面试回答。

---

# 35. 外键 / 引用完整性

概念：

一条业务记录引用的对象应该真实存在。

例如投喂记录的 material_id 不应该指向不存在的物料。

实际企业项目是否使用数据库外键，需要考虑迁移、写入顺序、性能和架构约束；今天只需要理解关系完整性的意义，不要把“必须全部上外键”说成绝对规则。

---

# 36. 索引

核心回答：

“索引是额外的数据结构，用空间和写入维护成本换查询效率。”

为什么不全部字段建索引？

因为索引会：

- 占空间
- 增加 INSERT/UPDATE/DELETE 维护成本
- 低选择性字段单独建索引未必收益高
- 索引过多还会增加优化器和维护复杂度

索引设计通常看：

- WHERE
- JOIN
- ORDER BY
- 数据量
- 字段选择性
- 联合查询模式

B+Tree 和联合索引明天/后续补。

---

# 37. 必须会写的 SQL

查询：

~~~sql
SELECT id, name
FROM ponds
WHERE status = 'active';
~~~

排序：

~~~sql
SELECT *
FROM ponds
ORDER BY created_at DESC;
~~~

分页基础：

~~~sql
SELECT *
FROM ponds
LIMIT 20 OFFSET 0;
~~~

INNER JOIN：

~~~sql
SELECT
    p.id,
    p.name,
    a.name AS area_name
FROM ponds AS p
JOIN areas AS a
    ON p.area_id = a.id;
~~~

LEFT JOIN：

左表数据保留，右表匹配不到时对应列为 NULL。

GROUP BY：

~~~sql
SELECT
    pond_id,
    SUM(quantity) AS total_quantity
FROM feedings
GROUP BY pond_id;
~~~

Top N：

~~~sql
SELECT
    pond_id,
    SUM(quantity) AS total_quantity
FROM feedings
GROUP BY pond_id
ORDER BY total_quantity DESC
LIMIT 5;
~~~

---

# 38. INNER JOIN vs LEFT JOIN

面试回答：

“INNER JOIN 只返回两边满足连接条件的记录；LEFT JOIN 会保留左表全部记录，右侧匹配不到时补 NULL。选择取决于业务上是否允许左侧对象没有关联记录。”

一定要加最后一句业务语义。

---

# 39. 第六模块：事务——Day 1 第二重点

## 39.1 为什么有事务

投喂核验可能涉及：

1. 更新生产状态
2. 扣减物料库存
3. 写库存流水
4. 记录成本事实
5. 写审计

如果第 1、2 步已经提交，第 3 步报错，系统就出现部分成功。

业务需要：

~~~
全部成功 → commit
任意失败 → rollback
~~~

事务解决的是“一组必须保持一致的数据库操作”。

---

# 40. ACID

## Atomicity 原子性

事务中的操作要么全部成功，要么全部失败。

## Consistency 一致性

事务前后数据必须满足数据库约束和业务不变量。

注意：一致性不是数据库自动理解全部业务，很多业务规则需要应用层 + 数据库约束共同保证。

## Isolation 隔离性

并发事务之间尽量避免互相看到不该看到的中间状态。

## Durability 持久性

事务提交后，结果应该持久保存。

面试不要只背英文缩写，要用一个投喂/库存例子解释。

---

# 41. UnitOfWork

项目真实规则：

> 一个业务操作 = 一个 UnitOfWork = 最多一次最外层 commit。

它解决的问题是“事务边界应该由谁控制”。

错误方式：

~~~
InventoryRepository.update()
内部 commit

CostRepository.insert()
内部 commit
~~~

如果库存提交成功，成本失败，外层已经无法整体回滚。

正确思想：

~~~
with uow.begin() as tx:
    更新生产
    扣库存
    写流水
    写成本
    写审计
# 正常退出时统一 commit
~~~

任何一步异常：

~~~
rollback
~~~

所以 Repository/Service 的子步骤不应该随意各自提交。

---

# 42. autocommit=False

项目数据库连接明确关闭 autocommit。

意义：

SQL 执行后不会每条自动独立提交，而是由事务边界统一决定什么时候 commit。

这使业务可以：

- 一起提交
- 一起回滚

---

# 43. SAVEPOINT

SAVEPOINT 是事务内部的保存点。

用途：

在一个大事务内部，如果某个可选子步骤失败，只希望回滚到某个点，而不是整个事务全部回滚，可以使用保存点。

~~~
BEGIN
 A
 B
 SAVEPOINT s1
 C
 C 失败
 ROLLBACK TO s1
 D
COMMIT
~~~

渔芯 UnitOfWork 提供 savepoint。

今天知道语义即可，不要求手写复杂案例。

---

# 44. 事务是不是越大越好

不是。

长事务会带来：

- 更长锁持有时间
- 更多连接占用
- 并发降低
- 冲突/死锁风险提高
- 回滚成本增加

事务边界应该覆盖“必须一起保持一致”的业务单元，但尽可能短。

这是企业面试非常喜欢的回答。

---

# 45. 写操作真实性：Agent 项目的高价值设计点

虽然 Agent 深度在后面学，但今天先理解一个核心事实。

系统不能让模型自己宣布：

“已经成功创建/修改了。”

项目的写路径强调：

~~~
校验
 ↓
事务写入
 ↓
commit
 ↓
重新读取真实资源
 ↓
确认结果
 ↓
服务端返回成功事实
 ↓
模型只负责转述
~~~

核心原则：

> “执行了代码”不等于“业务事实已经正确落库”。

因此项目把成功状态建立在数据库提交与回读事实之上。

面试问题：

“Tool 已经返回 success，为什么还要回读？”

项目化回答：

“因为工具函数被调用只能证明代码路径执行过，不能完全代表最终业务状态就是预期状态。这个项目把执行与事实分开，写操作提交以后重新读取目标资源，用真实数据库状态构造结果；模型不能凭自然语言自行宣布成功。”

今天理解到 L3；Agent Gateway 细节 Day 3 再深入。

---

# 46. 第七模块：第一天真实代码走读

不要从仓库第一行看到最后一行。

严格按下面顺序。

## 46.1 README.md

目标：

关掉 README 后能够回答：

- 项目解决什么问题？
- 核心业务域有哪些？
- 技术栈是什么？
- “一个 Capability 只声明一次”是什么意思？

预计 20 分钟。

---

## 46.2 docs/ARCHITECTURE.md

只读以下主题：

- Capability 是唯一事实来源
- 分层与依赖方向
- 显式事务
- Agent 三层防御（今天只建立印象）

不要试图背完整文档。

预计 25 分钟。

---

## 46.3 backend/yuxin/web/app.py

搜索并找到：

- create_app
- before_request
- after_request
- add_url_rule
- request
- current_app
- errorhandler
- _current_actor
- _read_input

做一件事：

拿 pond.list 或类似请求，在纸上写出 Web 层发生什么。

预计 25 分钟。

---

## 46.4 backend/yuxin/kernel/uow.py

找到：

- ConnectionConfig
- UnitOfWork
- begin
- _commit
- _rollback_quietly
- savepoint
- query_one
- query_all
- execute
- translate_mysql_error

重点不是记代码。

重点是回答：

“如果 with uow.begin() 中间抛异常，后面发生什么？”

预计 30 分钟。

---

## 46.5 backend/yuxin/kernel/capability.py

今天只找：

- Risk
- Confirmation
- AgentExposure
- AuditPolicy
- Capability
- HandlerResult

重点理解：

Capability 本质上是“业务能力元数据 + 执行入口”的统一契约。

预计 30 分钟。

---

## 46.6 domains/master_data/capabilities.py

找到 pond.list / pond.get / pond.create。

逐字段解释：

- 为什么 GET/POST
- 为什么需要 required_permission
- scope 是什么概念
- risk 为什么不同
- agent_exposure 是什么
- idempotent 为什么写操作会关心

DataScope 与幂等实现明天深入。

预计 25 分钟。

---

## 46.7 domains/production/write.py

不要全部读。

只观察一个写流程的形状：

~~~
require permission
 ↓
scope concrete row
 ↓
状态/业务校验
 ↓
业务写
 ↓
返回 HandlerResult(resource_id=...)
~~~

看到跨域库存/成本调用时，只理解“生产域不应该自己偷偷写仓储和成本表”。

预计 25 分钟。

---

# 47. 企业面试：项目开场答案

先自己讲，再对照。

参考结构：

“渔芯是我做的一套水产养殖生产经营系统，业务上覆盖基地、塘口、养殖批次、投喂、仓储、采购、销售和成本。前端使用 Vue3 + TypeScript，后端使用 Flask + PyMySQL + MySQL。

这个项目比较核心的一点，是我没有把页面接口和 Agent 能力做成两套业务逻辑，而是把每个业务操作抽象成 Capability，例如创建塘口、建立批次、登记投喂。Capability 同时描述接口、权限、DataScope、幂等、确认、审计和 Agent 暴露策略，再由系统统一派生 REST Route 和 Agent Tool 等能力。

真正执行写操作时，业务最终进入 Domain Service 和 UnitOfWork，同一业务操作里的多条数据库写入由同一个事务控制，异常整体 rollback。这样人工页面和 Agent 最终执行的是同一套业务规则，而不是让模型直接操作数据库。”

注意：

不要逐字背。

最后必须能换自己的表达说出来。

---

# 48. 第一轮真实追问

下面全部先口头答，再看要点。

## Q1：为什么要做这个项目？

要点：

- 不是因为“想练 Flask”
- 是把养殖生产、库存、采购销售、成本这些关联业务统一管理
- 后期增加 Agent，但 Agent 不能成为第二套业务系统

## Q2：最有特点的设计是什么？

优先讲：

Capability 单一事实来源 + 页面/API/Agent 共用业务服务。

## Q3：Flask 在哪里？

Web/HTTP 适配层。

## Q4：业务逻辑在哪里？

domains 与统一 Runner/Kernel 约束，不在 Route 中堆积。

## Q5：数据库怎么访问？

PyMySQL + 手写 SQL + UnitOfWork。

## Q6：为什么不用 SQLAlchemy？

参考：

“这是这个项目的明确选型。项目需要非常明确地控制 SQL、事务和行级并发行为，所以选择 PyMySQL + 手写 SQL。代价是 CRUD 和映射代码会更多、开发效率不如 ORM；如果是业务模型变化快、复杂查询少的普通后台，我不会因为手写 SQL 更底层就认为它一定更好。”

这句话体现取舍，而不是贬低 ORM。

## Q7：为什么不能 Repository 自己 commit？

必须能完整回答，见事务章节。

## Q8：Agent 为什么不直接连数据库？

今天基础回答：

“模型不是可信执行主体。数据库写需要权限、DataScope、业务规则、事务、幂等和审计；如果模型直接发 SQL，这些边界很容易被绕过。因此 Agent 只能调用已经开放的业务能力。”

详细三层权限 Day 2/3 学。

---

# 49. Python 面试快速检查

关掉课件后回答：

1. dict 和 set 底层为什么适合快速成员查询？
2. tuple 和 list 怎么选？
3. is None 为什么比 == None 更常见？
4. **kwargs 得到什么？
5. dataclass 解决什么问题？
6. frozen dataclass 表达什么意图？
7. with 为什么能用于数据库事务？
8. finally 一定执行吗？（注意极端进程退出等情况，不要说“绝对任何情况”）
9. type hint 会强制运行时类型吗？
10. Decimal 为什么常用于金额？

达标：8/10。

---

# 50. Flask 面试快速检查

1. 一个 Flask 请求从进入到返回大致经历什么？
2. before_request 适合做什么？
3. 为什么业务规则不放 Route？
4. current_app 大致是什么？
5. 401 和 403 区别？
6. Cookie 与 Session 区别？
7. CSRF 的攻击成立条件是什么？
8. errorhandler 的价值？
9. REST 是不是等于 CRUD？
10. 渔芯为什么动态注册 Route？

达标：8/10。

---

# 51. MySQL 面试快速检查

1. 主键做什么？
2. UNIQUE 为什么比“先查后插”更可靠？
3. INNER JOIN 和 LEFT JOIN 区别？
4. 索引为什么能加速？
5. 为什么不能所有列加索引？
6. GROUP BY 做什么？
7. 事务为什么存在？
8. ACID 分别是什么？
9. commit 和 rollback 分别什么时候？
10. 为什么事务不能无限大？

达标：8/10。

---

# 52. 今日必须手写的 4 个题

不要复制。

## 题 1

写 SQL：统计每个 pond_id 的总投喂 quantity，按总量从高到低取前 5。

## 题 2

写 Python 函数：

输入 permissions 集合和 permission 字符串，返回是否拥有权限。

## 题 3

用伪代码写一个事务：

- 新建投喂记录
- 扣库存
- 写成本
- 任一步失败整体回滚

## 题 4

画完整调用链：

Vue → Flask → Capability → Runner → Domain → UnitOfWork → PyMySQL → MySQL

并口述每层职责。

---

# 53. 今日禁止死磕内容

今天遇到下面内容，只记录问题，不展开：

- MySQL MVCC 详细实现
- Repeatable Read 深层机制
- Next-Key Lock
- B+Tree 页分裂
- Python GIL
- CPython GC
- Flask 源码
- Agent Harness 进程池细节
- LangChain / LangGraph
- RAG
- MCP

原因不是它们不重要，而是 Day 1 的任务是搭稳项目后端骨架。

---

# 54. Day 1 最终模拟面试

建议在晚上完整做一遍，不能看答案。

## Part A：项目 30 分

1. 3 分钟介绍渔芯（8 分）
2. 画业务主链（5 分）
3. 画技术调用链（5 分）
4. Capability Registry 是什么（6 分）
5. 为什么这样设计（6 分）

合格：24/30。

## Part B：Python 20 分

随机抽 8 个：

- list/tuple/set/dict
- is/==
- mutable
- *args/**kwargs
- classmethod/staticmethod
- dataclass
- type hint
- with/contextmanager
- exception
- Decimal

合格：15/20。

## Part C：Flask 20 分

随机抽：

- 请求链
- request
- Route
- before_request
- errorhandler
- Cookie/Session
- CSRF
- 401/403
- REST
- 为什么薄 Web 层

合格：16/20。

## Part D：MySQL 20 分

随机抽：

- PK
- UNIQUE
- Index
- JOIN
- GROUP BY
- 事务
- ACID
- commit/rollback
- 长事务问题

合格：15/20。

## Part E：代码定位 10 分

不用精确行号，但必须知道：

- Flask HTTP 入口：backend/yuxin/web
- Capability 核心：backend/yuxin/kernel/capability.py
- 事务：backend/yuxin/kernel/uow.py
- 业务域：backend/yuxin/domains
- 生产写路径：backend/yuxin/domains/production/write.py

合格：8/10。

总分合格线：78/100。

---

# 55. 如果没过 78 分怎么办

不从头重学。

按失分最高的两个模块补。

例如：

- 项目 18/30 → 重新画架构 + 口述项目
- Python 17/20 → 不补
- Flask 12/20 → 重走 app.py
- MySQL 16/20 → 不补
- 代码 8/10 → 不补

只补项目和 Flask。

五天冲刺最怕“因为一个点不会，把整天重新学一遍”。

---

# 56. Day 1 完成后你应该能说出的 10 句话

1. 渔芯解决的是水产养殖企业生产经营管理，不是单纯 CRUD Demo。
2. 核心业务包括塘口、批次、生产、仓储、采购、销售和成本。
3. 后端当前真实栈是 Flask + PyMySQL + MySQL，数据库层手写 SQL。
4. Capability 是业务能力的统一声明。
5. REST Route、权限、DataScope、Agent Tool 等围绕 Capability 统一派生或执行。
6. Flask Web 层负责 HTTP 适配，不承载全部业务规则。
7. Domain Service 处理具体业务。
8. UnitOfWork 控制业务操作的事务边界。
9. 多个必须一致成功的写操作应该统一 commit 或 rollback。
10. Agent 不能直接操作数据库，必须经过受控业务能力。

如果这十句不能脱稿说出来，Day 1 还没有完成。

---

# 57. Day 1 与后续四天的连接

Day 1 建好了：

项目 → Python → Flask → MySQL → Transaction

Day 2 在这个基础上继续：

事务
 ↓
并发
 ↓
悲观锁 / 乐观锁
 ↓
幂等
 ↓
RBAC
 ↓
DataScope
 ↓
审计 / Human Confirmation

Day 3：

LLM
 ↓
Tool Calling
 ↓
Agent
 ↓
LangChain / LangGraph
 ↓
RAG
 ↓
MCP
 ↓
渔芯 / Smart Assistant

所以 Day 1 不需要提前抢 Day 2、Day 3 的内容。

---

# 58. 2026-09-21 正式开始时的执行方式

正式教学时不要从头朗读本课件。

每个模块执行：

1. 先由 ChatGPT 用 5～15 分钟讲清概念。
2. 立即定位渔芯真实代码。
3. 用户用自己的话复述。
4. ChatGPT 作为真实面试官提第一问。
5. 根据用户回答继续三层追问。
6. 暴露错误后立即纠正。
7. 用户重新回答。
8. 通过后才进入下一块。

学习过程要求“能输出”，而不是“看过”。

---

# 59. 第一日结束标志

只有同时满足下面条件，才标记 Day 1 完成：

- 项目介绍自然，不明显背稿
- 能画业务图
- 能画调用链
- Capability 能解释到设计动机
- Python 基础题至少 80%
- Flask 基础题至少 80%
- MySQL 基础题至少 75%
- UnitOfWork 能承受至少 3 层追问
- 能在真实仓库定位 5 个核心位置
- 综合测试 ≥ 78/100

完成后把成绩、错误题、薄弱点更新到本仓库 PROGRESS.md。

