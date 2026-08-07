---
name: monitor-education-companies
description: "可配置企业监控采集 Skill:对一级观察名单企业(默认好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道)做本周信息检索,识别 AI 相关新变化,产出每家企业的候选信号 JSON。当需要采集企业信号、为周报候选池建档时使用。"
---

# monitor-education-companies — 可配置企业监控(采集类 Skill)

> 契约基准:信号字段以 **A 信号数据手册** 为准;流程归属以 **D 执行手册** D1 步骤为准;来源分级见主文档「信息源分级」。一家 Skill 覆盖全部一级名单,新增/删减企业只需改输入清单,无需新建 Skill。

## 一、用途

对一级观察名单企业做**本周信息采集**,识别每家企业在时间窗内与 AI 相关的新变化,并产出**候选信号 JSON**(report_status 一律 `candidate`,未去重、未评分)。

本 Skill 解决的问题:

- 每次采集不用为每家企业分别建监控任务,一个调用覆盖全部一级名单;
- 企业清单**可配置**:支持单家、多家、增删(如临时加看高途、ClassIn 等二级名单企业);
- 即使某家企业本周**没有**值得记录的 AI 新变化,也完成一次信息检查并输出「无新增」记录,而非留空或跳过(遵循 D 分册 D1「无新增时记录『无新增』而非留空」)。

**职责边界(红线)**:

- 本 Skill **只做采集与建档**;去重归 **D 执行手册 D5 步骤**(AI 读 dedup_key 做事件级去重,不单独立 Skill),本 Skill 仅构造 dedup_key **初值**;
- 评分归 **score-taoli-relevance**(按 E 评分锚点手册);本 Skill 不评分,`scoring` 留空待后续填充;
- 来源核查归 **verify-source-evidence** 与 **check-source-quality**;本 Skill 发现 B 级线索时如实标注 source_level=B,不回源、不擅自升级;
- **不冻结候选池、不置 published、不触发任何发布动作**(发布归 D 执行手册节点 4 **人工操作**,AI 不自动发布);
- **不自动过滤进正文**:凡实质新变化一律建档为候选,进不进正文由后续评分与人工冻结决定。

## 二、输入 schema

输入为一次「配置 + 时间窗 + 检索深度」的调用,JSON 形式(也可用自然语言指令携带,字段同上):

```json
{
  "companies": [
    {"name": "好未来", "aliases": ["学而思", "TAL"], "focus": ["学习机", "九章大模型", "小思AI 1对1", "智能教辅"]},
    {"name": "新东方", "aliases": ["新东方教育科技"], "focus": ["AI 1对1", "AI 助教", "语言学习", "教学内容生产"]}
  ],
  "time_window": {"start": "2026-08-03", "end": "2026-08-07"},
  "search_depth": "medium",
  "notes": "可选:本期附加关注点(如竞品价格、特定产品线)"
}
```

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| companies | array\<object\> | 否(有默认) | 企业清单。**不传则用默认一级名单 7 家**(好未来／新东方／猿辅导／作业帮／科大讯飞教育／希沃／网易有道);每项 `{name, aliases[], focus[]}`,focus 参考主文档「一级观察名单」的「重点产品与能力」 |
| time_window | object | 否(有默认) | 检索时间窗 `{start, end}`,ISO 日期 YYYY-MM-DD;**默认取本周一至当前日期** |
| search_depth | enum(light/medium/deep) | 否 | 检索深度,默认 `medium`(定义见「四、执行步骤」) |
| notes | string | 否 | 本期附加关注点,可注入到每家企业的检索指令中 |

**默认一级名单 7 家(companies 省略时)**:

