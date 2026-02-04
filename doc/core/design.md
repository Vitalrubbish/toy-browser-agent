# Orchestrator Module (Main Loop)

## 模块职责
程序的入口与主控制器。不包含具体业务逻辑，仅负责模块间的调度与数据流转。

## 控制流程算法

1. **Init**: 调用 Interface 解析任务。
2. **Loop**:
   a. 调用 Browser.Observer 获取 `browser_state`。
   b. 调用 Memory.Retrieval 获取 `memory_context`。
   c. 将 (任务 + 状态 + 记忆) 传给 Agent。
   d. 获取 Agent 决策结果 (`action` 和 `is_done`)。
   e. **IF** `is_done` == true:
      - 调用 Memory.Writeback 记录 `task_summary`。
      - 退出程序。
   f. **ELSE**:
      - 调用 Browser.Executor 执行 `action`。
      - 获取执行结果 `result`。
      - 调用 Memory.Writeback 记录 `step_trace`。
      - 继续循环。
