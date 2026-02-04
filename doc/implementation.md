# 阶段设计

### 1. 输入阶段（用户任务）
- 用户将自然语言任务写入输入文件。

### 2. 观察阶段（获取网页信息）
- 获取原始 DOM 与 Accessibility Tree。
- 获取浏览器当前截图。

### 3. 连接阶段（构建上下文）
- 将用户输入与浏览器状态拼装为 context。
- **Prompt 结构**使用 XML 标签分隔不同信息源：
  ```xml
  <agent_state> ... </agent_state>
  <browser_state> ... </browser_state>
  <agent_history> ... </agent_history>
  <memory_retrieval> ... </memory_retrieval>
  ```
- **关键点**：传入可交互元素的**索引列表**，避免直接传原始 HTML。

### 4. 记忆检索阶段（新增，调用点）
- **位置**：在“连接阶段”之后、进入“思考阶段”之前调用。
- **输入**：
  - 当前任务文本
  - 当前页面的可交互元素索引摘要
  - 可选：页面 URL、站点域名、任务类型标签
- **输出**：
  - 相关历史任务的成功轨迹（步骤序列）
  - 关键操作模式（例如：搜索框输入 + 提交）
  - 失败轨迹的规避提示
- **目的**：为决策生成提供候选动作与先验路径。

### 5. 思考阶段：决策生成
- **输入**：
  - 格式化 Prompt
  - 屏幕截图
  - **记忆检索结果**（注入 `<memory_retrieval>`）
- **任务**：
  1. 理解用户指令
  2. 对齐当前页面元素
  3. 在记忆建议与当前页面之间做约束匹配
  4. 决定下一步动作
  5. 判断任务是否完成
- **输出**：符合约束的 JSON 动作结构。

### 6. 行动阶段：执行操作
- 解析 LLM 输出的 JSON。
- 发送指令执行真实浏览器操作。

### 7. 记忆写入阶段
- **位置**：在每步动作执行完成后，以及任务结束时。
- **写入内容**：
  - 任务描述 + 页面摘要 + 动作序列 + 执行结果（成功/失败）
  - 关键可复用片段（成功子路径）
- **用途**：为后续任务提供可检索的成功轨迹。

---

# 阶段间数据传输汇总（内容 + 格式）

### 1 → 2（输入阶段 → 观察阶段）
- **数据内容**：用户任务文本、可选任务约束（站点/语言/完成标准）
- **数据格式（JSON）**：
  - `task_text`: string
  - `constraints`: { `site`?: string, `language`?: string, `done_criteria`?: string }

### 2 → 3（观察阶段 → 连接阶段）
- **数据内容**：页面结构与交互元素摘要、截图、页面统计
- **数据格式（JSON）**：
  - `url`: string
  - `page_stats`: { `links`: number, `inputs`: number, `buttons`: number }
  - `interactive_elements`: Array<{ `id`: number, `type`: string, `label`: string, `attributes`: object }>
  - `dom_snapshot_ref`: string（指向原始 DOM 的引用/哈希）
  - `a11y_tree_ref`: string（指向 Accessibility Tree 的引用/哈希）
  - `screenshot_ref`: string（截图存储位置/哈希）

### 3 → 4（连接阶段 → 记忆检索阶段）
- **数据内容**：用于检索的查询包（任务 + 页面摘要 + 可选上下文）
- **数据格式（JSON）**：
  - `query`: { `task_text`: string, `url`?: string, `domain`?: string, `page_hint`: string }
  - `elements_brief`: Array<{ `id`: number, `type`: string, `label`: string }>

### 4 → 5（记忆检索阶段 → 思考阶段）
- **数据内容**：相关记忆轨迹、可复用操作模式、规避提示
- **数据格式（JSON）**：
  - `memory_hits`: Array<{ `task_text`: string, `steps`: Array<object>, `success`: boolean }>
  - `action_patterns`: Array<{ `name`: string, `steps`: Array<object> }>
  - `avoidance`: Array<string>

### 5 → 6（思考阶段 → 行动阶段）
- **数据内容**：模型输出的动作指令与状态判定
- **数据格式（JSON）**：
  - `action`: { `type`: string, `target_id`?: number, `value`?: string, `options`?: object }
  - `is_done`: boolean
  - `reason`?: string

### 6 → 7（行动阶段 → 记忆写入阶段）
- **数据内容**：执行结果与轨迹记录
- **数据格式（JSON）**：
  - `step_trace`: { `action`: object, `result`: { `success`: boolean, `error`?: string }, `timestamp`: string }
  - `task_summary`: { `task_text`: string, `final_status`: string, `steps`: Array<object> }
