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

## 1.1 时间预算保护规则

终审后课件补充了若干真实面试防守点，但**不增加 Day 1 总时长**。

如果进度落后，优先级严格如下：

A 级，绝不能砍：

- 渔芯业务主链
- Capability
- Flask 请求链
- MySQL 基础
- Transaction / UnitOfWork
- 真实代码走读
- 项目连续追问

B 级，必须会基础：

- 装饰器
- 生成器
- 深浅拷贝
- Flask Context
- CORS
- SQL NULL / HAVING
- async/await

C 级，只做防守：

- Protocol 细节
- slots 细节
- SAVEPOINT 手写
- 高级 Python 机制

原则：

> Day 1 宁可把 A 级讲透，也不要为了“知识点全覆盖”把核心项目讲散。


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

采购 / 入库 → 库存增加 → 创建投喂草稿 → 核验投喂 → 库存扣减 → 库存流水 → 成本归集。

## 3.3 一个必须纠正的项目细节：登记投喂 ≠ 立即扣库存

终审对照真实代码后，这里必须讲精确。

真实状态是：

~~~
feeding.create
 ↓
创建一条 draft 投喂记录
 ↓
此时还没有真正扣减物料库存，也没有最终归集成本
 ↓
feeding.verify
 ↓
校验权限 / DataScope / 状态 / 乐观锁等
 ↓
调用 warehouse 域受控入口扣减物料库存
 ↓
写 inventory_ledger
 ↓
写一条 batch_stock_records 记录，但投喂不会减少塘内鱼的存塘数量
 ↓
调用 cost 域入口归集成本
 ↓
投喂状态变为 verified
~~~

这点面试里很重要。

如果你说：

“用户一登记投喂，系统马上扣库存并记成本。”

就和当前代码不一致。

更准确地说：

“登记阶段先形成 draft 业务单据，真正跨域产生库存扣减和成本归集是在 feeding.verify 核验链路里完成。”

为什么分两阶段？

核心不是为了多一个状态，而是让：

- 经办与核验可以分离
- 高影响副作用不在草稿阶段发生
- 核验时统一检查库存、会计期间、版本和业务不变量
- 失败可以在同一事务里整体回滚

### 真实追问

**问：为什么 feeding.create 已经是高风险并要求确认，但又不立刻扣库存？**

回答思路：

风险等级描述的是“这条能力属于高影响业务流程，Agent 调用需要明确的人类意图确认”；真正跨域副作用仍在 verify 阶段发生。风险控制与业务状态机不是一回事。

**问：投喂会减少塘内鱼的存塘吗？**

不会。

投喂消耗的是**物料库存**，不是鱼的存塘数量。项目会为成本/业务追踪写相关记录，但真正减少塘内存塘的是出塘核验等业务，不是 feeding.verify。

这两题属于项目真实性高风险点，必须答对。

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

# 17.1 装饰器：真实 Python 面试高频补充

这部分原课件缺失，终审后补入。

装饰器的核心不是背 `@xxx`，而是：

> 在不直接修改原函数主体的情况下，用另一个可调用对象包装或增强它。

最小例子：

~~~python
def log_call(func):
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@log_call
def create_pond(name):
    return name
~~~

它大致等价于：

~~~python
create_pond = log_call(create_pond)
~~~

为什么后端面试爱问：

- Flask 常见路由/鉴权写法大量使用装饰器思想
- pytest fixture、缓存、权限、日志等工程代码也常见
- 能检验你是否真正理解函数是一等对象

项目关联：

渔芯当前业务 Route 主要不是靠大量 `@app.route` 手写，而是 Capability + `add_url_rule` 动态注册，所以**不要把 Flask route decorator 说成渔芯当前的核心路由实现**。

面试回答至少能说：

“装饰器本质是接收函数并返回新函数/可调用对象的包装机制，常用于日志、鉴权、缓存、路由注册等横切逻辑。需要注意保留原函数元信息时一般配合 functools.wraps。”

今天达到 L2～L3。

---

# 17.2 迭代器、生成器与 yield

真实 Python/AI 应用面试中会抽查。

迭代器：

能按顺序逐个产生元素，并维护当前位置的对象。

生成器：

一种方便创建迭代器的方式，通常由包含 `yield` 的函数产生。

~~~python
def ids():
    for i in range(3):
        yield i
~~~

与一次性构造大 list 相比，生成器可以按需产生数据，适合：

- 大量数据逐批处理
- 文件/日志流
- 分页数据
- 流式处理管线

