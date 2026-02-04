# Agent Module (Prompt & LLM)

## 模块职责
代理的大脑。负责接收多模态信息（截图、DOM、记忆），组装 Prompt，调用 LLM，并解析返回的决策。

## 1. 核心流程
1. **Context Builder**: 将 Interface 的任务、Browser 的状态、Memory 的检索结果组装成 XML 格式 Prompt。
2. **Planner**: 调用 LLM API (如 GPT-4o / Gemini)。
3. **Parser**: 提取 JSON 动作指令。

## 2. 输入 (Input)
- **来源**: Orchestrator 汇总的信息。
- **内容**:
  - `user_task` (来自 Interface)
  - `browser_state` (来自 Browser Observer)
  - `memory_context` (来自 Memory Retrieval)

## 3. 输出 (Output)
- **流向**: Orchestrator (分发给 Browser Executor 或终止程序)。
- **数据结构 (JSON)**:
  ```json
  {
    "action": {
      "type": "string",
      "target_id": "number",
      "value": "string"
    },
    "is_done": "boolean (任务完成标志)",
    "reason": "string (自我陈述的原因)"
  }
  ```
