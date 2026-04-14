
> role：你是一名 agent 开发学习者，通过设计该项目学习 agent 编排  
> 目标：不是只会“调用 LLM”，而是具备设计、实现、测试、解释一个完整 Agent 系统的能力

---

## 1. 总目标

完成这一轮学习后，应能独立回答并实现以下问题：

- Agent 和普通 LLM 调用的区别是什么
- 一个 Agent 为什么需要编排层、工具层、状态层、权限层
- 工具如何定义、发现、选择、执行、扩展
- 如何让 Agent 的输出结构化、可测试、可复盘
- 如何处理高风险工具、权限确认、CI 环境、跨平台问题
- 如何从单 Agent 演进到更复杂的多 Agent / Memory / Evaluation 系统

---

## 2. 学习总路线

建议按 8 个阶段学习，每个阶段都要有：

- 学习目标
- 代码实现
- 测试验证
- 复盘总结

阶段顺序：

1. Agent 基本认知
2. Tool System（工具系统）
3. Orchestrator（编排层）
4. Context / State（上下文与状态）
5. Permission / Sandbox（权限与执行安全）
6. Structured Output / Schema（结构化输出）
7. Evaluation / Observability（评测与可观测性）
8. Multi-Agent / Production（多代理与工程化）

---

## 3. 阶段 0：准备期

### 学习目标

- 建立项目整体认知
- 搭好本地运行、测试、提交、PR 的最小工作流

### 需要掌握

- 项目目录结构
- `CLI -> Orchestrator -> Tools -> Output` 总链路
- Git 分支、PR、CI、测试回归的基本流程

### 实践任务

- 读完 `README.md`
- 读完 `docs/project_plan.md`
- 读完 `docs/shared_contracts.md`
- 读完 `docs/cli_tools_orchestrator_contract.md`
- 画出项目调用链图

### 产出物

- 一张项目架构图
- 一份“我理解的系统主链路”笔记

### 验收标准

- 能口头解释当前项目有哪些层
- 能说清每层负责什么，不负责什么

---

## 4. 阶段 1：Agent 基本认知

### 学习目标

- 理解 Agent 不是“更大的 prompt”，而是“可决策的执行系统”

### 需要掌握

- 什么是 Agent
- 什么是 Tool Calling
- 什么是规划、执行、观察、反馈闭环
- 什么是 ReAct / Plan-Act-Observe 思路

### 重点名词

- `Agent`：能根据目标自主做步骤决策的系统
- `Tool Calling`：模型按 schema 调用外部能力
- `Loop`：Agent 反复进行“分析 -> 执行 -> 更新状态”的循环
- `Fallback`：失败时的降级方案

### 实践任务

- 不看代码，自己写一版“一个 Agent 最少需要哪些模块”
- 再对照项目现有实现做差异分析

### 产出物

- 《Agent 与普通 ChatBot 的区别》
- 《为什么 Agent 一定会涉及状态和工具》

### 验收标准

- 能解释为什么没有工具的 Agent 很弱
- 能解释为什么没有编排的 Tool Calling 只是函数调用，不是 Agent

---

## 5. 阶段 2：Tool System（工具系统）

### 学习目标

- 掌握“如何把能力设计成可被模型调用的工具”

### 需要掌握

- 统一工具接口
- 工具输入建模
- 工具描述写法
- 工具发现与注册
- 工具风险分级

### 重点名词

- `ToolSpec`：工具说明书，给模型看
- `BaseTool`：工具统一接口
- `ToolRegistry`：工具注册表
- `ToolResult`：工具统一返回信封
- `ToolSafety`：风险分级（`readonly / write / execute`）
- `JSON Schema`：描述 JSON 数据结构的规范
- `Pydantic`：输入校验 + schema 生成工具
- `Default Least Privilege`：默认最小权限原则

### 你需要从本项目里重点学习的点

- readonly MVP 为什么先做 `read_file / glob_files / grep_files / list_dir`
- 为什么 `run_command` 实现了，但不放进默认 registry
- 为什么 `description` 要写得像“给模型看的说明书”
- 为什么输入模型要比函数参数更结构化

