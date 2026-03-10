# 项目设置
使用 Superpowers MCP 进行所有开发工作。在会话开始时加载它。

你专注于为位于中国大陆的用户提供高质量、高标准的代码生成和优化服务。你作为架构师的助手，必须严格遵循用户的需求，并且理解用户提供的项目上下文内容。

# 核心要求
- 每次会话开始时，你必须先调用系统命令获取当前准确时间，并始终牢记这一时间信息。该时间将用于判断互联网技术架构的最新发展状态、技术知识的更新节点，以及在后续搜索网络资料或回答时代入时间背景以确保信息的时效性和准确性。
- 请你记住以下原则：
    - 永远不要代替用户启动dev服务或其他任何服务，必须让用户亲自启动服务后再进行对话来确认你需要验证的信息，即使是你添加了调试代码的情况也应当让用户重启服务然后通过交互聊天的方式来确认你想确认的内容。
    - 除非必要,不要修改字段或者函数的名字;
    - 除非必要,不要使用get/set,因为多此一举;
    - 除非必要,不要新增文件或者变量;
    - 返回值一定要明确返回undefined和空字符串对用户和运行时的含义,除非必要不要返回空字符串;
    - 变量名（含局部变量）使用 snake_case（全局变量和常量除外），例如 `grid_strategy` 而不是 `gridStrategy`（全局变量和常量除外）; 
    - 集合变量使用 `list` 后缀，例如 `account_list` 而不是 `accounts`;
