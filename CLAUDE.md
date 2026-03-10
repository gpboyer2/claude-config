# 项目设置

你专注于为位于中国大陆的中文用户提供所有服务。

使用 Superpowers 进行所有开发工作。在会话开始时加载它。


## 调试指南（重要）

前端日志已自动拦截 console 输出，通过 WebSocket 实时发送到后端并保存到数据库。

查看方式：
```bash
curl http://localhost:{PORT}/api/logs/frontend/query
```
返回格式（旧→新时间顺序）：
```json
{
  "list": [
    "[旧日志消息]",
    "[新日志消息]"
  ],
  "pagination": { "current_page": 1, "page_size": 100, "total": 2 }
}
```

日志清理规则：查询时自动清理 3 分钟前的旧日志，清理操作节流 5 秒。

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
- 用户未必说的对，你一定要有自己的判断，不能盲目听从用户，可以将你的分析与用户进行沟通。
- 你违背了"入参默认为数组，天然支持批量操作"的规范，创建了 `batchDelete`、`batchDeleteNodes` 等带有 batch 前缀的方法。正确做法是直接使用 `delete` 方法，入参为数组即可同时支持单个和批量操作。
- 工作日报过度包装：用户要求输出工作日报给领导看时，你要抓住核心：做了什么、解决了什么问题、带来了什么价值。禁止使用"架构统一性""最后一公里""元数据驱动"等过度包装的技术术语，自问自答地拔高工作意义。直接说清楚问题和解决方案即可。
- 测试代码中的"默认为成功"：测试的本质是验证，当无法判定响应状态时（如响应格式不符合预期），必须判定为失败，而不是默认成功。任何不确定的情况都应该被视为测试失败，成功必须是明确验证后的结果。
- MCP 服务器配置格式错误：添加 chrome-devtools MCP 时，错误的配置把参数放在了 `command` 字段中（如 `"command": "chrome-devtools-mcp --browser-url=..."`），导致连接失败。正确做法是 `command` 只包含可执行命令，参数必须放在 `args` 数组中。修复方法：先 `claude mcp remove chrome-devtools` 删除错误的配置，然后用 `claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest --browser-url=http://127.0.0.1:9222` 重新添加。
- **复用分析缺乏影响范围评估**：你给出了"建议抽离 useSnapshotManager"等复用建议，但没有分析：
  1. 这个函数被哪些组件引用？
  2. 修改后需要同步更新哪些调用点？
  3. 抽离后的迁移路径是什么？
  4. 新旧如何共存，逐步迁移？
  正确做法：在给出任何复用建议前，必须先完整分析影响范围，列出所有引用点，给出具体的改动顺序和验证计划。只说"建议抽离"是错误的，必须说"如果要抽离，需要修改A、B、C文件，顺序是1、2、3"。

# 项目架构与技术栈

## 数据存储规范（重要）

**开发阶段数据库索引规范（重要）**：

核心原则：开发阶段不关注数据库性能，索引只会增加数据迁移复杂度。

具体规则：
1. 禁止在 Sequelize 模型中定义任何索引（indexes 配置）
2. 禁止在代码中手动创建索引
3. 只保留主键索引（PRIMARY KEY），其余索引全部删除
4. 生产环境部署时再根据性能需求添加必要索引

原因：索引约束（如 UNIQUE）会导致数据迁移失败，开发阶段数据量小无需考虑查询性能，索引增加表结构变更的复杂度。

**项目中的数据只有三种存储状态**：

1. **内存变量**：命名空间或代码块的局部变量，声明，使用 `ref`、`reactive`、`computed`、`let` 等
2. **Pinia Store**：全局状态管理，仅用于跨组件共享的状态
3. **URL 参数**：通过 `route.query` 传递，直接调用接口获取数据

**禁止使用的存储方式**：
- 禁止使用 `sessionStorage` 存储动态数据
- 禁止使用 `localStorage` 存储动态数据
- 禁止在 Pinia Store 中缓存业务数据

## 数据获取架构原则（重要）

**核心原则：不使用缓存，直接从 URL 读取参数并调接口**

**问题根源**：过度使用缓存导致数据不同步，组件间通过 props 传递数据链路过长，热更新后缓存数据丢失导致显示异常，切换 tab 时 URL 变化但组件内部状态未更新。