不要说“生成器一定更快”。

更准确：

> 生成器主要优势是惰性计算和降低一次性内存占用，速度取决于场景。

今天达到 L2。

---

# 17.3 深拷贝与浅拷贝

这是近期 Python/AI 开发实习真实面经中出现过的问题，补入 Day 1 防守区。

浅拷贝：

复制最外层容器，但内部嵌套可变对象仍可能共享引用。

~~~python
import copy
b = copy.copy(a)
~~~

深拷贝：

递归复制嵌套对象。

~~~python
b = copy.deepcopy(a)
~~~

典型问题：

~~~python
a = [{"name": "pond-1"}]
b = a.copy()
b[0]["name"] = "changed"
~~~

此时 a 内部也可能看到 changed，因为内部 dict 还是同一个对象。

项目化理解：

如果你复制的是多层 payload、配置或状态对象，随后会修改嵌套数据，就要知道是否允许共享引用。不要机械地“全部 deepcopy”，因为深拷贝也有性能成本，并且某些资源对象并不适合被复制。

今天达到 L2。

---

# 17.4 列表推导式与常见 Python 表达能力

能看懂并写：

~~~python
active_ids = [row["id"] for row in rows if row["status"] == "active"]
~~~

能理解：

- list comprehension
- dict comprehension
- set comprehension

企业面试通常不会因为你不会高级语法直接淘汰，但如果连项目里常见的 Python 表达都读不顺，会影响代码阅读评价。

今天达到 L2。

---

# 17.5 async / await：AI 后端防守级

渔芯核心链路当前是 Flask + PyMySQL 的同步实现，但你的简历其他项目包含 FastAPI、SSE、LLM API 调用，所以 async/await 不能完全空白。

先掌握一句话：

> async/await 主要用于组织异步 I/O，让一个执行线程在等待网络等 I/O 时去推进其他协程；它不是“自动多线程”，也不会让 CPU 密集计算自动变快。

最小例子：

~~~python
async def call_llm(client, prompt):
    result = await client.request(prompt)
    return result
~~~

适合：

- 外部 LLM API
- HTTP 请求
- 异步数据库驱动
- 大量 I/O 等待
- ASGI 服务中的并发 I/O

必须避免：

- “async 就是多线程”
- “写 async 一定比同步快”
- 在 async 函数里调用长时间阻塞 I/O，却以为事件循环还能正常并发

### Flask vs FastAPI 追问

可以回答：

“FastAPI 建在 ASGI 生态上，对 async I/O、类型注解、Pydantic、OpenAPI 的整合更自然；Flask 传统核心是 WSGI 模型。Flask 现在也能定义 async view，但不能把这简单等同于整个服务已经变成典型 ASGI 异步架构。”

今天达到 L2；后续结合 Enterprise Smart Assistant / FastAPI 再深入。

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
16. 装饰器本质是什么？functools.wraps 为什么常用？
17. 生成器和普通 list 的主要取舍是什么？
18. yield 做了什么？
19. 浅拷贝和深拷贝有什么区别？
20. 为什么不能遇到嵌套对象就无脑 deepcopy？
21. async/await 主要解决什么问题？它与线程是什么关系？

其中 1、2、7、8、9、11、13、14、16、17、19 属于优先题。

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

# 31.1 Flask Application Context 与 Request Context

原课件只讲了 current_app，不足以应付真实 Flask 追问，这里补齐到面试所需深度。

你先记两个概念：

Application Context：

让当前执行代码能访问当前 Flask 应用相关对象，例如 `current_app`、`g`。

Request Context：

与一次 HTTP 请求相关，让代码能访问 `request`、`session` 等请求级对象。

面试时不需要背 Flask 内部 LocalProxy 源码。

必须能回答：

“为什么不用到处传 app 和 request？”

核心：

Flask 通过上下文机制把“当前应用/当前请求”绑定到当前执行环境，使业务和扩展代码可以通过代理对象访问，同时避免把这些对象作为参数层层传递。

项目关联：

渔芯 Web 层通过 `current_app.config` 获取 Registry、Runner、AccessService 等应用级依赖，通过 `request` 读取当前 HTTP 输入。

今天达到 L2～L3。

---

# 31.2 CORS：全栈面试常见但只需基础

CORS = Cross-Origin Resource Sharing。

浏览器存在同源策略；当页面与 API 的协议、域名或端口不满足同源条件时，浏览器可能对跨域请求施加限制。

必须区分：