| name | aliases | focus(参考) |
|---|---|---|
| 好未来 | 学而思、TAL | 学而思学习机、九章大模型、小思 AI 1对1、九章爱学、智能教辅、AI 课堂、教师备课、学情分析 |
| 新东方 | 新东方教育科技 | AI 1对1、AI 教师/助教、语言学习、教学内容生产、教师培训、学情诊断、国际教育 |
| 猿辅导 | 猿力科技 | 小猿 AI、小猿学习机、小猿口算、海豚 AI 学、斑马 AI 学、猿编程、AI 大阅读 |
| 作业帮 | — | 作业帮学习机、AI 超级老师、AI 精准练、AI 通关练、拍照搜题、自动批改、多模态解题 |
| 科大讯飞教育 | 讯飞、科大讯飞智慧教育 | 星火教育大模型、星火教师助手、AI 黑板、智能批阅机、AI 学习机、智慧考试、区域学情 |
| 希沃 | 视源股份、Seewo | 希沃教学大模型、AI 备课、AI 授课、AI 评价、AI 教研、课堂智能反馈、智慧教室 |
| 网易有道 | 有道 | 有道智慧教育、有道词典笔、AI 学习硬件、AI 文档知识库、语言学习 |

> 增删规则:传入 companies 即覆盖默认名单;增看企业照 `{name, aliases, focus}` 追加即可,无需改 Skill。

## 三、输出 schema

输出为**每家企业的候选信号 JSON 数组**。按企业分组返回:每家企业一个结果对象,含该企业 0..n 条候选信号;**没有新变化的返回「无新增」记录,而非空数组**。

**结果结构**:

```json
{
  "skill": "monitor-education-companies",
  "generated_at": "2026-08-06T10:00:00+08:00",
  "week": "2026-W32",
  "time_window": {"start": "2026-08-03", "end": "2026-08-07"},
  "results": [
    {
      "company": "好未来",
      "status": "has_change",
      "signals": [ { "…候选信号 JSON,见下…" } ]
    },
    {
      "company": "新东方",
      "status": "no_change",
      "signals": [],
      "no_change_note": "本周未发现与 AI 相关的实质新变化;已完成官网/公众号/财报/行业媒体检查(2026-08-03—2026-08-07)"
    }
  ]
}
```

**候选信号 JSON(每条,字段与 A 分册信号 JSON 对齐)**:

| 字段 | 类型 | 采集阶段取值 | 说明 |
|---|---|---|---|
| signal_id | string | 初值 `SIG-YYYY-WW-序号`(如 SIG-2026-W32-0003) | 周内递增、跨周不复用;正式分配由 D5 入库时统一校验/重排,采集阶段给初值占位 |
| company | string | 母集团名 | 别名统一见主文档观察名单 |
| brand | string | 品牌名 | 无独立品牌时取 company |
| product | string | 具体产品名 | 无法定位到产品时填公司级业务线 |
| title | string | ≤40 字 | 直陈"谁 + 做了什么 + 对谁有用",去重与简报用 |
| summary | string | ≤80 字 | 一句话摘要,可直接进「信息速览」区 |
| published_at | string | YYYY-MM-DD | 信息来源对外发布日期 |
| event_at | string | YYYY-MM-DD | 事件实际发生时间;与 published_at 不一致时如实填写 |
| week | string | `YYYY-WW`(ISO 周) | 决定写入哪个 signals JSONL 文件 |
| source_url | array\<string\> | ≥1 条 | 全部来源 URL;多条为不同来源 |
| source_level | enum(S/A/B/C) | 该信号**最高**来源级别 | 核心事实须有 S 级;B 级线索如实标注,不擅自升级 |
| source_type | string | 如 official_product_update / official_news / media_report / financial_report | 见 A 分册示例取值 |
| education_scenario | string | 如 个性化学习 / 作业批改 / 课堂授课 / 教研备课 / 学情诊断 | AI 进入的教育场景 |
| workflow_stage | array\<string\> | 被 AI 介入的业务流阶段 | 对齐主文档「企业 AI 工作流拆解模板」第 3 步 |
| ai_capabilities | array\<string\> | OCR / 多模态理解 / 知识库 / 推荐 / Agent / 内容生成 / 诊断 / 规划 | 按实际能力勾选 |
| new_change | string | **非空** | 本次新变化:相对上一次观察具体改变了什么(去重键组成部分) |
| target_users | array\<string\> | 教师 / 学生 / 家长 / 教务运营 / 学校 | 服务对象 |
| evidence | array\<object\> | 每项 `{fact, source_level, source_url}` | 核心事实证据,S 级核心事实至少 1 条 |
| dedup_key | string | 初值,构造规则见「五、写作规范」 | 采集入池时生成,同键视为同一事件;去重执行在 D5 |
| report_status | enum | **固定 `candidate`** | 未去重未评分 |
| verification | object\|null | `null` | 由 verify-source-evidence 填充,本 Skill 不填 |
| limitations | array\<object\> | 可选 | 采集阶段已知局限(如"未披露准确率""仅支持某学科"),不强制 |
| scoring | object\|null | `null` | 由 score-taoli-relevance 按 E 锚点填充,本 Skill 不评分 |
| reviewer_feedback | string | 空 | 人工审核意见落点 |
| special_override | object\|null | `null` | 人工强制进入标记,本 Skill 不填 |
| recommended_action | string | 空 | 语义归 C 分册桃李建议,由 write-taoli-actions 产出 |