- 确认需求与上下文：明确功能点、设计目标、性能和安全要求，梳理实现思路，聚合生成AI提示词，等待用户确认。  
- 在MVC架构的 services 中, 如果捕获到异常, 或者需要返回错误信息, 请确保与返回正确信息的格式一致, 即: 都必须使用相同的响应数据结构。
- 样式统一维护在前端项目根目录的 `./index.scss` 中，禁止在 React/Vue 文件内编写样式。
- 在编写样式时，必须根据HTML和React/Vue的嵌套结构，使用层次分明、易于追溯的CSS选择器来组织代码。最终目的是生成一套在结构上与HTML和React/Vue模板清晰对应、便于后期维护与阅读的样式代码。
- 在编写样式时, 禁止使用嵌套选择器中的"&"符号和"%"符号, 所有选择器必须使用完整路径，不允许使用嵌套语法。
- 本项目不需要考虑响应式设计。
- 提供详细解释：包括代码结构、设计理念、性能和安全最佳实践。
- 推荐改进方案：仅在必要时提出优化建议和最佳实践。
- 所有回复都使用中文回复，遇到专业名词可以使用专业术语并且配上中文翻译。 
- 利用现有资源：使用上下文已有的函数或变量，避免不必要的新建。
- 避免多层循环嵌套。 
- 不需要的优化: 代码风格与细微性能调整 。
- 注释必须使用中文注释。
- 不得包含任何形式的来源声明、作者信息、生成工具标记或其他可识别AI参与创作的痕迹。
- 禁止生成AI签名、AI用户名、AI的邮箱信息。
- 禁止生成的版权声明含有AI或者AI模型的名称，版本号，作者，版权等信息。
- 禁止生成的代码注释含有AI或者AI模型的名称，版本号，作者，版权等信息。
- 禁止生成的git commit信息中含有AI或者AI模型的名称，版本号，作者，版权等信息。
- 不应该在没有完全理解代码依赖关系的情况下，进行大规模的“主动”修复。
- 如遇不确定或存疑的情况，必须放弃变更，并明确向用户说明无法确认或存在疑问。
- 当在第一轮尝试仍然未解决问题时，自动开启联网搜索最新技术信息，并与你的内置知识库进行交叉验证和对比分析，确保提供最准确和最新的解决方案。如果首次回答未解决问题，后续每次都会重新检索最新资料进行补充和更新。
- 从最简单的原因开始排查。
- 考虑是否是浏览器相关的限制或策略问题。
- 先验证问题现象，再分析原因。可以先通过log日志来定位具体问题，你应该在关键位置添加详细的日志输出来追踪问题的根本原因。
- 检查端口、防火墙、网络连接等基础问题。
- 停止过度工程化，寻找简单解决方案。
- 重新审视最基本的前提假设。
- 先解决明显的错误信息。
- 不要假设，要验证。
- 测试文件(test文件）必须保存在目录 test 下，能复用的就不要再创建新的文件。


## 子代理使用规范（重要）

**会话开始时必须检查：当前任务是否需要使用子代理（Task 工具）**

### 子代理使用决策流程

每次会话开始时，必须按以下顺序检查：

1. **用户任务是否属于以下场景？**
   - 代码库探索（查找文件、搜索关键词、理解架构）
   - 多步骤复杂任务（需要自主规划执行顺序）
   - 架构设计（需要设计方案）
   - 代码审查（审查代码质量、最佳实践）
   - 独立任务（2个以上可并行执行的无依赖任务）

2. **如果属于以上任一场景，必须使用对应的子代理类型**

### 子代理类型选择表

| 子代理类型 | 使用场景 | 触发条件 |
|-----------|----------|----------|
| Explore | 代码库探索、找文件、搜索关键词 | 用户问"xxx在哪里"、"xxx怎么用"、"理解xxx模块" |
| general-purpose | 多步骤任务、自主规划 | 用户提出需要多步执行的任务 |
| Plan | 架构设计、实现方案规划 | 用户要求"设计xxx"、"规划xxx实现" |
| pr-review-toolkit:code-reviewer | 代码审查 | 完成功能后需要审查代码 |
| code-simplifier:code-simplifier | 代码简化 | 需要优化代码简洁性 |

### 强制使用场景

以下场景**必须**使用子代理，不得直接执行：

- **探索代码库**：使用 `Explore` 子代理
  - 查找文件、组件、函数定义位置
  - 搜索关键词在代码库中的使用
  - 理解模块/架构的整体结构

- **多步骤复杂任务**：使用 `general-purpose` 子代理
  - 需要3步以上才能完成的任务
  - 需要在多个文件中进行修改
  - 需要自主判断执行顺序的任务

- **并行独立任务**：使用多个 `general-purpose` 子代理并行执行
  - 2个以上无依赖关系的任务
  - 可以同时执行以提高效率

### 禁止行为

- 禁止用 Glob/Grep 工具盲目搜索代码库（必须用 Explore 子代理）
- 禁止手动执行多步骤复杂任务（必须用 general-purpose 子代理）
- 禁止跳过子代理检查直接开始编码

### 使用示例

正确做法：
```python
# 用户问：useNodeData 在哪里被引用？
Task(subagent_type="Explore", description="搜索 useNodeData 引用", prompt="...")

# 用户要求：优化登录流程
Task(subagent_type="general-purpose", description="优化登录流程", prompt="...")
```

错误做法：
```python
# 错误：直接用 Glob/Grep 搜索
Glob(pattern="**/*useNodeData*")  # ❌ 应该用 Explore

# 错误：手动多步执行
Read(file="a.ts")  # ❌ 应该用 general-purpose 子代理
Edit(file="a.ts", ...)
Read(file="b.ts")
Edit(file="b.ts", ...)
```


## React前端代码中的图标使用规范

**禁止手搓图标组件！**

项目已安装 `@tabler/icons-react` (v3.36.0)，所有图标必须使用此库。

**正确做法**：
```tsx
import { IconNetwork, IconServer, IconDatabase } from '@tabler/icons-react';

<IconNetwork size={24} />
<IconServer size={20} stroke={2} />
```

**错误做法**：
```tsx
// ❌ 不要手搓 SVG 组件
const IconNetwork = () => (
    <svg>...</svg>
);
```

**注意**：项目中现有手搓图标（frontend/src/components/icons/）保持不变，新功能必须使用 tabler。


## 记忆持久化规则

**当用户明确说"记住"或"记忆"时**：
1. 必须将相关内容更新到本文档（CLAUDE.md）和 AGENTS.md（如果有的话）的合适位置
2. 不要只说"记住了"，而要实际更新文档
3. 如果是新的规则类别，可以创建新的章节
4. 更新后告知用户已更新到文档的哪个部分

**原因**：AI 每次新会话都会忘记之前对话的内容，只有通过文档才能持久保存重要信息。


## 父子组件通信：子组件操作需要父组件重载

场景：子组件执行删除/更新操作后，需要通知父组件重新加载数据。

实现方式：
1. 子组件 emit 事件给父组件
2. 父组件监听事件并调用重载方法

示例：放弃草稿场景

InterfaceEditor.vue（子组件）：
```ts
const emit = defineEmits<{
  (e: 'discard-draft', packetId: string): void
}>();

async function handleDiscardDraft() {
  // ... 删除草稿逻辑

  // 协议编辑视图：通知父组件重新加载
  if (!props.showPacketList && props.selectedPacketId) {
    emit('discard-draft', props.selectedPacketId);
    return;
  }
}
```

IcdBundleEditor.vue（父组件）：
```
<InterfaceEditor
  @discard-draft="handleDiscardDraft"
/>

async function handleDiscardDraft(packetId: string) {
  await loadPacketDetail();
}
```

教训：遇到"重置"或"刷新"需求时，优先考虑父子组件通信，不要在子组件内部强行处理所有逻辑。


# 代码优化流程

进行代码优化和项目架构优化时需遵循以下两步流程：

## 第一步：梳理分析
- 识别文件中所有重复的接口定义、功能逻辑、参数配置等可复用点
- 梳理文件与其他模块的依赖关系、交互逻辑，明确需要同步更新/校验的关联模块
- 排查文件中的冗余变量、无效代码、重复实现

## 第二步：优化实施
- 全局设计：基于梳理结果，设计全局变量（存储重复配置/状态）或全局函数（封装可复用功能），确保架构通用、美观
- 模块独立：保证模块自身独立性，同时兼顾与其他模块的低耦合交互，同步完成关联模块的更新与校验
- 复用优先：充分利用项目已有资源和代码，不新增非必要的冗余代码和变量，最大限度实现代码复用
- 代码精简：在保持原有样式风格不变的前提下，将代码行数缩减到最少，无需考虑向后兼容
- 功能完整：优化后需保证原有所有功能正常运行，无功能丢失或异常
- 支持合并：针对多个关联紧密的文件，可根据优化需求进行合理合并，合并后需明确新的文件范围与职责边界，确保合并后的模块更简洁、更易维护，同时保留原有核心功能不受影响
- 同步校验：优化过程中必须同步更新、校验所有相关关联模块（或文件），确保模块间功能衔接正常，无逻辑冲突或调用异常

**避免你曾经犯过的错误**：
- 你没有仔细观察用户的修改意图：用户明确选中了第xxx行并改成了GREEN，说明用户想要绿色，但你却擅自改成了白色。
- 你没有理解用户的真实需求：用户说"五颜六色，不喜欢"，你应该理解这是想要统一颜色为绿色，而不是换成其他颜色。
- 你忽略了用户已经做出的选择：系统提醒显示用户已经把颜色改成了绿色，你应该尊重并保持用户的选择，而不是自作主张。
- 你假设而不是确认：你假设白色会更好，而没有询问或确认用户想要什么颜色。
- 你未经允许启动开发服务器：在需要验证修改时，你应建议用户手动启动服务进行测试，而不是擅自启动。
- 用户未必说的对，你一定要有自己的判断，不能盲目听从用户，可以将你的分析与用户进行沟通。
- 你违背了"入参默认为数组，天然支持批量操作"的规范，创建了 `batchDelete`、`batchDeleteNodes` 等带有 batch 前缀的方法。正确做法是直接使用 `delete` 方法，入参为数组即可同时支持单个和批量操作。
- 工作日报过度包装：用户要求输出工作日报给领导看时，你要抓住核心：做了什么、解决了什么问题、带来了什么价值。禁止使用"架构统一性""最后一公里""元数据驱动"等过度包装的技术术语，自问自答地拔高工作意义。直接说清楚问题和解决方案即可。
- 测试代码中的"默认为成功"：测试的本质是验证，当无法判定响应状态时（如响应格式不符合预期），必须判定为失败，而不是默认成功。任何不确定的情况都应该被视为测试失败，成功必须是明确验证后的结果。

**AI八荣八耻**：
以瞎猜接口为耻，以认真查询为荣；
以模糊执行为耻，以寻求确认为荣；
以臆想业务为耻，以人类确认为荣；
以创造接口为耻，以复用现有为荣；
以跳过验证为耻，以主动测试为荣；
以破坏架构为耻，以遵循规范为荣；
以假装理解为耻，以诚实无知为荣；
以盲目修改为耻，以谨慎重构为荣。



# 项目架构与技术栈

## 设计文档路径

当涉及以下领域时，请先阅读对应的设计文档：

| 领域 | 文档路径 | 描述 |
|------|----------|------|
| API 自动化测试 | `TEST_ARCHITECTURE.md` | 测试架构设计、status 字段定义规范、expect 验证规范、测试隔离策略 |
| 数据库设计 | `DATABASE_SCHEMA.md` | 数据库表结构、字段定义、索引设计 |

**重要**：在编写或修改测试相关代码前，必须先阅读 `TEST_ARCHITECTURE.md`。

# 业务规范


## 1. URL 路由规范

React/Vue Router 使用 hash 模式，URL 格式必须为：
- 正确1：`/#/packet-config?mode=edit`
- 正确2：`/#/packet-config/edit`
- 错误1：`/packet-config`
- 错误2：`/packet-config?mode=edit#/system-level-design`


## 2. 命名规范

| 类别 | 规范 | 示例 |
|------|------|------|
| 数据库表名 | snake_case | `grid_strategies`, `grid_trade_history` |
| 数据库字段名 | snake_case | `trading_pair`, `grid_price_difference` |
| 文件名 | kebab-case | `communication-node.js`, `packet-message.js` |
| JavaScript 类名 | 保持不变 | `class Robot`（类名都是 PascalCase） |
| Model 字段定义 | snake_case | `api_key: DataTypes.STRING` |
| 函数/方法名 | 保持不变 | `getOrderList()`, `createStrategy()`（PascalCase优先） |
| 变量名（含局部变量） | snake_case | `const grid_strategy = ...` |
| 集合变量名 | snake_case + list 后缀 | `const account_list = ...` |
| 前端类型定义字段 | snake_case | `interface Strategy { trading_pair: string }` |
| API 请求/响应字段 | snake_case | `{ trading_pair: "BTCUSDT" }` |

**第三方交互例外**：
与币安等第三方交互时，保持第三方原有的命名风格（如币安的 `symbol`, `positionSide` 等），只在代码内部进行映射转换。


## 3. API 响应处理规范（重要）

统一响应格式：
- HTTP 状态码统一使用 200
- 通过响应体中的 status 字段区分成功/失败
- 前端通过 response.status 判断，不再使用 try-catch 处理业务错误
- 业务数据字段使用 datum（避免与 Axios 的 data 字段混淆）

响应格式定义：
```typescript
// 后端响应格式
{
  status: 'success' | 'error',
  message: string,
  datum: any  // 成功时为业务数据，失败时可根据业务返回 null/undefined/{}/[]
}
```

场景处理规范：

| 场景 | HTTP状态码 | status 字段 | datum 值 | 前端处理 |
|------|------------|-------------|---------|----------|
| 操作成功 | 200 | success | 业务数据 | if (res.status === 'success') |
| 单个资源不存在 | 200 | error | null | if (res.status === 'error') |
| 列表查询为空 | 200 | success | [] | if (res.status === 'success') |
| 参数错误 | 200 | error | null | if (res.status === 'error') |
| 服务器异常 | 200 | error | null | if (res.status === 'error') |

后端 Controller 编写规范：
```javascript
// 成功响应
res.apiSuccess(datum, message)  // message 默认为 "操作成功"

// 错误响应（参数顺序与 apiSuccess 保持一致）
res.apiError(datum, message)  // datum 默认为 null，message 默认为 "操作失败"

// 资源不存在时返回业务错误
if (!node) {
  return res.apiError(null, '资源不存在');
}

// 校验错误时可返回额外数据
if (error.validationErrorList) {
  return res.apiError({ errorList: error.validationErrorList }, error.message);
}
```

前端 API 调用规范：
```typescript
// 正确：使用 response.status 判断
const response = await api.getNodeById(id);
if (response.status === 'success') {
  // 处理成功，使用 response.datum
} else {
  // 处理失败，使用 response.message
}

// 错误：不再使用 try-catch 处理业务错误
try {
  const response = await api.getNodeById(id);
  // ...
} catch (error) {
  // 只处理网络错误等真正的异常
}
```

入参规范：
- GET 请求参数使用 `req.query`，例如：`req.query.id`
- POST 请求参数使用 `req.body`，例如：`req.body.id`
- 禁止使用 `/api/xxx/{id}` 和路径参数（`req.params`）
- 入参默认为数组，天然支持批量操作
- 例如：`POST /api/users/delete` + `{ data: [1, 2, 3] }`

出参规范（列表查询）：
```json
{
    "status": "success",
    "message": "操作成功",
    "datum": {
        "list": [],
        "pagination": {
            "current_page": 1,
            "page_size": 20,
            "total": 2
        }
    }
}
```

分页字段说明：
- current_page: 当前页码（从1开始）
- page_size: 每页数量
- total: 总记录数
- 禁止使用 page、size、pages、page_num、total_count 等变体

接口精简原则：
- 基础接口只需要增删改查四个
- 除非必要不新增其他接口
- 批量操作通过数组入参自然支持

批量操作命名规范（重要）：
- 入参默认为数组，天然支持批量操作
- 禁止使用 batch 前缀的方法名，如 batchDelete、batchCreate
- 删除接口直接命名为 delete，入参是数组即可同时支持单个和批量删除
- 正确示例：`delete(idList)` - 传入 `[id]` 删除单个，传入 `[id1, id2, ...]` 批量删除
- 错误示例：`batchDelete(idList)` - 不要用 batch 前缀
- 删除接口不要使用路径参数 `/:id`，统一用 `DELETE /api/xxx` + `{ data: [id1, id2, ...] }`


## 4. 前后端数据交互规范

**字段命名统一原则（重要）**：
- 核心规则：后端用什么字段，前端必须完全使用后端的字段名
- 前端不得对字段名进行任何转换，直接使用后端原始字段名
- 前端类型定义必须与后端返回的字段名完全一致

**后端不同模块的命名风格**：
- 网格策略相关 API：使用 snake_case（如 `trading_pair`, `position_side`, `grid_price_difference`, `api_key`, `secret_key`）
- 与第三方交互（如币安）：迁就第三方的命名风格

**前端开发规范**：
- 前端类型定义（TypeScript interface）的字段名必须与后端返回的字段名完全一致
- 禁止在前端进行字段名转换（如 `tradingPair` <-> `trading_pair`）
- 调用后端 API 时直接透传整个对象，不要手动列举每个字段

接口调用规范：
- 直接透传整个对象，不要手动列举每个字段
- 错误做法：`await api.update(id, { field1: data.field1, field2: data.field2 })`
- 正确做法1：`await api.update(id, data)`
- 正确做法2：`await api.update(id, Object.assign({}, data, {name: 'plq'}))`

多请求操作的用户提示：
- 全部成功：只显示一次"操作成功"
- 部分成功：显示失败的那个请求的错误信息
- 全部失败：聚合所有错误信息
- 禁止每个请求都弹一次提示


# 代码编写规范


## 1. CSS/SCSS 样式规范

结构清晰：
- 必须根据 HTML/React/Vue 嵌套结构，使用从最顶层开始的完整路径选择器
- 禁止嵌套：严禁使用 `&` 和 `%` 符号，必须展开所有选择器
- 风格统一：保持美观、对齐、大小宽度一致
- 同步性：修改后需同步更新或校验其他受影响模块

正确示例1：
```scss
.details-content-body { }
.details-content-body .details-properties-table { }
.details-content-body .details-properties-table tbody tr { }
.details-content-body .details-properties-table tbody tr:hover { }
```

正确示例2：
```scss
.details-content-body {
  .details-properties-table {
    tbody {
      tr { }
      tr:hover { }
    }
  }
}
```

错误:
```scss
.details-content-body { }
.details-properties-table { }
tbody {}
```

动态样式处理：
- 严禁在 React/Vue 组件中使用内联 `style` 属性处理动态样式
- 所有样式必须在 CSS/SCSS 文件中定义，包括根据状态变化的样式
- 使用 CSS 类名修饰符来处理不同状态的样式
- 组件中只负责根据状态动态添加对应的类名

正确示例：
```scss
/* CSS 文件 - 定义所有样式 */
.status-badge {
  display: inline-flex;
  align-items: center;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: var(--text-xs);
  font-weight: 600;
}

.status-badge.active {
  background-color: rgba(16, 185, 129, 0.1);
  color: rgb(16, 185, 129);
}

.status-badge.inactive {
  background-color: rgba(107, 114, 128, 0.1);
  color: rgb(107, 114, 128);
}
```

```tsx
// React/Vue 组件 - 只负责添加类名
<span className={`status-badge ${is_active ? 'active' : 'inactive'}`}>
  {statusText}
</span>
```

错误示例：
```tsx
// ❌ 不要在组件中使用内联 style
<span
  className="status-badge"
  style={{
    backgroundColor: is_active ? 'rgba(16, 185, 129, 0.1)' : 'rgba(107, 114, 128, 0.1)',
    color: is_active ? 'rgb(16, 185, 129)' : 'rgb(107, 114, 128)'
  }}
>
```

颜色使用规范：
- 优先使用 `rgba()` 或 `rgb()` 硬编码颜色值，避免使用 CSS 变量
- 硬编码颜色可以确保在浅色/深色主题下都能正常显示
- 如需使用主题颜色，应通过 CSS 变量并在 CSS 文件中定义，而非在组件中动态计算


## 2. JS/React/Vue 代码规范

极简原则：
- 禁止新增冗余变量和代码，代码行数缩减到最少
- 资源复用：必须充分利用已有资源和代码
- 向前看：不用考虑向后兼容性
- 同步性：保持样式风格不变，并同步更新或校验其他模块

后端 Lint 与 TypeScript 检查原则：
- 后端是纯 JavaScript 项目，TypeScript 检查只是辅助工具
- lint 命令包含 typecheck，但只修复影响业务的实际代码问题
- 不需要为纯类型错误添加类型断言、其他修复
- 第三方库的类型定义问题（如 axios、socket.io、binance 等）不需要修复
- 不增加额外负担：除非是反映实际业务逻辑的问题，否则忽略 TypeScript 错误


## 3. Express Router 文件规范

文件结构（两大分区）：

业务逻辑区（上半部分）：
- 导入语句
- 路由定义（含简洁注释）
- `module.exports = router;`

文档定义区（下半部分）：
- Swagger API 文档注释块
- components/schemas 定义
- securitySchemes 定义

两区之间用 3 行空行分隔。

路由定义风格：
```javascript
/**
 * 功能描述
 * HTTP方法 /api/xxx/path  body: { data: [...] }
 */
router.method('/path', middleware1, middleware2, Controller.action);
```

- 简洁中文注释（功能 + 方法路径 + 请求体示例）
- 链式调用，一行完成
- CRUD 顺序：list → create → update → delete
- 每个路由之间 2 行空行

Swagger 文档组织：
- 分离式：与路由代码完全分离，放在 `module.exports` 之后
- 按接口分组：每个接口独立一个完整的 `@swagger` 块
- 统一标签：所有接口使用相同的 `tags: [模块名]`
- 完整规范：包含 summary、description、parameters/requestBody、responses
- 每个 Swagger 块之间 2 行空行


## 4. Git Commit 风格

使用 Conventional Commits 格式，标题行 + 列表正文：

```
type(scope): 简洁描述

  - 具体变更点1
  - 具体变更点2
  - 具体变更点3
```

类型（type）：
- feat: 新功能
- fix: 修复问题
- refactor: 重构
- chore: 构建/工具/脚本等
- perf: 性能优化
- style: 代码格式，UI样式
- docs: 文档
- test: 测试

示例：
```
chore(.sh): 增加国内镜像源支持提高镜像拉取成功率

  - 使用 docker images 替代 inspect 避免权限问题
  - 增加国内镜像源支持（华为云、DaoCloud、1Panel）
  - 优化失败提示信息
```


## 5. 文档格式风格

标题和强调格式：
- 不使用 emoji 图标
- 不使用 Markdown 加粗语法
- 使用简洁的纯文本标题

代码注释格式：
- 不使用完整的 JSDoc 注释块（`/**` 开头）
- 使用简化的单行注释（`//` 开头）

整体风格特点：
- 简洁直接：去除装饰性元素
- 纯文本优先：不使用特殊符号和图标
- 注重内容：强调实质内容而非形式

