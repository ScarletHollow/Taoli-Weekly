---
name: monitor-china-models
description: "采集国内 9 大模型(DeepSeek/Kimi/通义千问/豆包/腾讯混元/智谱GLM/百度文心/MiniMax/科大讯飞星火)的文档视觉、Agent 工作流、生产应用三类能力更新,只记录可能改变教育业务的能力变化,输出 A 分册候选信号 JSON。用于 D 执行手册 D3「模型更新采集」,当需要监测国内模型能力变化、判断其对桃李教育业务的影响时使用。"
---

# monitor-china-models — 国内大模型能力更新监测

## 一、用途

采集类 Skill。监测国内 9 家一级模型/平台的**官方能力更新**,识别其中**可能改变教育业务的能力变化**,输出 A 分册信号 JSON(状态 `candidate`)。

**定位**（呼应主文档「四、国内模型与 AI 平台观察名单」）：国内模型本身不是周报主角。只有模型能力可能改变教育业务时才进入正文；纯参数、榜单、发布会口号一律不记录。

- 监测对象（一级模型名单，默认全部，可传子集）：DeepSeek、Kimi／月之暗面、通义千问、豆包、腾讯混元、智谱 GLM、百度文心、MiniMax、科大讯飞星火。
- 监测能力（三类，能力子项见「输入 schema」）：文档与视觉、Agent 与工作流、生产应用。
- 产出：能力变化候选信号 JSON，供 D 分册 D5「保存候选 + 去重 + 分类 + 初评」追加写入 `signals/signals-YYYY-WW.jsonl`。

**边界（红线，本 Skill 不做）**：

- **不去重**：事件级去重归 **D 执行手册 D5 步骤**；本 Skill 只在采集时按 A 分册第 5 节构造 `dedup_key`，供 D5 使用。
- **不发布**：本 Skill 不产生任何自动发布行为；发布归 **D 执行手册节点 4 人工操作**，AI 不自动发布。
- **不评分**：六维评分由 `score-taoli-relevance` 按 E 评分锚点手册初评、人工终审修正；本 Skill 采集阶段不判分。
- **不核查 B 级线索**：B 级媒体线索的逐条回源核查归 `verify-source-evidence`（D 分册 D4）；本 Skill 只以官方 S/A 级源为记录基准。

## 二、输入 schema

```json
{
  "_schema": "2026-08-v1",
  "model_list": ["DeepSeek", "Kimi", "通义千问", "豆包", "腾讯混元", "智谱GLM", "百度文心", "MiniMax", "科大讯飞星火"],
  "capability_scope": ["document_vision", "agent_workflow", "production"],
  "time_window": {"start": "2026-08-03", "end": "2026-08-07", "week": "2026-W33"},
  "context": {
    "taoli_business": "桃李核心业务与在研项目摘要(供判定相关性)",
    "history": "上周 signals-YYYY-WW.jsonl 的 dedup_key 清单(供构造去重键与判断新旧)"
  }
}
```

字段说明：

| 字段 | 必填 | 说明 |
|---|---|---|
| model_list | 是 | 本次监测的模型名单；默认 9 家，可传子集。每家即使无更新也须完成一次信息检查。 |
| capability_scope | 是 | 本次监测的三类能力，默认全选。 |
| time_window | 是 | 监测时间窗（起止日期 + ISO 周号）。 |
| context.taoli_business | 是 | 桃李业务画像，用于判定「能力变化对教育业务/桃李是否有实质影响」；具体以 D 执行手册维护的最新版为准。 |
| context.history | 是 | 上周信号库的 dedup_key，用于判断是否为**新**能力变化（旧闻重发不入候选）。 |

**三类能力子项清单**（判定命中用，来自主文档「四、国内模型与 AI 平台观察名单」）：

