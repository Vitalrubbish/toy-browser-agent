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
  - **单步时**: 
    ```json
    {
      "url": "string (当前 URL)",
      "page_stats": {
        "links": "number",
        "inputs": "number",
        "buttons": "number"
      },
      "interactive_elements": [
        {
          "id": "number (元素索引)",
          "type": "string (标签类型)",
          "label": "string (文本内容)",
          "attributes": "object (关键属性)"
        }
      ],
      "dom_snapshot_ref": "string (DOM存储引用/Hash)",
      "a11y_tree_ref": "string (AccessTree引用/Hash)",
      "screenshot_ref": "string (截图文件路径)"
    }
    ```
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

---

## 记忆的存储结构 JSON 单元

### 单步执行情况存储结构
```json
{
  "step_id": "string (UUID)", // 系统自动生成
  "timestamp": "string",      // 系统当前时间
  "task_context": {
    "task_text": "string",    // 用户的输入
    "url": "string"           // observer 当前的 url
  },
  "observation": {
    "page_hint": "string", // agent 对 observer 输出的理解或者摘要
    "elements": [
      { "id": "number", "type": "string", "label": "string" }
    ] // 交互元素列表
  },
  "action": "object (来自 Executor 输出的 step_trace.action)", // 操作
  "result": "object (来自 Executor 输出的 step_trace.result)"  // 执行结果（成功/报错）
}
```

### 整体执行情况存储结构
```json
{
  "session_id": "string (UUID)", // 会话标识符
  "task_info": {
    "task_text": "string",       // 任务描述
    "domain": "string",          // 任务的主要 URL
    "start_url": "string"        // 任务起始 URL
  },
  "final_status": "string (success/failed)", // 是否成功
  "steps": [
    {
      "step_number": "number",   // 编号
      "action": "object",        // 操作
      "url": "string",
      "observation_hint": "string",
      "success": "boolean"
    }
  ]
}
```


