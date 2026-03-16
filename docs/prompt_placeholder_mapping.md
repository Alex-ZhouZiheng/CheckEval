# Prompt 占位符在 CheckEval 项目中的落地映射

本仓库当前代码路径里，在线推理阶段使用的是 `inference_checkeval.py` 中的评测模板（打分问答模板），而不是“生成子问题”的元提示模板。

## 1) 项目里实际被调用的 prompt

`src/inference_checkeval.py` 定义了两套模板：
- `summeval_template`：给新闻原文+摘要+问题列表，要求回答 Yes/No。
- `topical_chat_template`：给对话历史+fact+候选回复+问题列表，要求回答 Yes/No。

随后代码会把 `<aspect>`, `<definition>`, `<source>/<summary>` 或 `<document>/<fact>/<response>`, `<questions>` 等占位符替换成真实内容后发送给模型。

## 2) 你给出的元提示与项目字段的对应关系

对于你给出的模板：
- `{dimension}` ↔ `aspect`（命令行 `--aspects` 传入）
- `{def}` ↔ YAML 里的 `definition[template_type]`
- `{key components}` ↔ YAML 中 `sub_aspect` 的一级键（例如 `Logical Flow`）
- “seed name and question pairs” ↔ `sub_aspect` 下的问题条目（seed/diversification/elaboration 三个版本）
- `{benchmark info}` ↔ 推理时输入样本字段（summeval: source/system_output；topical_chat: document/fact/response）

## 3) constraints 与 conditions 在本仓库里的“实际内容”

仓库中没有单独的 `constraints` / `conditions` 文本变量文件；它们是**由问题 YAML 和模板结构隐式约束**出来的：

- **constraints（过程约束）**：
  1. 只围绕单一 `aspect` + `definition` 评测。
  2. 问题来源固定为 `sub_aspect` 列表；每个问题最终被编号为 `Q1...Qn`。
  3. 输出格式被硬性规定为 `Qk: Yes/No`（通过模板与后处理正则共同约束）。

- **conditions（好问题条件）**：
  1. 问题必须落在对应 `sub_aspect` 语义内（例如 coherence 下的 Logical Flow / Continuity / Relevance）。
  2. diversification 版本强调覆盖更多边界与变体；elaboration 版本强调对 seed 问题的细化拆分。
  3. 每个问题都可被二元判断（Yes/No），便于稳定聚合。

因此，如果把你的“子问题生成模板”映射到仓库实现：
- `constraints` 最接近“模板输出格式 + 二元回答协议 + 按 aspect 约束”。
- `conditions` 最接近“YAML 中 sub_aspect 问题编写原则（相关性、可判定性、覆盖度）”。