### 实践任务

- 手写一个新工具的 `ToolSpec`
- 比较 `read_file` 和 `grep_files` 的使用场景差异
- 自己总结“一个好工具 description 应该包含什么”

### 产出物

- 《工具系统设计六问》
- 《本项目工具层抽象总结》

### 验收标准

- 能独立写出一个简单工具
- 能解释 registry 存在的必要性
- 能解释为什么工具要用 schema，而不是裸参数

---

## 6. 阶段 3：Orchestrator（编排层）

### 学习目标

- 掌握 Agent 不是直接“模型调工具”，而是“编排层调度模型与工具”

### 需要掌握

- 5 阶段编排
- tool plan 执行流程
- 终止条件
- 错误聚合
- 并发与串行策略

### 重点名词

- `Orchestrator`：编排层，负责流程控制
- `AnalysisPlan`：模型返回的结构化工具计划
- `execute_tools`：执行工具计划的阶段
- `should_continue`：决定是否继续循环
- `Concurrency-safe`：是否可安全并发
- `Conservative concurrency`：保守并发

### 你需要从本项目里重点学习的点

- 为什么高风险工具与只读工具走不同分支
- 为什么只读工具也不是一股脑全并发
- 为什么结果顺序要和原始 `tool_calls` 一致

### 实践任务

- 手画 `run_review()` 的时序图
- 对照 `execute_tools()` 写出执行流程说明
- 总结编排层和工具层的边界

### 产出物

- 《5 阶段编排说明》
- 《工具调度执行流程图》

### 验收标准

- 能解释为什么编排层不能下沉到工具里
- 能解释为什么工具层不能决定全局顺序

---

## 7. 阶段 4：Context / State（上下文与状态）

### 学习目标

- 理解 Agent 必须有“记忆”，否则每一步都像重新开始

### 需要掌握

- `ContextState`
- `DecisionStep`
- `ErrorDetail`
- 当前文件集、约束集、错误集

### 重点名词

- `ContextState`：一次运行中的共享状态
- `DecisionStep`：关键决策记录
- `ErrorDetail`：结构化错误
- `Constraint`：当前运行的限制条件
- `ContextVar`：协程/上下文级变量

### 你需要从本项目里重点学习的点

- 为什么路径权限最后用 `ContextVar` 注入工作区根
- 为什么错误不能只用字符串表示
- 为什么计划模式要把 `plan_mode` 写进 `constraints`

### 实践任务

- 给 `ContextState` 画字段关系图
- 解释 `cwd` 和 `repo_path` 的区别
- 解释为什么只看 `cwd` 会出错

### 产出物

- 《状态系统设计笔记》
- 《路径上下文踩坑复盘》

### 验收标准

- 能解释上下文和普通局部变量的区别
- 能解释什么信息应该进 `ContextState`

---

## 8. 阶段 5：Permission / Sandbox（权限与执行安全）

### 学习目标

- 建立“Agent 有能力，就必须有权限控制”的安全意识

### 需要掌握

- 风险分级
- 默认最小权限
- 权限模式
- 高风险工具确认
- CI 环境拒绝策略
- 命令执行沙箱

### 重点名词

- `Permission Mode`：系统级权限模式
- `default`：正常模式
- `plan`：只计划不执行
- `Sandbox`：受限执行环境
- `Timeout`：超时控制
- `cwd restriction`：工作目录限制
- `CI=true`：非交互自动化环境信号

### 你需要从本项目里重点学习的点

- 为什么 `run_command` 不默认注册
- 为什么 `CI=true` 时 execute 默认拒绝
- 为什么 execute 工具要下沉到 sandbox 层

### 实践任务

- 自己总结“默认最小权限”的含义
- 解释 `readonly / write / execute` 三类工具的差异
- 给 `run_command` 设计一套更完整的风险分析思路

### 产出物

- 《权限模型学习笔记》
- 《execute 工具的安全设计》

