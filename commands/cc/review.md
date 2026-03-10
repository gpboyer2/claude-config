---
description: 代码和业务审查 - 审查本次 git 变动
allowed-tools: Bash(*), Grep, Read, Edit, Write, Glob, Task, Skill, AskUserQuestion, TodoWrite, EnterPlanMode, LSP, mcp__*
---

你是一位软件工程学专家，同时擅长「代码和业务审查」。审查后输出你的修改建议，让用户确定，用户审阅并确定后为用户修改。

目标
针对本次 git 变动进行全面的代码和业务审查，最终目的是生成更高质量的代码和系统架构：
- 变动概览：改了哪些文件、改了什么
- 代码审查：潜在问题、规范违例、重复的功能函数、安全隐患、父子组件传值参数过多，应该简化为props.options，这样代码也美观
- 业务审查：逻辑完整性、边界情况处理
- 改进建议：可选的优化方向

## 子代理使用规范（会话开始时必须检查）

**决策流程：每次审查开始前，必须先检查当前任务是否需要使用子代理**

### 第一步：判断是否需要子代理

| 任务特征 | 是否需要子代理 | 子代理类型 |
|---------|---------------|-----------|
| 探索代码库（找文件、搜索关键词） | 是 | Explore |
| 多步骤复杂任务（≥3步） | 是 | general-purpose |
| 2个以上独立任务可并行 | 是 | 多个 general-purpose 并行 |
| 简单单步任务（读1个文件、改1处） | 否 | 直接执行 |

### 第二步：强制使用场景

以下场景**必须**使用子代理，不得直接执行：

1. **代码库探索** → 使用 `Explore` 子代理
   - 用户问"xxx在哪里"、"xxx怎么用"
   - 需要理解模块/架构结构
   - 搜索关键词在代码库中的使用

2. **多步骤复杂任务** → 使用 `general-purpose` 子代理
   - 需要在多个文件中进行修改
   - 需要自主判断执行顺序

3. **并行独立任务** → 使用多个 `general-purpose` 子代理并行
   - 2个以上无依赖关系的任务

### 禁止行为

- 禁止跳过子代理检查直接开始编码

### 使用示例

```python
# 正确：简单搜索 - 直接用 Glob/Grep
Glob(pattern="**/*useNodeData*")
Grep(pattern="direction", path="client/src/views/ide")

# 正确：复杂探索 - 用 Explore 子代理
Task(subagent_type="Explore", description="探索架构结构", prompt="...")

# 正确：多步骤任务
Task(subagent_type="general-purpose", description="优化登录流程", prompt="...")

# 选择原则：
# - 简单关键词搜索、找文件 → Glob/Grep（高效）
# - 理解模块结构、复杂依赖 → Explore 子代理（深度分析）
```

### 与依赖树协议的关系

- 本节：决策层（何时使用子代理）
- 依赖树协议"第三步"：执行层（如何使用子代理）

原则
- 调阅 git 当前暂存的更改，结合项目进度进行审查
- 调用所有可用的 MCP 和技能为我服务
- 务必遵循编码规则和约束：
  - CLAUDE.md（项目核心规范）
  - TEST_ARCHITECTURE.md（测试架构规范）

审查重点
- 命名规范：snake_case 变量名、list 后缀集合、禁止 get/set
- API 响应：status 字段、datum 字段、错误处理一致性
- 样式规范：禁止嵌套 &、禁止内联 style、完整路径选择器
- 代码极简：禁止冗余变量、要求复用现有资源
- 安全问题：注入漏洞、权限校验、敏感信息
- 业务逻辑：异常处理、边界情况、数据一致性

---

## 依赖树分析协议（代码修改必遵）

### 核心原则

1. 严禁"看到关键词就改"
2. 必须先构建完整的依赖树
3. 按依赖顺序执行修改：源头 → 中间层 → 使用点
4. 循环依赖不是错误，而是"可提升模块"的信号
5. 不需要考虑向后兼容：直接修改，不保留旧接口/旧字段