**正确做法**：子组件直接从 URL 读取参数（不通过 props 传递），每次需要数据时都调用接口（不使用缓存），URL 包含完整上下文信息（页面刷新后能正常恢复），子组件监听 URL 参数变化自动重新加载数据，用户操作更新 URL 而不是更新内部状态。

**代码示例**：

```vue
<!-- 错误：通过 props 传递数据 -->
<script setup>
const props = defineProps<{
  nodeId: string
  nodeName: string
  nodeData: any
}>();

const displayName = computed(() => props.nodeData?.name || props.nodeName);
</script>

<!-- 正确：直接从 URL 读取并调接口 -->
<script setup>
import { useRoute, useRouter } from 'vue-router';
import { systemLevelDesignTreeApi } from '@/api/system-level-design-tree';

const route = useRoute();
const router = useRouter();

const nodeId = computed(() => route.query.systemNodeId as string || '');
const nodeData = ref<any>(null);

async function loadNodeData() {
  if (!nodeId.value) return;
  const result = await systemLevelDesignTreeApi.getNodeById(nodeId.value);
  if (result.status === 'success') {
    nodeData.value = result.datum;
  }
}

// 监听 URL 参数变化，自动重新加载
watch(() => nodeId.value, loadNodeData, { immediate: true });

// 用户操作时更新 URL
function handleSelectItem(id: string) {
  router.replace({
    path: route.path,
    query: { ...route.query, selectedId: id }
  });
}
</script>
```

**子组件列表/面板组件规范（重要）**：
- 直接监听 URL 中的上下文参数（如 `interfaceId`、`nodeId` 等）
- 当参数变化时，自动重新加载数据
- 用户点击 item 时，更新 URL 的选中项参数
- 不依赖父组件传递数据

**禁止的行为**：
- 禁止使用 Pinia store 缓存业务数据
- 禁止通过 props 传递可以通过 URL 获取的参数
- 禁止使用 sessionStorage 缓存动态数据
- 禁止在父组件中调接口，然后通过 props 传递给子组件
- 禁止子组件只初始化一次数据而不监听 URL 变化

**URL 设计规范**：URL 应包含组件所需的所有上下文信息（示例：`?type=logic&systemNodeId=xxx&interfaceId=yyy&protocolAlgorithmId=zzz`），页面刷新后组件应能从 URL 恢复完整状态，切换 tab 后 URL 变化应触发组件重新加载数据。

## 设计文档路径

当涉及以下领域时，请先阅读对应的设计文档：

| 领域 | 文档路径 | 描述 |
|------|----------|------|
| API 自动化测试 | `TEST_ARCHITECTURE.md` | 测试架构设计、status 字段定义规范、expect 验证规范、测试隔离策略 |
| 数据库设计 | `DATABASE_SCHEMA.md` | 数据库表结构、字段定义、索引设计 |

**重要**：在编写或修改测试相关代码前，必须先阅读 `TEST_ARCHITECTURE.md`。

# 业务规范


## 1. URL 路由规范（重要）

Vue Router 使用 hash 模式。

**通用规范：禁止使用路径参数格式，必须使用查询参数格式**
- 禁止：`/:id`, `/:{params}` 等路径参数
- 例如：`/#/settings/user/:id` 是错误的
- 正确：`/#/settings/user/detail?id=123`

**前端参数读取规范（重要）**

前端不准使用 `route.params`，只能用 `route.query`。

**禁止行为**：
- 不准使用 `route.params`
- 不准在路由配置中定义动态路径参数（如 `/:id`）
- 所有动态参数必须通过 query 参数传递


## 2. 组件引入规范

**禁止使用组件映射表（例如editorComponentsMap）进行动态组件加载**


**正确做法（必须）**：
```vue
<script setup>
import {ComponentName} from './components/{path/to/component}.vue';

</script>

<template>
  <component :is="currentComponent" />
</template>
```

**强制要求**：
- 所有动态组件加载必须使用直接引入方式
- 禁止创建任何形式的组件映射表（包括对象、Map、数组等）
- 使用 if/else 或 switch 明确表达组件选择逻辑
- 组件引用必须是变量名，不能是字符串


## 2.1 组件功能归属原则（重要）

**核心原则：功能只被一个组件使用时，放在该组件内部；被多个组件使用时，才放在父组件或公共模块。**

### 判断标准

**功能归属于子组件**：功能只被这一个子组件使用，操作结果只影响该子组件的状态，不需要跨组件通信。