> **输出去向**:候选信号由 D 分册 D5 保存时追加写入 `signals/signals-YYYY-WW.jsonl`(首行 `{"_schema":"2026-08-v1"}`,写入前 .bak 备份)。本 Skill 只产出 JSON,不负责落盘;落盘时机归 D5。

## 四、执行步骤

对清单中**每家**企业执行以下步骤。一家企业的检索完成后再进入下一家。

### 步骤 1 组装检索策略

按企业 focus 与 time_window 组装检索。检索入口(按 S→A→B→C 优先级):

- **S 级**:企业官网新闻中心、官方产品页、官方公众号/公告、发布会实录、上市公司财报与公告、官方技术文档/模型卡;
- **A 级**:学校/教育局应用案例、招投标、客户案例、大会演讲、App 更新记录、专利论文、招聘信息;
- **B 级**:芥末堆、多知、36 氪、机器之心、量子位、新智元、甲子光年、雷峰网、教育行业公众号;
- **C 级**:小红书、知乎、B 站、抖音等**只作弱信号**,不进入候选(除非配合 S/A/B 来源佐证)。

检索示例(每家):`<企业名> + AI + <时间窗内日期>`;对 focus 中的重点产品追加检索(如「学而思 学习机 AI」「讯飞 星火 教育」)。

**检索深度定义**:

| 深度 | 每家企业检索轮数 | 覆盖范围 | 产出规模 |
|---|---|---|---|
| light | 1-2 轮 | 官网新闻 + 最新行业媒体 | 只记最显著 1-2 条 |
| medium(默认) | 2-4 轮 | 官网/官方公众号 + 财报/公告 + 行业媒体 | 全部实质新变化 |
| deep | 5+ 轮 | 全部来源 + 产品页逐个核对 + 必要时回源核查 | 全部新变化 + 细节证据 |

### 步骤 2 识别 AI 相关新变化

逐条判断检索结果是否构成「本次 AI 相关新变化」。识别时对照主文档「一级观察名单」各企业的**每周需要回答**(如好未来看"新增了什么 AI 功能、服务谁、是否调用学生数据、是否形成闭环")。**只记录 AI 进入教育场景/工作流的实质变化**。

**应过滤、不产出的类型**(呼应 E 分册「过滤规则」):融资新闻(除非明确投教育 AI 且改变格局)、发布会口号(无产品/技术/工作流/业务变化)、普通品牌宣传、旧闻重发。高热度 ≠ 高价值。

### 步骤 3 判断"相对上一次观察"的新变化

- `new_change` 必须回答「**这次和上次比,到底变了什么**」;仅重复报道历史功能不构成 new_change;
- 有上期信号库(读 `signals/signals-YYYY-WW.jsonl` 及历史周文件)时先对比去重,再定 new_change;
- 无法确认是"新变化"的,降级为「无新增」或记为弱线索并在 limitations 注明。

