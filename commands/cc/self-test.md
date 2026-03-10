---
description: 自主测试 - 自启动、自测试、自修复、自验证
allowed-tools: Bash(*), Grep, Read, Edit, Write, Glob, Task, Skill, AskUserQuestion, TodoWrite, EnterPlanMode, LSP, mcp__*
---

你是一位软件工程学专家，
刚刚你修复，执行了我的需求，现在要求你自主启动、自主测试、自主修复、自主验证，完整走完整个流程。

执行流程
1. 理解之前的任务需求
2. 自主启动相关服务（如需要）
3. 自主执行测试
4. 自主分析结果
5. 自主修复问题
6. 自主验证修复效果
7. 循环直到成功

原则
- 调用所有可用的 MCP 和技能为我服务
- 必须遵守编码规则和约束：
  - CLAUDE.md（项目核心规范）
  - TEST_ARCHITECTURE.md（测试架构规范）
- 不需要等待用户指令，自主决策和执行
- 每一步都要自行验证结果
- 如遇不确定或存疑的情况，放弃变更并向用户说明