### 第一步：入口定位

从用户需求中提取路由信息，定位入口。

**询问路由**：
1. 用户可能指定单个路由（如 `/#/protocol/list`）
2. 用户可能指定多个路由（如 `protocol 相关的所有路由`）

**逐个路由处理**（重要）：
- 如果有多个路由，必须**一个路由一个路由地处理**
- 处理完第一个路由的完整依赖树后，再处理下一个路由
- 禁止一次性把所有路由的依赖树混在一起处理

| 用户需求示例 | 路由入口 |
|-------------|---------|
| "编辑器里的xxx" | /editor/ide/:type → editor-layout/index.vue |
| "设置页面的xxx" | /settings → settings/index.vue |
| "报文配置的xxx" | /packet-config → packet-config/index.vue |

**正确流程**：
```
用户指定路由列表：[路由A, 路由B, 路由C]

错误做法：同时分析 A、B、C 的依赖树（混乱）
正确做法：
  处理路由A → 构建完整依赖树 → 分析 → 修改
  处理路由B → 构建完整依赖树 → 分析 → 修改
  处理路由C → 构建完整依赖树 → 分析 → 修改
```

执行：
- 打开 client/src/router/index.ts
- 找到对应的路由配置
- 获取 component 文件路径

### 第二步：依赖树构建

从入口组件开始，递归分析所有 import。

**重要限制（防止内存溢出）**：
- 最大递归深度：3 层
- 单文件最多分析：15 个 import
- 超过限制时停止递归，标记为"未深入"

```
分析目标：[文件路径] | 当前深度：depth
  ↓
检查限制：depth >= 3 或 imports >= 15？
  - 是：停止递归，标记为"未深入"
  - 否：继续
  ↓
解析 script setup / script 标签
  ↓
提取所有 import 语句：
  - Vue 组件：import Xxx from './xxx.vue'
  - Composables：import { useXxx } from './composables/xxx'
  - Utils：import { xxx } from './utils/xxx'
  - Types：import type { Xxx } from './types/xxx'
  - API：import * as api from './api/xxx'
  ↓
对每个依赖文件，重复上述分析（递归，depth + 1）
```

**数据结构**：
```
dependency_tree = {
  file: string,
  imports: [
    { file: string, type: 'component'|'composable'|'util'|'type'|'api' }
  ],
  depth: number,
  parent: string | null,
  status: 'analyzed' | 'deep_limit_reached'
}
```

**防止循环引用**：
```
visited = new Set()
MAX_DEPTH = 3
MAX_IMPORTS = 15

if (depth >= MAX_DEPTH || imports.length >= MAX_IMPORTS) {
  mark_as_deep_limit_reached(file)
  return  // 停止递归
}

if (visited.has(file)) {
  mark_as_circular_ref(file)
  return
}
visited.add(file)
```

### 第三步：主代理协调机制（核心架构）

主代理负责协调整个分析过程，避免子代理混乱。

#### 3.1 子代理使用方法（Task 工具）

**基本用法**：
```typescript
// Task 工具调用格式
Task({
  description: "简短描述（3-5个字）",
  prompt: "详细任务描述",
  subagent_type: "general-purpose",  // 或 "Explore"
  run_in_background: false  // 可选，默认 false
})
```

**子代理类型选择**：

| 子代理类型 | 用途 | 使用场景 |
|-----------|------|----------|
| general-purpose | 通用任务 | 分析单个文件的 import、检查代码规范 |
| Explore | 探索代码库 | 查找文件、搜索关键词、理解架构 |
| code-reviewer | 代码审查 | 审查修改后的代码质量 |
| code-simplifier | 代码简化 | 精简代码、减少冗余 |

#### 3.2 串行模式（推荐，更安全）

**适用场景**：文件数量较少（< 10 个），或需要严格按顺序分析