### 步骤 4 构造候选信号 JSON

按第三节 schema 输出每条候选信号。要点:

- 多来源报道同一事件 → **聚合为一条**,source_url 并入数组,source_level 取最高级别;
- 核心事实至少 1 个 S 级来源;只有 B 级线索的,标注 B 级并写"待 verify-source-evidence 回源核查"(不自行升级);
- evidence 每项必须可溯源到具体 URL;数字、日期、功能名必须来自来源原文。

### 步骤 5 无新变化处理

某家企业本周无实质新变化时,输出 `{"company": "...", "status": "no_change", "no_change_note": "…已完成检查,未发现 AI 相关实质新变化…"}`。

**不是空跳过**:必须完成一次信息检查(步骤 1 至少 light 深度),并在 no_change_note 中记录检查范围与时间窗。

### 步骤 6 汇总输出

合并各企业结果,输出第三节的完整结构。每条候选信号 `report_status="candidate"`,不做评分、不做去重、不冻结、不发布。

## 五、写作规范

### 5.1 标题与摘要

- **title ≤ 40 字**:直陈"谁 + 做了什么 + 对谁有用",不用修辞与感叹号。例:「学而思学习机上线错题归因重练闭环」。
- **summary ≤ 80 字**:一句话说清核心事实,可直接进「信息速览」区。例:「学习机自动归因错题、生成变式题并回测,把课后补弱做成闭环,教师端可审核。 」

### 5.2 字段语义规范

- **published_at / event_at / week**:published_at 取来源对外发布日期;event_at 取事件实际发生时间(发布会当天 vs 功能上线当天);week 取 ISO 周号 `YYYY-WW`。
- **source_level**:取该信号**最高**来源级别;核心事实必须 S 级;S/A/B/C 分级定义见主文档「信息源分级」。
- **source_type**:official_product_update / official_news / financial_report / media_report / case_study 等(参考 A 分册示例)。
- **education_scenario / workflow_stage / ai_capabilities**:三字段对齐主文档「企业 AI 工作流拆解模板」的「AI 介入点 / AI 能力」两列;workflow_stage 必须是流程阶段(备课/授课/练习/测评/批改/诊断/补弱…),不是能力名。
- **target_users**:教师 / 学生 / 家长 / 教务运营 / 学校,多选。
- **evidence[]**:每项 `{fact, source_level, source_url}`;fact 为可验证陈述,不写推测。

### 5.3 dedup_key 初值构造(去重执行归 D5)

```
dedup_key = norm(company) + "|" + norm(brand) + "|" + norm(product)
          + "|" + norm(education_scenario) + "|" + norm(new_change)
```

- `norm()`:全小写、去首尾与内部空白、去全角标点;
- **别名归并**:好未来 = 学而思 = TAL;科大讯飞 = 讯飞;小猿 = 猿辅导;希沃 = 视源股份/Seewo 等;
- 例:`haolaixueershi|xueersi|xiaosiai1dui1|gexinghuaxuexi-xiezuopiyue|xin_zeng_zuowen_piyue_agent`。

> 本 Skill 只构造 dedup_key **初值**;同键合并、跨来源聚合由 D 分册 D5 执行。

### 5.4 来源与证据纪律

- 每条候选信号 **source_url ≥ 1**;核心事实至少有 1 个 **S 级**来源;
- 多篇 B 级媒体重复报道 → 聚合为一条,回到 S/A 原始来源核查后再确认事实;
- **B 级不得作为唯一事实来源**写进候选(可作线索并标注待核查);
- 检索工具(WebSearch/WebFetch)不可用时的降级:在输出结构顶层加 `"fallback": "检索工具不可用,以下为人工采集标注"`,并在每条信号 limitations 注明来源未经机器核验。

### 5.5 红线(写死,不可违反)