| 能力类别 | 键 | 子项清单 |
|---|---|---|
| 文档与视觉 | document_vision | 教材解析、试卷解析、图表识别、数学公式识别、手写内容识别、长文档理解、页面布局理解、图片高清化与重绘 |
| Agent 与工作流 | agent_workflow | Tool Use、Function Calling、MCP、Skill、多 Agent、长时间任务、浏览器/计算机操作、定时任务、工作流状态保存、人工审批 |
| 生产应用 | production | API 价格、缓存价格、上下文长度、结构化 JSON 输出、并发与限流、数据合规、私有化部署、服务稳定性、模型版本弃用 |

## 三、输出 schema

输出为 **A 分册信号 JSON（字段总表见 A 信号数据手册 §2.1）**，`report_status = "candidate"`。输出按模型分组包装，**没有新变化的模型输出「无新增」记录，而非空数组**，便于 D5 落盘与核验（与 monitor-education-companies 的 results 包装结构一致，`company` 字段换成 `model`）。

**结果结构**：

```json
{
  "skill": "monitor-china-models",
  "generated_at": "2026-08-06T10:00:00+08:00",
  "week": "2026-W33",
  "time_window": {"start": "2026-08-03", "end": "2026-08-07"},
  "results": [
    {
      "model": "Kimi",
      "status": "has_change",
      "signals": [ { "…候选信号 JSON,字段见下…" } ]
    },
    {
      "model": "DeepSeek",
      "status": "no_change",
      "signals": [],
      "no_change_note": "本周未发现与教育相关的模型能力实质新变化;已完成官方公告/技术文档/API 更新页检查(2026-08-03—2026-08-07)"
    }
  ]
}
```

各字段的填写责任标注如下（单条候选信号字段，即 `results[].signals[]` 内元素）：

| 字段 | 采集阶段状态 | 说明 |
|---|---|---|
| signal_id | 初值占位 `SIG-YYYY-WW-序号`（如 SIG-2026-W33-0001） | 周内递增、跨周不复用；正式分配由 D5 入库统一校验重排，采集阶段给初值占位 |
| company / brand / product | 本 Skill 填写 | 模型归属公司/品牌/具体产品（开放平台、应用或 SDK） |
| title | 本 Skill 填写 | ≤40 字 |
| summary | 本 Skill 填写 | ≤80 字 |
| published_at / event_at / week | 本 Skill 填写 | 来源对外发布日期 / 事件实际发生时间 / ISO 周号 |
| source_url | 本 Skill 填写 | array\<string\> ≥1，全部来源 URL |
| source_level | 本 Skill 填写 | S/A/B/C（见主文档「信息源分级」）；核心事实须有 S 级 |
| source_type | 本 Skill 填写 | 如 official_product_update / official_news / media_report |
| education_scenario | 本 Skill 填写 | 该能力**可能进入的教育场景**（个性化学习 / 作业批改 / 课堂授课 / 教研备课 / 学情诊断 / 教务运营 等） |
| workflow_stage | 本 Skill 填写 | 被 AI 介入的业务流阶段（对齐主文档「企业 AI 工作流拆解模板」第 3 步） |
| ai_capabilities | 本 Skill 填写 | 固定枚举子集：OCR / 多模态理解 / 知识库 / 推荐 / Agent / 内容生成 / 诊断 / 规划 |
| new_change | 本 Skill 填写 | **本次新变化**：相对上次观察具体改变了什么；去重键组成部分，不可为空 |
| target_users | 本 Skill 填写 | 教师 / 学生 / 家长 / 教务运营 / 学校（该能力最终服务对象） |
| evidence | 本 Skill 填写 | 核心事实证据 `[{fact, source_level, source_url}]`；S 级核心事实 ≥1 条 |
| limitations | 本 Skill 填写 | 已知局限与不确定性 `[{aspect, note}]`；不可证伪点必须记录 |
| verification | null（下游填写） | 由 `verify-source-evidence`（D4）回源核查后写入 |
| scoring | null（下游填写） | 由 `score-taoli-relevance` 按 E 锚点初评、人工终审修正后写入 |
| report_status | "candidate" | 采集入库默认状态 |
| dedup_key | 本 Skill 填写 | 构造规则见下；采集时生成，同键视为同一事件 |
| reviewer_feedback | 空 | 人工终审落点 |
| special_override | null | 人工强制进入标记 |
| recommended_action | 空 | 保留字段（取值语义归 C 分册桃李建议） |