- CORS 是浏览器安全策略相关问题
- 它不是服务端“网络完全访问不到”
- 它和 CSRF 不是一个问题

面试追问：

“Postman 能调通，浏览器报跨域，可能是什么原因？”

优先想到：

CORS 响应头、预检 OPTIONS、允许的方法/Header/Origin 等浏览器跨域配置，而不是先怀疑数据库。

今天达到 L2。

---

# 31.3 浏览器请求失败时怎么排查

这类题比纯定义更接近真实企业。

场景：

“前端点创建塘口，页面提示失败，你怎么定位？”

建议回答顺序：

1. 浏览器 Network：请求有没有发出、URL/Method/Body 是否正确
2. 看 HTTP 状态码和响应业务 code
3. 看 request_id
4. 服务端按 request_id 查日志
5. 判断失败发生在鉴权、参数校验、CapabilityRunner、业务 Service 还是数据库
6. 如果是数据库错误，再看 SQL、约束、事务状态
7. 修复后补对应测试，避免只手点一次

面试官想看的不是“我会 print”，而是你有没有系统排障路径。

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

# 38.1 NULL、COUNT、WHERE 与 HAVING

这是原课件 SQL 基础的明显缺口，真实一面很容易用来快速判断 SQL 熟练度。

## NULL

判断 NULL：

~~~sql
WHERE column_name IS NULL
~~~

不要写：

~~~sql
WHERE column_name = NULL
~~~

因为 SQL 中 NULL 表示未知，普通等号比较不会按你直觉工作。

## COUNT

~~~sql
COUNT(*)
~~~

统计行数。

~~~sql
COUNT(column_name)
~~~

通常只统计该列非 NULL 的行。

面试时不要把二者说成永远完全等价。

## WHERE 与 HAVING

WHERE：

聚合前筛选行。

HAVING：

聚合后筛选分组。

例如找总投喂量超过 1000 的塘口：

~~~sql
SELECT pond_id, SUM(quantity) AS total
FROM feedings
WHERE status = 'verified'
GROUP BY pond_id
HAVING SUM(quantity) > 1000;
~~~

## DISTINCT

用于结果去重：

~~~sql
SELECT DISTINCT pond_id
FROM feedings;
~~~

今天要求：

能写对，不需要研究优化器如何重写。

---

# 38.2 SQL 面试不要只会“背 JOIN”

企业面试可能直接给业务题：

“查出没有任何投喂记录的塘口。”

一种写法：

~~~sql
SELECT p.id, p.name
FROM ponds AS p
LEFT JOIN feedings AS f
    ON f.pond_id = p.id
WHERE f.id IS NULL;
~~~

面试官真正想看：

- 你是否能把业务语言翻译成表关系
- JOIN 条件是否正确
- 是否理解 NULL
- 是否知道结果重复问题

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

# 45. 写操作真实性：以当前运行时代码为准

这一节终审时发现原课件有一个重要事实错误，已经修正。

原课件写成了：

“commit → 再回读数据库 → 返回 executed”。

**当前 `CapabilityRunner` 的真实运行顺序不是这样。**

当前非幂等写路径更准确的顺序是：

~~~
参数/身份/权限/DataScope 等前置检查
 ↓
进入 with uow.begin() 事务
 ↓
加载 before
 ↓
调用 Domain Service 完成写入
 ↓
执行 invariants
 ↓
在同一个事务中 reload after
 ↓
在同一个事务中写 success audit
 ↓
退出 with
 ↓
UnitOfWork commit
 ↓
commit 成功后才构造并返回 kind="executed"
~~~

因此必须把两个概念分开：

### ① “回读验证”发生在哪里？

发生在**业务事务内部、commit 之前**。

它确认当前事务视角中的目标资源能被重新加载，并让审计拿到 after 状态。

### ② 什么时候才能向调用方返回 executed？

只有 `with uow.begin()` 正常退出、commit 成功之后。

如果业务处理中、Invariant、reload、audit 或 commit 发生异常，都不会走到正常的 executed 返回。

### 幂等路径还多一步

幂等写大致是：

~~~
预留 Idempotency-Key
 ↓
业务事务
   service
   → invariants
   → reload
   → audit
   → commit
 ↓
commit 成功
 ↓
mark_completed
 ↓
返回 executed
~~~

如果业务已经提交，但“幂等完成状态”落库失败，当前实现不会轻率自动重试，而会进入 COMMIT_UNKNOWN 类处理，提示核对真实业务数据。