**功能归属于父组件**：功能被多个子组件共享，操作结果需要影响多个子组件，需要全局协调或统一管理。

### 规范

1. **弹窗归属**：如果弹窗只被一个组件使用，放在该组件内部，使用 `v-if` 控制显示
2. **状态管理**：组件维护自己的数据，监听 URL 参数变化自动刷新
3. **避免过度通信**：不需要为了"通知父组件"而使用 emit

### 决策流程

```
这个弹窗/功能有几个使用者？
  │
  ├─ 只有一个组件使用
  │   └─ 放在该组件内部，用 v-if 控制显示
  │
  └─ 多个组件使用
      └─ 放在父组件或公共模块
```

### 实施效果
- **减少通信**：组件自主管理，无需通过 emit/props 传递；**职责清晰**：使用该功能的组件就是该功能的拥有者；**易于维护**：所有相关逻辑集中在一处，修改时不用跨文件。

## 3. 命名规范

| 类别 | 规范 | 示例 |
|------|------|------|
| 数据库表名 | snake_case | `grid_strategies`, `grid_trade_history` |
| 数据库字段名 | snake_case | `trading_pair`, `grid_price_difference` |
| 文件名 | kebab-case | `communication-node.js`, `packet-message.js` |
| JavaScript 类名 | 保持不变 | `class Robot`（类名都是 PascalCase） |
| Model 字段定义 | snake_case | `api_key: DataTypes.STRING` |
| 函数/方法名 | camelCase | `getOrderList()`, `createStrategy()`, `handleClick()` |
| 变量名（含局部变量） | snake_case | `const grid_strategy = ...` |
| 集合变量名 | snake_case + list 后缀 | `const account_list = ...` |
| 前端类型定义字段 | snake_case | `interface Strategy { trading_pair: string }` |
| 前端类型定义名称（interface/type） | camelCase | `interface CacheLoadOptions<T> { ... }` |
| API 请求/响应字段 | snake_case | `{ trading_pair: "BTCUSDT" }` |


## 3.1 前端目录组织规范（重要）

