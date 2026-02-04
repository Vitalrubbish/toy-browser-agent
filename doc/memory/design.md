# Memory Module (RAG & Storage)

## 模块职责
管理长期记忆库。包括在决策前检索相似的历史成功案例，以及在决策后记录当前的轨迹。

---

## 子功能 1: Retrieval (检索)

### 1. 输入 (Input)
- **来源**: Agent (Prompt构建前)。
- **数据结构**:
  ```json
  {
    "query": {
      "task_text": "string (当前任务)",
      "url": "string",
      "domain": "string",
      "page_hint": "string (页面特征摘要)"
    },
    "elements_brief": [
      { "id": "number", "type": "string", "label": "string" }
    ]
  }
  ```

### 2. 输出 (Output)
- **流向**: Agent (注入 Prompt)。
- **数据结构**:
  ```json
  {
    "avoidance": ["string (失败避坑指南)"],
    "memory_hits": [
      {
        "task_text": "string (相似的历史任务)",
        "steps": ["object (历史动作序列)"],
        "success": "boolean"
      }
    ],
    "action_patterns": [
      { "name": "string", "steps": ["object"] }
    ]
  }
  ```

---

## 子功能 2: Writeback (写入)

### 1. 输入 (Input)
- **来源**: Orchestrator (单步执行后 / 任务结束后)。
- **数据结构**:
  - **单步时**: 接收 `step_trace` (见 Browser 模块输出)。
  - **任务结束时**:
    ```json
    {
      "task_summary": {
        "task_text": "string",
        "final_status": "string (success/failed)",
        "steps": ["object (完整动作序列)"]
      }
    }
    ```

### 2. 输出 (Output)
- **流向**: Database/File System。
- **内容**: Boolean (写入成功/失败状态)。