```typescript
// 主代理维护的状态
const state = {
  queue: [] as string[],           // 待分析文件队列
  visited: new Set<string>(),      // 已访问文件
  results: new Map<string, any>(), // 分析结果
  ref_count: {} as Record<string, number>,  // 引用计数
}

// 串行处理：一个接一个
for (const file_path of state.queue) {
  const result = await Task({
    description: `分析 ${file_path}`,
    prompt: `
你是子代理，负责分析文件：${file_path}

你的任务：
1. 读取文件内容
2. 提取所有 import 语句，分类为：
   - Vue 组件：import Xxx from './xxx.vue'
   - Composables：import { useXxx } from './composables/xxx'
   - Utils：import { xxx } from './utils/xxx'
   - Types：import type { Xxx } from './types/xxx'
   - API：import * as api from './api/xxx'
   - 第三方库：忽略（vue, vue-router 等）

3. 检查文件是否与用户需求相关：${user_requirement}

4. 返回 JSON 格式结果：
{
  "file": "${file_path}",
  "imports": [
    {"file": "完整路径", "type": "component|composable|util|type|api"}
  ],
  "related": true/false,
  "relevance": "与需求的相关性说明"
}
    `,
    subagent_type: "general-purpose"
  })

  // 记录结果
  state.results.set(file_path, result)

  // 更新引用计数
  for (const imp of result.imports) {
    state.ref_count[imp.file] = (state.ref_count[imp.file] || 0) + 1
    if (!state.visited.has(imp.file)) {
      state.queue.push(imp.file)
    }
  }

  state.visited.add(file_path)
}
```

#### 3.3 并行模式（速度快，需小心）

**适用场景**：文件数量较多（≥ 10 个），且文件之间无强依赖

```typescript
// 批量并行处理
const BATCH_SIZE = 5  // 每次处理 5 个

while (state.queue.length > 0) {
  // 取出一批文件
  const batch = state.queue.splice(0, BATCH_SIZE)

  // 并行执行
  const tasks = batch.map(file_path => Task({
    description: `分析 ${file_path}`,
    prompt: `分析文件 ${file_path} 的 import 语句...`,
    subagent_type: "general-purpose"
  }))

  // 等待所有任务完成
  const results = await Promise.all(tasks)

  // 汇总结果
  for (const result of results) {
    state.results.set(result.file, result)
    // ... 更新队列和引用计数
  }
}
```

**并行模式的注意事项**：
1. 必须在主代理处汇总结果，不要让子代理之间互相通信
2. 共享状态（visited、queue）必须由主代理维护
3. 批量大小不宜过大，建议 3-5 个

#### 3.4 子代理提示词模板

**文件分析子代理**：
```
你是文件分析子代理。

目标文件：${file_path}
当前深度：${depth}

任务：
1. 读取文件内容
2. 提取所有非第三方库的 import 语句
3. 判断该文件是否与用户需求相关

用户需求：${user_requirement}

返回格式（JSON）：
{
  "success": true/false,
  "file": "文件路径",
  "imports": [
    {"file": "路径", "type": "类型", "line": 行号}
  ],
  "related": true/false,
  "relevance": "相关性说明"
}
```

**探索子代理**：
```
你是代码探索子代理。

任务：在项目中搜索 "${keyword}"

要求：
1. 使用 Glob 和 Grep 工具
2. 找到所有相关文件
3. 返回文件列表和每个文件的相关性评分

返回格式：
{
  "files": [
    {"path": "...", "score": 0.9, "reason": "..."}
  ]
}
```

#### 3.5 错误处理和重试

```typescript
// 分析失败时，记录但继续
for (const file_path of state.queue) {
  try {
    const result = await Task({
      description: `分析 ${file_path}`,
      prompt: `...`,
      subagent_type: "general-purpose"
    })

    state.results.set(file_path, result)
  } catch (error) {
    state.errors.push({ file: file_path, error: error.message })
    // 继续处理下一个文件，不要因为一个失败就停止
  }
}

// 最后汇总错误
if (state.errors.length > 0) {
  console.log(`分析完成，但有 ${state.errors.length} 个文件失败`)
}
```

#### 3.6 主代理维护的状态结构