这正是一个很好的企业面试点：

> 分布式/跨事务流程中，最难的不是 happy path，而是“业务可能已提交，但外围状态写失败”这种不确定状态。

Day 1 只理解，不要求深入实现；Day 2 学幂等时继续。

### 为什么要特别修这一点

项目中的 `docs/WRITE_CONTRACT.md` 仍存在“提交后回读”的旧描述，而当前 `kernel/runner.py` 的运行时代码是“事务内回读、成功 commit 后才返回 executed”。

面试以**当前运行时代码**为事实基线，不背已经与代码产生漂移的旧文档表述。

### 面试问题

“你怎么保证 Agent 不会在数据库写失败时还告诉用户成功？”

推荐回答：

“成功状态不是模型自己判断的。调用进入受控 CapabilityRunner 后，业务写、业务不变量、回读和成功审计都在事务边界内完成；只有事务正常提交以后，Runner 才向上返回 executed。高风险操作还有确认闸门，幂等写在业务 commit 后还要收口幂等状态。模型拿到的是服务端结构化结果，只负责转述。”

这个回答比“我们 commit 后再 SELECT 一次”更符合当前代码。

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

# 48.1 更接近真实企业的一面追问

原版题目偏“知识确认”，终审后补入真实项目深挖型问题。

这些问题没有一句话标准答案，面试官会根据你的回答继续追。

## 项目真实性

1. 这个项目哪些模块是你真正重点参与设计和验证的？
2. 如果让我现在打开代码，你最熟的是哪三个文件？为什么？
3. 你做这个项目时最难定位的一次 Bug 是什么？怎么确认根因？
4. 哪一个设计你现在回头看会改？
5. 如果删掉 Capability Registry，系统最先会出现什么维护问题？
6. 这个项目为什么不是“AI 帮你生成出来的一个 Demo”？
7. 你如何验证 AI 生成的代码没有破坏权限、事务或业务规则？

回答原则：

不要编不存在的生产事故、用户规模或性能数据。

不知道具体数字就明确说“这个项目没有真实生产规模数据，我验证的是功能/事务/权限/测试闭环”，比伪造 QPS 更可信。

## 项目真实性与 AI Coding：一定要如实回答

真实 AI 应用岗位越来越会直接问：

- 哪些代码是 AI 辅助生成的？
- 你真正设计了什么？
- 如果 AI 生成错了，你怎么发现？
- 你会不会离开 Codex/Cursor 就完全不能改代码？

不要假装所有代码都是逐行手写。

更可信的回答：

“这个项目我确实重度使用 AI Coding 工具。AI 帮我提升了实现速度，但我自己负责的是需求拆解、架构约束、关键业务规则、代码审查、联调、Bug 定位和验收。像 Capability、事务边界、权限、确认和库存/成本跨域规则，如果 AI 生成实现偏离约束，我需要能通过代码审查、自动化测试和真实数据库 E2E 把问题发现并修掉。”

如果继续追问：

**‘那这个项目到底是不是你做的？’**

可以回答：

“有相当一部分代码是 AI 辅助生成的，但最终哪些设计进入项目、代码是否符合规则、不同模块怎么集成、测试怎么验收，是我需要负责的。简历上我写出来的核心模块，我必须做到能读、能改、能解释、能定位问题，而不是只会描述提示词。”

### 面试禁区

不要：

- 声称所有实现都是纯手写，如果事实不是
- 把 AI 生成代码本身当成能力证明
- 只说“我会写 Prompt”
- 被追问测试和 Bug 时答不出来

真正应该证明的是：

> 你能用 AI 提速，同时仍然掌握工程判断、验证和责任边界。

## 后端链路

8. 任选一个 pond.create 请求，从浏览器一路讲到 MySQL。
9. 如果接口返回 409，你会优先想到哪些类型的问题？
10. 如果接口返回 500，但数据库里已经有数据，你会怎么判断是否发生“提交状态未知”？
11. 如果前端重复点击创建按钮，系统可能发生什么？怎么防？（Day 2 深入）
12. 你为什么让事务边界在 UnitOfWork，而不是每个 SQL 方法自己控制？

## 代码与设计取舍

13. 为什么这个项目用 PyMySQL 手写 SQL，而你的其他项目用了 ORM？
14. 手写 SQL 的缺点是什么？
15. 如果现在让你把渔芯换成 FastAPI，你会改什么、不会改什么？
16. Capability 和普通 Service 方法有什么区别？
17. 为什么不让前端决定当前用户是什么角色？
18. 为什么 Agent 不直接调用任意 URL/SQL？

