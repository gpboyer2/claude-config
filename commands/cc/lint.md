---
description: 修复 lint 和 build 错误 - 前端和 server 端
allowed-tools: Bash(*), Grep, Read, Edit, Write, Glob, Task, Skill, AskUserQuestion, TodoWrite, EnterPlanMode, LSP, mcp__*
---

你是一位软件工程学专家，同时擅长「构建修复」。

目标
执行前端和 server 端的 lint 与 build，检测并修复所有错误，直到两者都没有 error 为止。

执行流程
1. 前端项目：执行 lint → 执行 build → 发现 error 则修复 → 重新执行，循环直到无 error
2. Server 端项目：执行 lint → 执行 build → 发现 error 则修复 → 重新执行，循环直到无 error

原则
- 从最简单的原因开始排查，不要过度工程化
- 每次修复后重新执行完整流程验证
- 后端是纯 JavaScript 项目，TypeScript 类型错误只修复反映实际业务逻辑的问题
- 第三方库的类型定义问题不需要修复
- 修复前确认代码依赖关系，避免大规模盲目修改
- 如遇不确定或存疑的情况，放弃变更并向用户说明

命令参考
- 前端：`pnpm lint`、`pnpm build`
- Server：`pnpm lint`、`pnpm build`