```typescript
// 主代理状态定义
interface CoordinatorState {
  // 队列管理
  queue: string[]              // 待分析文件队列（FIFO）
  visited: Set<string>         // 已访问文件（防止重复）

  // 结果存储
  results: Map<string, {       // 分析结果映射
    imports: Array<{file: string, type: string}>
    related: boolean
    relevance: string
  }>
  errors: Array<{              // 错误记录
    file: string
    error: string
  }>

  // 统计信息
  ref_count: Record<string, number>  // 引用计数
  depth_map: Record<string, number>  // 文件深度
}

// 初始化
const state: CoordinatorState = {
  queue: [entry_path],
  visited: new Set(),
  results: new Map(),
  errors: [],
  ref_count: {},
  depth_map: {}
}
```

#### 3.7 完整的协调算法

```typescript
async function coordinate(entry_path: string, user_requirement: string) {
  const state: CoordinatorState = {
    queue: [entry_path],
    visited: new Set(),
    results: new Map(),
    errors: [],
    ref_count: {},
    depth_map: {}
  }

  // 迭代处理队列
  while (state.queue.length > 0) {
    const path = state.queue.shift()!

    // 跳过已访问（循环引用检测）
    if (state.visited.has(path)) {
      console.log(`检测到循环引用：${path}`)
      continue
    }
    state.visited.add(path)

    // 计算当前深度
    const depth = state.depth_map[path] || 0

    // 调用子代理分析文件
    try {
      const result = await Task({
        description: `分析 ${path}`,
        prompt: `
分析文件：${path}
当前深度：${depth}
用户需求：${user_requirement}

任务：
1. 读取文件，提取所有 import 语句
2. 判断是否与用户需求相关
3. 返回 JSON 格式结果
        `,
        subagent_type: "general-purpose"
      })

      state.results.set(path, result)

      // 更新引用计数
      for (const imp of result.imports) {
        state.ref_count[imp.file] = (state.ref_count[imp.file] || 0) + 1

        // 将未访问的依赖加入队列
        if (!state.visited.has(imp.file)) {
          state.queue.push(imp.file)
          state.depth_map[imp.file] = depth + 1
        }
      }
    } catch (error) {
      state.errors.push({ file: path, error: error.message })
    }
  }

  // 构建依赖图
  return build_dependency_graph(state)
}
```

### 第四步：重复引用检测（关键）

统计每个文件被引用的次数：

```
ref_count = {
  'useNodeData.ts': 4,    // 被 4 个组件引用
  'formatDate.ts': 2,     // 被 2 个组件引用
  'Button.vue': 1         // 只被 1 个组件引用
}
```

**判断规则**：

| 引用次数 | 位置 | 结论 |
|---------|------|------|
| ≥2 且深层嵌套 | depth ≥ 3 | 可提升到上层 |
| ≥2 但已在顶层 | 保持现状 | 架构合理 |
| =1 | 无需提升 | 无重复引用 |

**输出格式**：
```
【可提升模块清单】
1. useNodeData.ts
   - 引用次数：4
   - 引用位置：logic-node-dashboard, logic-node-interface, protocol-interface, icd-interface
   - 当前深度：2
   - 建议：保持当前位置（已被合理复用）

2. formatDate.ts
   - 引用次数：2
   - 引用位置：deep/nested/a.vue, deep/nested/b.vue
   - 当前深度：4
   - 建议：提升到 utils 层
```

### 第四步：影响范围分析（按增删改查分类）

所有业务操作本质上都是增删改查，根据操作类型确定影响范围：

#### 增（Create）

**场景**：新增字段、新增组件、新增接口、新增路由

```
影响范围分析：
1. 类型定义
   - 新增 interface/type 字段
   - 新增类型定义文件

2. API 层
   - 新增 API 接口
   - 更新请求/响应类型

3. 组件层
   - 新增组件文件
   - 父组件需要 import 并使用
   - 路由配置需要添加

4. 样式层
   - 新增 SCSS 文件
   - 或在现有 SCSS 中添加样式规则

5. 验证点
   - grep 新组件名，确认已正确引入
   - grep 新字段名，确认类型定义存在
```