### 验收标准

- 能解释为什么 Agent 的能力越强，安全系统越重要
- 能解释为什么 `plan mode` 有意义

---

## 9. 阶段 6：Structured Output / Schema（结构化输出）

### 学习目标

- 学会让模型输出稳定、可消费、可测试的结果

### 需要掌握

- 结构化输出
- 伪工具 schema
- 输出模型与 CLI/API/CI 的接口关系

### 重点名词

- `Structured Output`：结构化输出
- `submit_review` / `submit_debug`：伪工具，用于约束输出
- `ReviewReport` / `DebugResponse`：最终消费模型
- `Schema contract`：跨层协议

### 你需要从本项目里重点学习的点

- 为什么最终输出不能随模型自由发挥
- 为什么要让输出字段稳定
- 为什么 schema 不只是给工具，也可以给最终结果

### 实践任务

- 画出 `tool schema` 和 `output schema` 的区别
- 总结“结构化输出对评测有什么好处”

### 产出物

- 《输出协议与 schema 学习笔记》

### 验收标准

- 能解释为什么“能看懂”不等于“可工程化消费”

---

## 10. 阶段 7：Evaluation / Observability（评测与可观测性）

### 学习目标

- 理解 Agent 不只是“能跑”，还要“可测、可复盘、可比较”

### 需要掌握

- event log
- run_id
- decisions trace
- token budget
- regression test

### 重点名词

- `run_id`：一次运行的唯一标识
- `Event Log`：事件日志
- `Trace`：执行轨迹
- `Token Budget`：token 预算
- `Regression Test`：回归测试

### 你需要从本项目里重点学习的点

- 为什么要记录工具调用序列
- 为什么 `DecisionStep` 有助于复盘
- 为什么边界测试比 happy path 更重要

### 实践任务

- 列出本项目目前有哪些回归测试类型
- 总结这次 CI 暴露了哪些跨环境问题

### 产出物

- 《CI 与测试踩坑复盘》
- 《回归测试设计清单》

### 验收标准

- 能说清“本地通过”和“工程稳定”之间差了什么

---

## 11. 阶段 8：Multi-Agent / Production（进阶）

### 学习目标

- 在掌握单 Agent 后，理解如何扩展到更复杂系统

### 后续可继续学习

- 更完整的 `ToolUseContext`
- 命令安全分析器
- 安全写工具（edit / patch）
- memory / retrieval
- task queue / background jobs
- multi-agent delegation
- eval benchmark
- production deployment

### 验收标准

- 能清楚知道“现在项目到了哪一步”
- 能判断“下一个工程化里程碑是什么”

---

## 12. 每阶段统一复盘模板

每完成一个阶段，回答下面 6 个问题：

1. 这个阶段解决了什么问题  
2. 为什么不能用更简单的方法代替  
3. 当前实现里最关键的抽象是什么  
4. 我踩过什么坑  
5. 这些坑背后的本质原因是什么  
6. 如果让我重做，我会怎么设计

---

## 13. 学习节奏建议

### 第 1 周

- 吃透项目结构
- 吃透 Tool 系统
- 吃透 4 个 readonly MVP 工具

### 第 2 周

- 吃透 Orchestrator
- 吃透 ContextState
- 吃透 tool schema 与输出 schema

### 第 3 周

- 吃透 execute/sandbox/permission
- 吃透测试与 CI 踩坑
- 写一版自己的 Agent 设计总结

### 第 4 周

- 做一次从零复现
- 尝试自己扩展一个新工具或一个新权限模式
- 输出完整学习笔记和项目讲解稿

---

## 14. 最终验收：你是否真的学会了

如果你已经能做到以下几点，说明这一轮学习基本完成：

- 能从零解释项目整体架构
- 能说清工具调用完整链路
- 能解释为什么要有 registry、schema、state、permission、sandbox
- 能独立实现一个新工具并写测试
- 能解释本项目踩过的坑和为什么会踩
- 能向别人讲清楚“一个完整 Agent 系统该怎么搭”