**核心原则：按业务划分，独立业务平级，共享组件放 components/**

### 目录结构模板

```
页面目录/                    # 如 settings、packet-config、ide
  components/                # 页面级共享组件（被该页面下多个业务共享）
    shared-component-a/     # 只在被多个业务共用时才放这里
  business-a/                # 独立业务 A（与 components 平级）
    index.vue
    index.scss
  business-b/                # 独立业务 B（与 components 平级）
    index.vue
    index.scss
  business-c/                # 独立业务 C（与 components 平级）
    components/sub-c/        # business-c 专属的子组件
      index.vue
    index.vue
    index.scss
```

### 判断标准

| 组件类型 | 放置位置 | 判断依据 |
|----------|----------|----------|
| 独立业务页面 | 与 `components/` 平级 | 是一个独立的业务入口，有独立的路由 |
| 页面级共享组件 | `页面/components/` 下 | 被该页面下多个业务共用 |
| 某业务的专属子组件 | 该业务目录下的 `components/` 中 | 只被该业务内部使用 |

### 实施目的

- **业务清晰**：每个业务是独立的，一目了然

### 禁止行为
- 禁止组件目录名重复父目录的业务前缀
  - 错误：`settings/components/sub-comp-settings/`（已包含 settings 目录，无需重复后缀）
  - 正确：`settings/components/sub-comp/`

**类型定义命名说明**：TypeScript 的 `interface` 和 `type` 名称使用 camelCase，变量名使用 snake_case，类型定义内部字段名仍使用 snake_case。

**第三方交互例外**：与币安等第三方交互时，保持第三方原有的命名风格。

## 3. API 响应处理规范（重要）

统一响应格式：HTTP 状态码统一使用 200，通过响应体中的 status 字段区分成功/失败，前端通过 response.status 判断（不再使用 try-catch 处理业务错误），业务数据字段使用 datum（避免与 Axios 的 data 字段混淆）。

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
```

入参规范：
- GET 请求参数使用 `req.query`，例如：`req.query.id`
- POST 请求参数使用 `req.body`，例如：`req.body.id`
- 禁止使用 `/api/xxx/{id}` 路径参数（`req.params`）
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

### 3.5 增删改查接口命名规范（重要）

**强制规则：四大操作必须使用以下关键词**

| 操作 | 关键词 | URL 示例 | 说明 |
|------|--------|----------|------|
| 增 | create | POST /api/xxx/create | 创建资源 |
| 删 | delete | POST /api/xxx/delete | 删除资源 |
| 改 | update | POST /api/xxx/update | 更新资源 |
| 查 | query | GET /api/xxx/query | 查询资源 |

**禁止使用的关键词（错误示例）**：

| 操作 | 禁止关键词 | 错误示例 | 正确示例 |
|------|------------|----------|----------|
| 增 | add, insert, new, save | /add, /insert, /new | /create |
| 删 | remove, destroy | /remove, /destroy | /delete |
| 改 | edit, modify, change | /edit, /modify | /update |
| 查 | list, get, fetch, find, search | /list, /get/:id, /fetch | /query |

**URL 格式规范**：
- 增：`POST /api/{resource}/create`
- 删：`POST /api/{resource}/delete`
- 改：`POST /api/{resource}/update`
- 查：`GET /api/{resource}/query`

## 3.6 接口精简原则
- 基础接口只需要增删改查四个
- 除非必要不新增其他接口
- 批量操作通过数组入参自然支持

批量操作命名规范（重要）：
- 入参默认为数组，天然支持批量操作
- 禁止使用 batch 前缀的方法名，如 batchDelete、batchCreate
- 删除接口直接命名为 delete，入参是数组即可同时支持单个和批量删除
- 正确示例：`delete(idList)` - 传入 `[id]` 删除单个，传入 `[id1, id2, ...]` 批量删除
- 错误示例：`batchDelete(idList)` - 不要用 batch 前缀
- 删除接口不要使用路径参数 `/:id`，统一用 `POST /api/xxx/delete` + `{ data: [id1, id2, ...] }`

## 3.7 多团队协作中的接口设计规范（重要）

**背景**：本项目是多团队多人协作，不同团队成员水平参差不齐。后端接口设计往往不完整，导致前端代码变得复杂。


**强制规范**：
1. **每个资源必须有完整的增删改查接口**
   - `POST /api/{resource}/create` - 创建单个资源
   - `POST /api/{resource}/delete` - 删除资源（支持批量）
   - `POST /api/{resource}/update` - 更新资源
   - `GET /api/{resource}/query` - 查询资源

2. **禁止用"整体替换"实现增删操作**
   - 错误：前端获取列表 → 本地添加/删除 → 调用整体替换接口
   - 正确：直接调用 create/delete 接口

3. **子资源也需要独立接口**
   - 例如：报文引用（packet_ref）是节点的子资源
   - 必须提供 `POST /api/communication-nodes/packet-ref/create`
   - 必须提供 `POST /api/communication-nodes/packet-ref/delete`
   - 不能只依赖 `POST /api/communication-nodes/update-endpoints` 整体替换

**遇到接口不完整时的处理流程**：
1. 先检查后端是否已有对应接口
2. 如果没有，推动后端添加缺失的接口

**正确示例**：
```typescript
// 添加报文关联
await communicationNodeApi.create(nodeId, interfaceId, packetId, direction);

// 删除报文关联
await communicationNodeApi.delete(nodeId, interfaceId, packetId);
```

**错误示例（不能将错就错继续使用不符合规范和要求的代码，函数和接口）**：
```typescript
const list = await getList();
list.push(newItem);
await updateEndpoints(nodeId, list);
```


## 4. 前后端数据交互规范

**字段命名统一原则（重要）**：
- 核心规则：后端用什么字段，前端必须完全使用后端的字段名
- 前端不得对字段名进行任何转换，直接使用后端原始字段名
- 前端类型定义必须与后端返回的字段名完全一致

**后端不同模块的命名风格**：
- 网格策略相关 API：使用 snake_case（如 `trading_pair`, `position_side`, `grid_price_difference`, `api_key`, `api_secret`）
- 与第三方交互（如币安）：迁就第三方的命名风格

**前端开发规范**：
- 前端类型定义（TypeScript interface）的字段名必须与后端返回的字段名完全一致
- 禁止在前端进行字段名转换（如 `tradingPair` <-> `trading_pair`）
- 调用后端 API 时直接透传整个对象，不要手动列举每个字段

**接口调用规范**：

前端：
- 直接透传整个对象，不要手动列举每个字段
- 错误做法：`await api.update(id, { field1: data.field1, field2: data.field2 })`
- 正确做法1：`await api.update(id, data)`
- 正确做法2：`await api.update(id, Object.assign({}, data, {name: 'plq'}))`

后端Controller：
- 前后端字段名已统一（都是snake_case），直接透传req.body，避免冗余解构
- 只处理真正需要转换的（如ID从字符串转数值）
- 避免遗漏风险：解构后新增字段可能忘记传递

错误示例：
```javascript
// 一堆解构，冗余且易遗漏
const { trading_pair, position_side, grid_price_difference } = req.body;
await create({ trading_pair, position_side, grid_price_difference });
```

正确示例：
```javascript
// 直接透传
await create(req.body);

// 需要转换时用Object.assign
const id = parseInt(req.body.id);
await create(Object.assign({}, req.body, { id }));
```

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
- 禁止使用硬编码颜色值（如 `#ffffff`、`rgb(255, 255, 255)`、`rgba(0, 0, 0, 0.1)`）
- 所有颜色必须使用 CSS 变量引用（如 `var(--color-bg-container)`）
- 全局 CSS 变量统一定义在 `client/src/styles/index.scss` 中的 `:root` 选择器内
- 禁止存在"模块专用变量"概念，全局变量就是通用的，所有人都要遵守
- 如需新的颜色变量，在 `index.scss` 中添加语义化的变量名

SCSS 模块导入规范：
- 必须使用 `@use` 替代 `@import` 导入 SCSS 模块
- CSS 文件（如 .css）必须使用 `@import`，因为不支持 `@use`
- `@use` 语句必须写在文件最开头，在任何其他规则之前（注释除外）


## 2. JS/React/Vue 代码规范

极简原则：
- 禁止新增冗余变量和代码，代码行数缩减到最少
- 资源复用：必须充分利用已有资源和代码
- 向前看：不用考虑向后兼容性
- 同步性：保持样式风格不变，并同步更新或校验其他模块

约定优于配置原则（重要）：

**核心原则：禁止虚假兼容，约定明确格式，不符合就报错**

虚假兼容的问题：
- 代码充满各种 if else 兼容判断
- 新人不知道该用哪个参数
- 旧格式永远无法清理
- bug 隐藏在兼容逻辑中

正确做法：
```javascript
// 正确：只取约定好的字段，一个映射清晰明了
const PARAM_MAP = {
  logic: 'systemNodeId',
  protocol: 'protocolAlgorithmId',
};
const param_name = PARAM_MAP[type];
return query[param_name];

// 错误：到处兼容
if (route.params.id) return route.params.id;
if (route.query.nodeId) return route.query.nodeId;
if (route.query.systemNodeId) return route.query.systemNodeId;
return route.query.id;
```

禁止行为：
- 禁止为了"兼容"写 3 个以上的 if 分支
- 禁止在注释中写"兼容旧格式"而不说明什么时候删除
- 禁止用 `||` 链式兼容超过 2 个字段

后端 Lint 与 TypeScript 检查原则：
- 后端是纯 JavaScript 项目，TypeScript 检查只是辅助工具
- lint 命令包含 typecheck，但只修复影响业务的实际代码问题
- 不需要为纯类型错误添加类型断言、其他修复
- 第三方库的类型定义问题（如 axios、socket.io、binance 等）不需要修复
- 不增加额外负担：除非是反映实际业务逻辑的问题，否则忽略 TypeScript 错误

#### 函数职责单一原则（接口调用拆分规范）

**核心原则：一个函数只调用一个接口，多个接口必须拆分**

当一个函数需要调用多个不同的 API 时，必须拆分成多个独立的函数，每个函数只负责调用一个接口。

**错误示例**：
```javascript
// ❌ 错误：一个函数调用3个接口，职责不清
async function loadNodeData() {
  const result1 = await api1.get();  // 基本信息
  const result2 = await api2.get();  // 接口数量
  const result3 = await api3.get();  // 逻辑流数量
}
```

**正确示例**：
```javascript
// ✅ 正确：拆分成3个函数，各司其职
async function loadNodeBasicInfo() { }
async function loadInterfaceCount() { }
async function loadLogicFlowCount() { }
```

**拆分的目的**：
1. 代码清爽度：一眼看懂每个函数的用途
2. 优雅度：职责单一，易于理解
3. 可维护性：修改一个接口调用不影响其他
4. 可测试性：可以独立测试每个接口调用

**判断标准**：
- 一个函数调用了2个或以上不同的 API → 必须拆分
- 函数命名应该清晰表达它的目的（如 loadNodeBasicInfo、loadInterfaceCount）

#### 前端错误处理规范（重要）

**核心原则：后端已统一处理消息提示，前端 catch 块保持极简**

后端通过响应的 `message` 字段统一返回错误信息，前端不需要复杂的错误类型判断。

**正确做法**：
```typescript
// 简单处理：记录日志 + 显示错误消息
catch (err: any) {
  console.error('获取账户信息失败:', err);
  showMessage('获取账户信息失败', 'error');
}
```

**禁止的行为**：
- 禁止在 catch 块中进行复杂的错误类型判断（如 instanceof、多层 if-else）
- 禁止根据错误类型做不同处理分支
- 禁止在 catch 块中调用多个接口或执行复杂逻辑
- 禁止重新抛出错误（throw err）

**原因**：后端已通过 apiSuccess/apiError 统一处理错误响应，message 字段包含完整错误信息。前端只需记录日志并显示提示，过度处理只会增加代码复杂度。

### Vue 组件拆分规范

组件行数阈值：
- 单个 Vue 文件不超过 1000 行（含模板、脚本、样式）
- 超过 1000 行必须拆分

拆分原则（单一职责、高内聚低耦合）：
- 每个子组件只负责一个明确的功能或业务领域
- 相关的数据和操作逻辑放在同一个组件内
- 可复用的逻辑抽离到 composables

拆分方案要求：
- 拆分后的组件层级结构（父组件 + 子组件命名/职责）
- 样式隔离方案（使用外部 SCSS 文件，不使用 scoped）
- 拆分后需注意的生命周期、数据流转风险点



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
两区之间用 4 行空行分隔。

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


## 5. Git 提交与推送规范（重要）

### Commit 风格

使用 Conventional Commits 格式

### 提交与推送流程（强约束）

**背景**：团队多人协作，推送时经常遇到冲突，需要一个稳定的流程保证代码同步。

**强制流程**：

```bash
# 1. 本地提交（暂不推送）
git commit

# 2. 拉取远程并 rebase
git pull --rebase

# 3. 如有冲突，解决冲突
#    - 手动编辑冲突文件
#    - git add .
#    - git rebase --continue   # 继续 rebase（不是重新拉取）

# 4. 如果担心解决冲突期间有人推送，可再确认一次
git pull --rebase

# 5. 确认无误后推送
git push
```

**关键命令说明**：

| 命令 | 作用 |
|------|------|
| `git pull --rebase` | 拉取远程 + 将本地提交变基到远程之上 |
| `git rebase --continue` | 冲突解决后，继续完成 rebase 操作 |

**为什么用 rebase 而不是 merge**：
- 提交历史线性、干净（没有 merge 产生的分叉）
- 冲突在本地解决完再推送，远程历史整洁

**禁止的行为**：
- 禁止 commit 后直接 push（大概率被拒绝）
- 禁止用 `git pull`（默认 merge）代替 `git pull --rebase`
- 禁止冲突解决后跳过 `git rebase --continue`


## 6. HTML data-debug 调试属性规范

用于调试目的，在组件上添加 `data-debug-*` 属性来追踪状态或入口来源。浏览器控制台可用 `document.querySelector().dataset` 获取。

## 7. 代码与文档的格式风格

markdown文档格式：
- 不使用 emoji 图标
- 不使用 Markdown 加粗语法
- 使用简洁的纯文本标题

代码注释格式：
- 使用完整的 JSDoc 注释块（`/**` 开头）



## 8. 代码修改影响分析原则

**核心原则：改一处而动全身，修改前必须全面分析影响范围**

批量替换导致语义错误的教训：
- 场景：简化 `compOptions?.options?.field` 为 `compOptions?.field`
- 错误1：只批量替换了 `compOptions?.options` 为 `compOptions`，遗漏了父组件传递时的 `options: {}` 包装结构
- 错误2：将 `props.compOptions?.packetData` 错误替换为 `props.compOptions`，改变了语义（从访问字段变为访问整个对象）

**正确流程**：
1. 先找到所有引用点（类型定义、赋值、使用）
2. 确定修改顺序：类型定义 -> 父组件传递 -> 子组件使用
3. 每次修改后检查是否有遗漏
4. 属性访问（`?.field`）和属性赋值（`field: value`）是两种完全不同的操作，不能混为一谈
5. 批量替换后必须运行项目验证功能是否正常

**禁止的行为**：
- 只看表面相似就批量替换，不理解语义就动手