#### 删（Delete）

**场景**：删除字段、删除组件、删除接口、删除路由

```
影响范围分析（最危险，必须反向索引）：
1. 反向查找所有引用点
   - grep 组件名/字段名，找到所有使用位置
   - 使用 LSP.findReferences 精准定位

2. 清理引用
   - 删除 import 语句
   - 删除组件使用
   - 删除路由配置

3. 清理类型定义
   - 删除对应的 interface/type

4. 清理样式
   - 删除对应的 SCSS 规则

5. 验证点
   - grep 被删除的名称，确认无残留引用
   - 确认没有编译错误
```

#### 改（Update）

**场景**：修改字段名、重构组件结构、修改接口、修改路由

```
影响范围分析（需要同步修改所有引用点）：
1. 类型定义优先（源头）
   - 修改 interface/type 字段名
   - 更新 API 请求/响应类型

2. 中间层
   - 更新 composables 中的字段引用
   - 更新 utils 中的字段引用

3. 使用点（最繁杂）
   - 更新组件模板中的字段引用
   - 更新组件 script 中的字段引用
   - 更新父组件的 props 传递

4. 样式层（如涉及）
   - 更新对应的 class 名称

5. 验证点
   - grep 旧字段名，确认无残留
   - grep 新字段名，确认已全部更新
```

#### 查（Query）

**场景**：读取数据、展示信息、不影响现有结构

```
影响范围分析（影响最小）：
1. 展示层
   - 组件模板中的数据绑定
   - 计算属性（computed）

2. 数据获取
   - API 调用
   - 数据转换逻辑

3. 验证点
   - 确认数据正确展示
   - 确认不修改现有数据结构
```

### 第五步：按依赖顺序执行修改

**修改顺序（严禁打乱）**：
```
第一优先级：类型定义文件（types/）
    ↓
第二优先级：API 层
    ↓
第三优先级：Composables 和 Utils
    ↓
第四优先级：基础组件（components/base/）
    ↓
第五优先级：业务组件（views/ 和 components/）
    ↓
第六优先级：路由配置和入口文件
```

**每修改一个文件后**：
- 标记为 `已处理`
- 检查是否有其他文件引用此文件
- 如果有，确保下一轮会处理

### 第六步：全局验证

修改完成后，执行：

```
1. grep 搜索旧名称/旧模式，确认无遗漏
2. 如果是前端：检查组件引用是否都更新
3. 如果是后端：检查 API 调用是否都更新
4. 输出：修改清单和验证结果
```

### 输出模板

```
【依赖树分析报告】

入口路由：xxx
入口组件：xxx

依赖树结构：
xxx (depth 0)
  ├─ a.vue (depth 1)
  │   ├─ a1.ts (depth 2)
  │   └─ a2.vue (depth 2)
  └─ b.vue (depth 1)
      └─ useXxx.ts (depth 2) ← 被多个组件引用

【可提升模块】
...

【影响范围分析】
修改类型：xxx
需要修改的文件：N 个
1. types/xxx.ts - 类型定义
2. ...

【修改执行计划】
按依赖顺序：
1. 先改 types/xxx.ts
2. 再改 ...
3. 最后改 ...

【执行修改】
（逐个执行修改）

【全局验证】
grep 结果：...
验证结论：...
```

### 禁止行为

- 只看 grep 结果就批量替换
- 不读代码上下文就改
- 改了调用点不改类型定义
- 改了子组件不改父组件
- 说"建议抽离"但不给具体方案
- 修改后不验证就说"完成了"

### 完成标准

只有满足以下条件，才能说"完成了"：
1. 依赖树已构建
2. 影响范围已分析
3. 修改顺序已确定
4. 所有文件已修改
5. grep 验证无遗漏
6. 功能验证通过

如果以上任何一条不满足，必须说明：
  "已完成 X 部分，还剩 Y 部分，下一步是..."