**dedup_key 构造**（A 信号数据手册 §5）：

```
dedup_key = norm(company) + "|" + norm(brand) + "|" + norm(product)
          + "|" + norm(education_scenario) + "|" + norm(new_change)
```

`norm()`：全小写、去空白与全角标点；别名归并（如 科大讯飞=讯飞、Kimi=月之暗面），别名表以主文档观察名单为准。

## 四、执行步骤

D 分册 D3「模型更新采集」执行流程：

1. **读输入**：确认 `model_list`、`capability_scope`、`time_window`，加载 `context.history`（上周 dedup_key）与 `context.taoli_business`。
2. **逐家检索官方源**（S 级优先，按主文档「信息源分级」）：
   - 官方发布公告 / 更新日志 / changelog；
   - 官方技术文档、API 文档更新页、API 日志；
   - 官方产品页、模型卡、技术报告；
   - 官方 GitHub Release；
   - 官方公众号、正式发布会文字/视频实录。
3. **对照能力子项判定命中**：把每条更新映射到三类能力子项（见「输入 schema」子项清单）。命中才进入下一步；未命中的模型按第三节「结果结构」输出 `status="no_change"` 的「无新增」记录（`signals=[]` + `no_change_note`），而非留空或跳过。
4. **应用过滤规则（先过滤后记录）**——**以下类型不进候选**：
   - 纯参数/榜单：参数量、榜单名次、评测分数、跑分对比（即使热度高）；
   - 发布会口号、普通品牌宣传、企业形象稿；
   - 旧闻重发：与 `context.history` 中 dedup_key 相同或等价的能力变化（用 A 分册历史库前后对比判定）；
   - 与教育无关的通用能力噪声（仅「助力教育」口号的通用更新）。
5. **判定教育相关性**：该能力变化**是否可能改变教育业务**——映射到具体教育场景，回答主文档 6 问：进入哪个教育场景、改变原流程哪一步、是否形成数据闭环、是否提升教学效果或教师效率、桃李可否借鉴/验证/接入。无法映射到教育场景的能力变化不记录。
6. **只记能力变化，不止记发布**：`new_change` 写「相对上次观察改变了什么」（如：工作流新增人工审批节点、文档解析新增公式识别、开放平台新增私有化部署档），而不是「发布了新版本」。
7. **回源与分级**：多篇 B 级媒体重复报道的线索 → 聚合为一个事件，并回到官方原始页（S/A 级）确认事实；`source_level` 取该事件最高级别，`source_url` 收集全部来源（≥1）。
8. **构造 dedup_key**：按「输出 schema」规则生成，与 `context.history` 对照确认是新变化。
9. **产出候选信号 JSON**：按「输出 schema」完整填写；`report_status="candidate"`、`verification`/`scoring` 置 null。
10. **交付**：将候选 JSON 交 D5「保存候选」追加写入 `signals/signals-YYYY-WW.jsonl`（写入、备份、去重、分类、初评由 D5 及下游 Skill 负责，本 Skill 不越界）。

## 五、写作规范

- **教育视角写作**：title/summary 不是「某模型发布了某能力」，而是「该能力变化对教育业务意味着什么」。summary 可直接进入 C 分册「技术层条目」的「对教育意味着什么」候选素材。
- **字段约束**：title ≤40 字、summary ≤80 字、source_url 数组 ≥1、核心事实 ≥1 个 S 级来源；`source_level` 按主文档「信息源分级」判定。
- **三类能力标签**：产出时在 `ai_capabilities` 用固定枚举（OCR / 多模态理解 / 知识库 / 推荐 / Agent / 内容生成 / 诊断 / 规划），不要自造枚举值；能力子项细节写进 `new_change` 与 `evidence`。
- **保守记录**：官方文档中能力标注为「灰度/内测/仅白名单」的，必须写入 `limitations`；没有官方 S 级来源的能力变化，不作为核心事实写入 `evidence`，只能作为 B/C 级线索移交 `verify-source-evidence`。
- **检索工具降级兜底**：WebSearch/WebFetch 不可用时，在结果结构顶层加 `"fallback": "检索工具不可用,以下为人工采集标注"`，并在每条信号 `limitations` 注明「来源未经机器核验」；未经机器核验的信息不得标为 S 级核心事实。
- **区分厂商自述与事实**：S 级来源仍属企业自述，`summary` 与 `evidence.fact` 措辞用「官方发布/官方文档显示」，不用「实测证明」。
- **不写营销语言**：不用「震撼发布」「重大突破」等；能力变化描述使用可验证的操作性语言。
- **红线**：本 Skill 指令中不出现自动发布、自动冻结、自动去重行为；所有流转均按 A/C/D/E 分册的既定责任归属执行。

