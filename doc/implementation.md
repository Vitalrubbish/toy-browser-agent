# 阶段设计

### 1. 输入阶段（用户任务）
- 用户将自然语言任务写入输入文件，将其整理为 JSON 格式。

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
  - `task_text`: string（用户的原始任务描述，作为后续阶段的主目标）
  - `constraints`: { `site`?: string, `language`?: string, `done_criteria`?: string }（可选约束，用于限定站点、语言与完成标准）
    - `site`?: string（期望访问的目标站点或域名）
    - `language`?: string（用户偏好语言，用于页面内容或输入输出）
    - `done_criteria`?: string（完成判定条件的文本描述）

### 2 → 3（观察阶段 → 连接阶段）
- **数据内容**：页面结构与交互元素摘要、截图、页面统计
- **数据格式（JSON）**：
  - `url`: string（当前页面 URL，用于定位站点与上下文）
  - `page_stats`: { `links`: number, `inputs`: number, `buttons`: number }（页面统计，用于估计复杂度与交互密度）
    - `links`: number（链接数量）
    - `inputs`: number（输入控件数量）
    - `buttons`: number（按钮数量）
  - `interactive_elements`: Array<{ `id`: number, `type`: string, `label`: string, `attributes`: object }>（可交互元素摘要列表，用于动作选择）
    - `id`: number（元素索引，作为动作目标的唯一标识）
    - `type`: string（元素类型，如 button/input/link）
    - `label`: string（元素可见文本或可访问名称）
    - `attributes`: object（关键属性摘要，如 placeholder、name 等）
  - `dom_snapshot_ref`: string（原始 DOM 的引用或哈希，用于追溯）
  - `a11y_tree_ref`: string（可访问性树引用或哈希，用于语义对齐）
  - `screenshot_ref`: string（截图存储位置或哈希，用于视觉对齐）

### 3 → 4（连接阶段 → 记忆检索阶段）
- **数据内容**：用于检索的查询包（任务 + 页面摘要 + 可选上下文）
- **数据格式（JSON）**：
  - `query`: { `task_text`: string, `url`?: string, `domain`?: string, `page_hint`: string }（检索查询主体）
    - `task_text`: string（当前任务文本，用于语义检索）
    - `url`?: string（页面 URL，用于站点级相关性）
    - `domain`?: string（域名，用于跨页面匹配）
    - `page_hint`: string（页面摘要提示，如页面类型或功能）
  - `elements_brief`: Array<{ `id`: number, `type`: string, `label`: string }>（精简元素列表，用于匹配历史操作目标）
    - `id`: number（元素索引）
    - `type`: string（元素类型）
    - `label`: string（元素标签/文本）

### 4 → 5（记忆检索阶段 → 思考阶段）
- **数据内容**：相关记忆轨迹、可复用操作模式、规避提示
- **数据格式（JSON）**：
  - `memory_hits`: Array<{ `task_text`: string, `steps`: Array<object>, `success`: boolean }>（历史任务检索结果）
    - `task_text`: string（历史任务文本）
    - `steps`: Array<object>（历史动作序列）
    - `success`: boolean（是否成功完成）
  - `action_patterns`: Array<{ `name`: string, `steps`: Array<object> }>（可复用操作模板）
    - `name`: string（模式名称，如“搜索-提交”）
    - `steps`: Array<object>（模式步骤序列）
  - `avoidance`: Array<string>（规避提示列表，如常见失败原因）

### 5 → 6（思考阶段 → 行动阶段）
- **数据内容**：模型输出的动作指令与状态判定
- **数据格式（JSON）**：
  - `action`: { `type`: string, `target_id`?: number, `value`?: string, `options`?: object }（单步动作指令）
    - `type`: string（动作类型，如 click/input/scroll）
    - `target_id`?: number（目标元素索引）
    - `value`?: string（输入值或附加文本）
    - `options`?: object（动作参数，如滚动距离、按键组合）
  - `is_done`: boolean（是否判定任务完成）
  - `reason`?: string（完成/未完成的简要原因）

### 6 → 7（行动阶段 → 记忆写入阶段）
- **数据内容**：执行结果与轨迹记录
- **数据格式（JSON）**：
  - `step_trace`: { `action`: object, `result`: { `success`: boolean, `error`?: string }, `timestamp`: string }（单步执行记录）
    - `action`: object（执行的动作指令）
    - `result`: { `success`: boolean, `error`?: string }（执行结果）
      - `success`: boolean（是否成功）
      - `error`?: string（失败原因）
    - `timestamp`: string（执行时间）
  - `task_summary`: { `task_text`: string, `final_status`: string, `steps`: Array<object> }（任务级汇总）
    - `task_text`: string（任务文本）
    - `final_status`: string（最终状态，如 success/failed）
    - `steps`: Array<object>（完整动作序列）
