# Interface Module (User Input)

## 模块职责
负责解析用户输入的自然语言任务文件，提取任务目标与约束条件，将其转化为程序可理解的结构化数据。

## 1. 核心功能
- 读取输入文件。
- 格式校验。
- 提取任务元数据（语言、目标站点）。

## 2. 输入 (Input)
- **来源**: 用户提供的文本文件或命令行参数。
- **格式**: 自然语言字符串。
  > 示例: "请帮我打开谷歌，搜索现在的美元汇率"

## 3. 输出 (Output)
- **流向**: Orchestrator (传入 Observe 阶段)。
- **数据结构 (JSON)**:
  ```json
  {
    "task_text": "string (用户的原始任务描述)",
    "constraints": {
      "site": "string (可选: 目标域名)",
      "language": "string (可选: zh-CN/en-US)",
      "done_criteria": "string (可选: 完成判定标准)"
    }
  }
  ```