- **不自动发布**:本 Skill 不触发生成 newsletter、不置 published、不冻结候选池;冻结与发布归 D 执行手册节点 1/4 **人工操作**;
- **不去重执行**:只构造 dedup_key 初值;事件级去重由 D5 步骤完成;
- **不评分**:scoring 留 null,评分由 score-taoli-relevance 按 E 锚点完成;
- **不外判进正文**:凡实质新变化一律建档为 candidate,不在此处"砍条数"。

## 六、示例

### 6.1 单条候选信号示例(构造,非本期真实数据)

```json
{
  "signal_id": "SIG-2026-W32-0003",
  "company": "好未来",
  "brand": "学而思",
  "product": "学而思学习机",
  "title": "学而思学习机上线错题归因重练闭环",
  "summary": "学习机自动归因错题、生成变式题并回测,把课后补弱做成闭环,教师端可审核。",
  "published_at": "2026-08-04",
  "event_at": "2026-08-02",
  "week": "2026-W32",
  "source_url": [
    "https://www.100tal.com/news/xxx",
    "https://mp.weixin.qq.com/s/xxx"
  ],
  "source_level": "S",
  "source_type": "official_product_update",
  "education_scenario": "个性化学习-课后补弱",
  "workflow_stage": ["作业批改", "错因诊断", "补弱"],
  "ai_capabilities": ["多模态理解", "诊断", "内容生成"],
  "new_change": "新增错题归因重练:AI判错后归因、生成变式题、学生重练并回测",
  "target_users": ["学生", "家长", "教师"],
  "evidence": [
    {"fact": "教师端可审核AI生成的变式题", "source_level": "S", "source_url": "https://www.100tal.com/news/xxx"},
    {"fact": "官方公众号于2026-08-02发布功能上线说明", "source_level": "A", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "dedup_key": "haolaixueershi|xueersi|xueersixuexiji|gexinghuaxuexi-kchoubuyuo|xin_zeng_cuoti_guanyin_zhonglian",
  "report_status": "candidate",
  "verification": null,
  "scoring": null,
  "reviewer_feedback": "",
  "special_override": null,
  "recommended_action": ""
}
```

### 6.2 结果结构示例(混合有变化/无变化,构造)

```json
{
  "skill": "monitor-education-companies",
  "generated_at": "2026-08-07T18:00:00+08:00",
  "week": "2026-W32",
  "time_window": {"start": "2026-08-03", "end": "2026-08-07"},
  "results": [
    {
      "company": "好未来",
      "status": "has_change",
      "signals": [ { "…6.1 示例…" } ]
    },
    {
      "company": "新东方",
      "status": "no_change",
      "signals": [],
      "no_change_note": "2026-08-03—2026-08-07 完成官网新闻/官方公众号/财报/行业媒体检查,未发现与 AI 相关的实质新变化"
    }
  ]
}
```

### 6.3 调用示例(自然语言)

> 「执行今日 D1 企业采集:monitor-education-companies,默认 7 家一级名单,时间窗 2026-08-03 至 2026-08-07,检索深度 medium。另加看高途。」

等价于输入:`companies` 传入默认 7 家 + `{"name":"高途","aliases":[],"focus":["AI 伴学","学习服务"]}`,`search_depth="medium"`,`time_window` 如上。

## 七、产物去向与衔接

- 候选信号 JSON → D 分册 **D5 保存候选 + 去重 + 分类 + 初评**:追加写入 `signals/signals-YYYY-WW.jsonl`(首行 `{"_schema":"2026-08-v1"}`,写入前 .bak 备份);
- 去重(读 dedup_key)由 **D5** 负责;分类由 **classify-education-signal** 负责;初评由 **score-taoli-relevance** 负责;来源核查由 **verify-source-evidence / check-source-quality** 负责;
- 冻结候选池 → D 执行手册 **节点 1(人)**;生成初稿 → 节点 2(write-company-analysis / write-taoli-actions / format-all-staff-newsletter);人工审核 → 节点 3(人);发布 → **节点 4(人,AI 不自动发布)**。