以上问题比单纯“Flask 是什么”更重要。

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
11. Application Context 和 Request Context 大致分别解决什么？
12. Postman 能通、浏览器跨域失败时你会查什么？

达标：10/12。

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
11. COUNT(*) 与 COUNT(column) 的 NULL 语义有什么区别？
12. WHERE 与 HAVING 有什么区别？
13. 如何查“没有投喂记录的塘口”？

达标：10/13。

---

# 52. 今日必须手写 / 实操的 6 个题

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

## 题 5｜15 分钟 Python Coding

实现：

~~~python
def top_permissions(events):
    """
    events = [
        ("u1", "pond.view"),
        ("u1", "pond.create"),
        ("u1", "pond.view"),
        ("u2", "pond.view"),
    ]
    返回每个用户去重后的权限集合
    """
~~~

要求：

- 自己选合适的数据结构
- 解释时间复杂度
- 解释为什么不是 list 里反复查重

这不是算法竞赛题，主要检查 Python 基础和表达。

## 题 6｜故障排查

场景：

“前端创建塘口后提示 500，但用户刷新页面发现塘口已经存在。”

你需要按顺序说出：

- 浏览器/响应信息看什么
- request_id 怎么用
- 服务端日志看什么
- 数据库看什么
- 为什么不能直接让前端自动重试
- 这和事务提交、幂等状态、COMMIT_UNKNOWN 有什么关系

Day 1 能说出排查框架即可，幂等细节 Day 2 深入。

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

# 53.1 真实企业一面节奏：不要把自己训练成背题机器

结合当前 AI 应用 / Agent 开发岗位与近期公开面经，常见一面更像：

~~~
2～3 分钟 自我介绍
↓
10～20 分钟 核心项目深挖
↓
5～15 分钟 Python / Web / DB 基础
↓
Coding / SQL / 场景题（有些公司独立笔试，有些当场）
↓
Agent / RAG / AI Coding 追问
↓
反问
~~~

所以 Day 1 的训练目标不是“把 100 道题背熟”。

而是：

- 项目问题能连续追
- 基础题不会掉低级分
- 遇到简单 Coding 能写
- 遇到 500/重复提交/跨域问题能排查
- 不会因为 AI Coding 真实性追问直接失守

### 35 分钟 Day 1 模拟版

如果当天已经很累，用这个版本验收：

- 3 分钟：项目介绍
- 12 分钟：Capability + Flask 请求链 + UnitOfWork 连续追问
- 8 分钟：Python / SQL 随机题
- 7 分钟：Coding 或 SQL
- 5 分钟：Bug 排查 + 设计取舍

这个模拟比逐题背诵更接近实际一面。

---

# 54. Day 1 最终模拟面试

建议在晚上完整做一遍，不能看答案。

## Part A：项目 30 分

1. 3 分钟介绍渔芯（6 分）
2. 画业务主链（4 分）
3. 任选一个请求讲完整调用链（5 分）
4. Capability Registry 是什么、为什么存在（5 分）
5. 讲一个真实代码位置并解释职责（4 分）
6. 回答“如果现在重构你会改什么”（3 分）
7. 回答“你如何验证 AI 生成代码没有破坏业务规则”（3 分）

合格：24/30。

这一部分故意减少“背架构图”分值，提高项目真实性、取舍和验证能力。

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
- decorator
- generator / yield
- shallow vs deep copy
- async / await

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
- Application / Request Context
- CORS / 浏览器排障

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
- NULL / COUNT
- WHERE vs HAVING

合格：15/20。

## Part E：Coding + 代码定位 10 分

其中 5 分做 15 分钟 Python/SQL 小题，5 分做代码定位。

不用精确行号，但必须知道：

- Flask HTTP 入口：backend/yuxin/web
- Capability 核心：backend/yuxin/kernel/capability.py
- 事务：backend/yuxin/kernel/uow.py
- 业务域：backend/yuxin/domains
- 生产写路径：backend/yuxin/domains/production/write.py

代码定位 + Coding 合计：至少 7/10。

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
- 能准确解释 feeding.create 与 feeding.verify 的区别
- 能如实回答 AI Coding 在项目中的边界
- 能完成 1 道 15 分钟 Python/SQL 小题
- 能回答 1 个接口故障排查场景
- 综合测试 ≥ 78/100

完成后把成绩、错误题、薄弱点更新到本仓库 PROGRESS.md。