## 六、示例

> **构造示例，非本期真实数据**。仅演示字段如何按上文规范填写，不代表任何真实产品功能。

**输入**：`model_list=["Kimi"]`、`capability_scope=["agent_workflow"]`、`time_window={start:"2026-08-03", end:"2026-08-07", week:"2026-W33"}`；`context.taoli_business` 含「桃李教务运营有批量信息整理与内容生成任务」；`context.history` 中无同键记录。

**采集过程简述**：检索 Kimi 官方开放平台文档与官方公众号更新日志 → 发现工作流新增「人工审批」节点（命中 agent_workflow 子项「人工审批」）→ 教育相关性：可支撑教务运营批量任务（信息收集→内容生成→人确认）的自动化，改变「人全程手动」流程 → 非榜单/非口号 → 判定为新能力变化 → 构造 dedup_key。

**输出候选信号 JSON**（单条信号，位于结果结构 `results[0]`，`model="Kimi"`、`status="has_change"`、`signals[]` 内元素）：

```json
{
  "_schema": "2026-08-v1",
  "signal_id": "SIG-2026-W33-0001",
  "company": "月之暗面",
  "brand": "Kimi",
  "product": "Kimi 开放平台",
  "title": "Kimi 工作流新增人工审批节点",
  "summary": "Kimi 开放平台 Agent 工作流新增人工审批节点,长任务可在关键步骤暂停、等待人工确认后再继续,降低自动化任务失控风险。",
  "published_at": "2026-08-05",
  "event_at": "2026-08-05",
  "week": "2026-W33",
  "source_url": [
    "https://platform.moonshot.cn/docs/xxx",
    "https://mp.weixin.qq.com/s/xxx"
  ],
  "source_level": "S",
  "source_type": "official_product_update",
  "education_scenario": "教务运营-批量任务自动化",
  "workflow_stage": ["信息收集", "内容生成", "人工审批"],
  "ai_capabilities": ["Agent", "内容生成"],
  "new_change": "Agent 工作流新增人工审批节点,支持长任务中途暂停等人确认",
  "target_users": ["教务运营", "教师"],
  "evidence": [
    {"fact": "官方开放平台文档上线人工审批节点说明", "source_level": "S", "source_url": "https://platform.moonshot.cn/docs/xxx"},
    {"fact": "官方公众号发布工作流能力更新", "source_level": "S", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "verification": null,
  "limitations": [
    {"aspect": "未披露审批节点的并发与配额", "note": "生产规模可用性待实测"},
    {"aspect": "教育场景暂无官方落地案例", "note": "教育用途为基于桃李业务的推断"}
  ],
  "scoring": null,
  "report_status": "candidate",
  "dedup_key": "yuezhianmian|kimi|kimi_open_platform|jiaowuyunying-piliangrenwuzidonghua|agent_gongzuoliu_xinzeng_renshenpijiedian",
  "reviewer_feedback": "",
  "special_override": null,
  "recommended_action": ""
}
```

**后续流转**：该 JSON 交 D5 保存（入库统一校验重排 `signal_id`）→ `verify-source-evidence` 回源核查 → `classify-education-signal` 分类 → `score-taoli-relevance` 初评；是否进正文、是否发布由人工按 D 执行手册节点 1/3/4 决定，本 Skill 不参与。
