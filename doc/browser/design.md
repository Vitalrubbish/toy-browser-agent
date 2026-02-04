# Browser Module (Observer & Executor)

## 模块职责
负责控制浏览器实例（如 Playwright/Selenium）。包含两个子功能：
1. **Observer**: 获取当前页面的状态（DOM、截图、可访问性树）。
2. **Executor**: 执行具体的动作指令（点击、输入、滚动）。

---

## 子功能 1: Observer (观察者)

### 1. 输入 (Input)
- **来源**: Orchestrator (调用指令)。
- **内容**: void (无特定输入，基于当前页面状态)。

### 2. 输出 (Output)
- **流向**: Agent 模块 (用于构建 Context)。
- **数据结构 (JSON)**:
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

---

## 子功能 2: Executor (执行者)

### 1. 输入 (Input)
- **来源**: Agent (思考后的决策)。
- **数据结构 (JSON)**:
  ```json
  {
    "action": {
      "type": "string (click/input/scroll)",
      "target_id": "number (目标元素ID)",
      "value": "string (输入内容)",
      "options": "object"
    }
  }
  ```

### 2. 输出 (Output)
- **流向**: Memory (记录执行结果)。
- **数据结构 (JSON)**:
  ```json
  {
    "step_trace": {
      "action": "object (回传输入的动作)",
      "result": {
        "success": "boolean",
        "error": "string (如果失败，记录报错信息)"
      },
      "timestamp": "string"
    }
  }
  ```
