# HCLR 采集协议规范（v1.0）

> 版本：1.0.0（2026-08-12）｜ 作者：Rootosophy ｜ English: [PROTOCOL.md](PROTOCOL.md)
>
> 本文档定义**如何在任意AI对话系统中持续采集HCLR数据**。它是方法论文（[PAPER_CN.md](PAPER_CN.md)）的落地规范：论文定义"量什么"，本规范定义"怎么量"。

## 1. 定位

HCLR（人类认知杠杆率）= 模型输出总和 / 使用者介入总和，用于衡量使用者的判断对AI输出的影响程度。采集协议是HCLR的**常驻测量层**：它不按需触发，而是在每一次对话中自动记录任务事件。

```text
HCLR = ΣO / ΣI
  O = 任务事件内模型输出Token（或字符）总和，含全部生成轮次
  I = 使用者介入Token（或字符）总和，不含初始任务描述
```

## 2. 核心概念

| 概念 | 定义 |
|---|---|
| 任务事件 | 一个有主题的对话单元：从使用者提出请求到成果产出（可含多轮交互） |
| O0 | 模型的初始生成（冻结保存，不可覆盖） |
| 介入 | 使用者针对模型输出的反馈/纠正/约束/方向调整（不计任务描述） |
| O1 | 介入后的最终成果 |
| C1 | 第一轮确认（adopt / partial / reject）——成果是否被采纳 |
| C2 | 第二轮确认（approved / partial / rejected / pending，或自定义采纳率）——成果是否获受众认可 |
| C2_source | C2来源：user（显式）/ landing（落地即确认）/ auto_timeout（24h默认）/ auto_confirm（采信） |
| 状态 | S0未采用 / S1已采用待确认 / S2已采用未获认可（C2<0.5）/ S3已采用并获认可（C2≥0.5） |

## 3. 采集时机（钩子）

| 时机 | 动作 |
|---|---|
| 任务事件开始 | 识别有主题的请求，记录 `task_description`（不计入I），冻结O0 |
| 每次模型输出 | 累加O（输出字符/Token） |
| 每次使用者介入 | 记录介入文本，累加I |
| 任务事件结束 | 保存O1，标记任务事件边界 |
| 会话结束 | **请求C1**（一行确认：adopt / partial / reject） |
| 延迟回访 | 成果实际使用后**请求C2**（approved / rejected / pending） |

## 4. 记录字段

与 [schema/hclr-record.schema.json](schema/hclr-record.schema.json) 一一对应：

```text
task_id, domain, model, audience,
task_description（不计I）, O0, O1, O_total,
interventions[ {seq, text, kind, timestamp} ], I, I_metric,
C1, C1_note, C2, C2_note, C2_source, c2_requested_at, status, created_at, period
```

## 5. 口径规则

1. **自动记录场景**默认以Token（或字符）计介入量；**人工记录场景**默认以判断数计。
2. **同一曲线内不得混用口径**（token/字符/判断数）。
3. I **不含**初始任务描述；O **含**全部生成轮次。
4. 测量边界：Token只测显性表达形式，不测认知成本与信息价值；使用者可通过压缩表达、合并命题、省略依据人为抬高比率——**采集时须保留原始记录以便复核**。
5. 单样本即可运行：`HCLR_j = O_j/I_j` 在第一个任务事件产生时即可计算；多样本用于趋势与稳定性。

## 6. 确认流程（双重确认 + C2 自动闭环）

```text
C1（第一轮，会话结束时）: adopt / partial / reject
  → 采纳进入S1，未采纳进入S0
C2（第二轮，成果实际使用后）: approved / partial / rejected（或自定义采纳率）
  → 认可（≥0.5）进入S3，未认可（<0.5）进入S2
```

**C2 自动闭环采集**（确认不因缺失反馈而中断）：

| 来源 | 触发条件 | 取值 |
|---|---|---|
| `landing` | 成果已在本地落地（保存/执行完成且使用者未撤销） | 视为 100% 认可 |
| `user` | 使用者显式返回认可度 | 按返回值 |
| `auto_timeout` | 发出确认请求后 24 小时未响应 | 默认视为认可 |
| `auto_confirm` | 主动再次确认后仍未响应 | 采信默认认可 |

- C1由使用者本人确认；C2以真实受众反馈为准；
- **落地即确认**：成果被实际保存/执行且未被撤销，是认可的最强隐式信号，无需另行询问；
- **缺失反馈不中断**：超时按上述规则给出默认值并标注来源（auto_timeout / auto_confirm），延迟反馈可回填覆盖；
- 显式来源（user/landing）与自动默认（auto_timeout/auto_confirm）分开统计，报告须披露各来源占比；
- 延迟结果回填原批次，不单独计为新任务事件。

## 7. 隐私与数据

- 原始记录（含完整对话）**本地保存**，不随公开仓库发布；
- 公开示例（如 [examples/PILOT_CASE_CN.md](examples/PILOT_CASE_CN.md)）为**匿名化**整理；
- 单条记录建议字段：O0/O1可保留匿名化文本，真实姓名、受众身份、完整反馈非必需。

## 8. 参考实现

| 组件 | 说明 |
|---|---|
| [pilot-data/pilot.py](pilot-data/pilot.py) | 命令行记录工具（new / record / c1 / c2 / c2req / pending / report），git忽略，本地使用 |
| [schema/hclr-record.schema.json](schema/hclr-record.schema.json) | 记录数据格式（v0.3，含 C2_source / c2_requested_at） |
| 自动化建议 | 在对话系统加消息钩子：assistant输出累加O、user介入累加I、会话结束自动请求C1 |

## 9. 平台适配

| 场景 | 采集方式 |
|---|---|
| Hermes（当前） | 对话内嵌采集：assistant在会话中记录O/I，会话结束请求C1 |
| 其他Agent框架 | 消息中间件/钩子：监听assistant/user消息，按字段记录 |
| 手动模式 | 任何工具可用：对话结束后按本规范人工填写记录 |
