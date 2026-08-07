# A 信号数据手册(MVP)

> **文档版本**:v1.0.0
> **修订日期**:2026-08-06
> **修订人**:Jason8345
> **变更**:承接 v0.2 原稿第七节「每一条企业信号的数据结构」升级为独立分册:source_url 数组化(≥1);新增 title / summary / week / dedup_key / scoring / special_override 字段;report_status 扩展为五态;引入 JSONL 历史信号库与去重键规则。
> **待 v0.4 深写**:六态状态机、merge/superseded、跨周对比成章(当前仅为 MVP 精简版,不深写完整版)。

---

## 修订记录

| 版本 | 修订日期 | 修订人 | 变更范围 | 修订原因 | 影响分册 | 回退版本 |
|---|---|---|---|---|---|---|
| v1.0.0 | 2026-08-06 | Jason8345 | 初版(承接 v0.2 第七节) | 主文档 v0.3 重构为「主文档 + 分册」体系 | — | — |
| 待 v0.4 完整登记 | 2026-08-06 | Jason8345 | 新增 classify-education-signal 输出的分类扩展字段:signal_category / signal_category_reason / monitor_bucket / education_scenario_reason(追加信号 JSON 顶层,紧邻 education_scenario 之后) | 与 classify-education-signal 输出契约对齐;schema 字段增删属「必须 bump」 | A(本册) + skills/registry.md | v1.0.0 |

> 后续修订登记遵循主文档「方法论版本管理」六步流程;schema 字段增删属「必须 bump」。

---

## 一、手册定位与数据流

**本册一句话定位**:定义信号 JSON schema 与精简状态,是采集、去重、评分、生成全流程的**唯一数据底座**——所有 Skill 的输入输出契约都以本册 schema 为准。

数据流(全流程唯一数据底座的含义):

1. **采集**:monitor-education-companies、monitor-china-models 等采集 Skill 产出原始事件,按本册 schema 结构化。
2. **入库**:追加写入本周 JSONL(状态 candidate),生成 signal_id 与 dedup_key。
3. **去重与初评**:classify-education-signal(事件分类)、score-taoli-relevance(评分,按 E 评分锚点手册)、verify-source-evidence(来源核验)、check-source-quality(来源质量检查);状态流转为 selected / rejected。
4. **生成**:write-company-analysis、write-taoli-actions、format-all-staff-newsletter 只读取 selected 信号。
5. **发布**:归属 **D 执行手册**节点 4 **人工操作**——AI 不自动发布,只提交 frozen 结果。

读者与接口:

- **方法论维护者**:schema 演进、字段增删、状态扩展,走主文档「方法论版本管理」修订流程。
- **单人+AI 协作执行者**:本册即采集 Skill 的输出契约、去重键构造手册、历史库归档手册。
- 评分权重与分级 → 主文档「进入周报的判断规则」;六维打标尺与对比例 → **E 评分锚点手册**。
- S-A-B-C 分级定义与使用规则 → 主文档「信息源分级」。
- 采集/去重/发布的具体执行步骤 → **D 执行手册**(D5 去重步骤、节点 4 人工发布)。
- 桃李建议(recommended_action 的语义取值) → **C 周报输出规范**。
- 9 个核心 Skill 精简注册表 → skills/registry.md。

---

## 二、信号 JSON 完整 Schema

### 2.1 字段总表

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| signal_id | string | 是 | 唯一信号 ID,格式 `SIG-YYYY-WW-序号`(如 SIG-2026-W33-0001);周内递增、跨周不复用 |
| company | string | 是 | 母集团/公司名(如 好未来);别名统一见主文档观察名单 |
| brand | string | 是 | 品牌名(如 学而思);无独立品牌时取 company |
| product | string | 是 | 具体产品名;无法定位到产品时填公司级业务线 |
| title | string | 是 | 信号标题(≤40 字),简报与去重展示用 |
| summary | string | 是 | 一句话摘要(≤80 字),可直接进入「信息速览」区 |
| published_at | string | 是 | 信息来源对外发布日期(YYYY-MM-DD) |
| event_at | string | 是 | 事件实际发生时间;与 published_at 不一致时如实填写 |
| week | string | 是 | 归属周号(ISO 周,如 2026-W33);决定写入哪个 JSONL 文件 |
| source_url | array\<string\> | 是(≥1) | 全部来源 URL,至少 1 条;多条为不同来源(区别于 v0.2 单字符串) |
| source_level | enum(S/A/B/C) | 是 | 该信号最高来源级别;核心事实须有 S 级(见主文档「信息源分级」) |
| source_type | string | 是 | 来源类型:official_product_update / policy / official_news / media_report / user_review / case_study 等 |
| education_scenario | string | 是 | AI 进入的教育场景(个性化学习 / 作业批改 / 课堂授课 / 教研备课 / 学情诊断等) |
| signal_category | enum | 是 | 分类标签:教育企业 / 大模型 / AI工具 / 政策 / 海外 / 待人工;由 classify-education-signal 产出(取值见该 SKILL §四 步骤 2) |
| signal_category_reason | string | 否 | 类型判定理由(≤80 字),引用信号字段或证据 |
| monitor_bucket | enum | 是 | 监测比例桶:教育企业及教育科技产品(约50%)/ 国内大模型、AI工具与技术平台(约25%)/ 教育政策、学校及区域落地案例(约15%)/ 海外重要参考(约10%)/ 待定(待人工);由 classify-education-signal 按 signal_category 映射 |
| education_scenario_reason | string | 否 | 场景判定理由(≤80 字),引用 workflow_stage / summary / evidence |
| workflow_stage | array\<string\> | 是 | 被 AI 介入的业务流阶段(对齐主文档「企业 AI 工作流拆解模板」第 3 步) |
| ai_capabilities | array\<string\> | 是 | AI 能力清单(OCR / 多模态理解 / 知识库 / 推荐 / Agent / 内容生成 / 诊断 / 规划) |
| new_change | string | 是 | 本次新变化:相对上一次观察具体改变了什么(去重键组成部分,不可为空) |
| target_users | array\<string\> | 是 | 服务对象:教师 / 学生 / 家长 / 教务运营 / 学校 |
| evidence | array\<object\> | 是 | 核心事实证据;每项 `{fact, source_level, source_url}`(见 2.2) |
| verification | object\|null | 否 | verify-source-evidence 的来源核查结论(见 2.2);null 表示尚未核查或无需核查 |
| limitations | array\<object\> | 否 | 已知局限与不确定性;每项 `{aspect, note}`;不可证伪点必须记录 |
| scoring | object | 是 | 六维评分 + 总分 + 等级 + 理由(见 2.2) |
| report_status | enum | 是 | candidate / selected / rejected / frozen / published(见第 3 节) |
| dedup_key | string | 是 | 去重键(见第 5 节);采集入池时生成,同键视为同一事件 |
| reviewer_feedback | string | 否 | 人工审核意见;终审修正评分、状态或 special_override 的落点 |
| special_override | object\|null | 否 | 人工强制进入标记;null 表示未强制(见 2.2) |
| recommended_action | string | 否 | **保留字段**:结构兼容 v0.2;取值语义已迁移至 **C 分册「桃李建议」**(由 write-taoli-actions 产出),本册不再维护其取值规范 |

### 2.2 嵌套对象约定

**evidence 项**:`{"fact": "<陈述>", "source_level": "S|A|B|C", "source_url": "<该证据的来源 URL>"}`;S 级核心事实至少 1 条。

**verification 项**:`{"source_level": "S|A|B|C", "checked_url": "<回源核查的原始页面 URL>", "conclusion": "<已核实/存疑/无法核实>", "note": "<核查备注>"}`;由 verify-source-evidence 在 D 分册 D4 回源核查时写入。

**scoring(六维,权重对齐主文档「进入周报的判断规则」)**:

| 维度 | 权重 | 评分键 | 说明 |
|---|---|---|---|
| 与教育业务的关系 | 25 | education_relevance | AI 进入教育场景的实质程度 |
| 与桃李未来的关系 | 25 | taoli_relevance | 桃李可借鉴/验证/接入程度 |
| 工作流创新程度 | 15 | workflow_novelty | 对原流程的改变与闭环程度 |
| 可实际验证程度 | 15 | verifiability | 能否用 S/A 级来源验证 |
| 信息可靠度 | 10 | credibility | 由 source_level 支撑 |
| 行业影响力 | 10 | industry_impact | 行业影响面 |

```json
"scoring": {
  "education_relevance": {"tier": "高", "score": 20, "reason": "直接进入写作教学场景"},
  "taoli_relevance":     {"tier": "高", "score": 18, "reason": "桃李可复用批改+诊断工作流"},
  "workflow_novelty":    {"tier": "高", "score": 12, "reason": "批改结果回流学情报告形成闭环"},
  "verifiability":       {"tier": "中", "score": 12, "reason": "有官方发布页可验证"},
  "credibility":         {"tier": "高", "score": 9,  "reason": "S 级官方来源"},
  "industry_impact":     {"tier": "中", "score": 8,  "reason": "头部企业功能更新"},
  "total": 79,
  "level": "重点关注",
  "summary": "S 级来源、功能真实可验证,建议重点关注"
}
```

等级映射(主文档「进入周报的判断规则」):80-100 本周必看;65-79 重点关注;50-64 信息速览;<50 不进入周报。初评由 AI 按 E 评分锚点手册完成,终审由人工修正并冻结(见 D 执行手册)。

**special_override**:

```json
"special_override": null
// 人工强制进入时:
"special_override": {"reason": "重大政策/直接竞品变化", "decided_by": "<人工>", "decided_at": "2026-08-06"}
```

总分低于 50 时,重大政策与直接竞品变化可经人工强制进入;标记保持历史一致,不得事后补记。

### 2.3 完整示例 JSON

```json
{
  "_schema": "2026-08-v1",
  "signal_id": "SIG-2026-W33-0001",
  "company": "好未来",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "title": "小思AI 1对1 新增作文批改 Agent 工作流",
  "summary": "学而思小思AI 1对1新增AI作文批改,支持逐句诊断与改写建议,并写入学情报告。",
  "published_at": "2026-08-03",
  "event_at": "2026-08-01",
  "week": "2026-W33",
  "source_url": [
    "https://www.100tal.com/news/xxx",
    "https://mp.weixin.qq.com/s/xxx"
  ],
  "source_level": "S",
  "source_type": "official_product_update",
  "education_scenario": "个性化学习-写作批改",
  "signal_category": "教育企业",
  "signal_category_reason": "主体为国内教育企业好未来旗下学而思,new_change 为教育产品功能落地",
  "monitor_bucket": "教育企业及教育科技产品(约50%)",
  "education_scenario_reason": "workflow_stage=作业批改→错因诊断→补弱,AI 进入写作练习闭环",
  "workflow_stage": ["作业批改", "错因诊断", "补弱"],
  "ai_capabilities": ["多模态理解", "作文诊断", "改写生成"],
  "new_change": "新增作文批改Agent:逐句诊断到生成改写建议并回流学情报告",
  "target_users": ["学生", "家长"],
  "evidence": [
    {"fact": "官方发布页于2026-08-01上线", "source_level": "S", "source_url": "https://www.100tal.com/news/xxx"},
    {"fact": "教师端可审核AI批改结果", "source_level": "A", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "verification": null,
  "limitations": [
    {"aspect": "未披露批改准确率数据", "note": "仅能验证功能存在,无法验证效果"},
    {"aspect": "仅支持语文学科", "note": "跨学科泛化能力未知"}
  ],
  "scoring": {
    "education_relevance": {"tier": "高", "score": 20, "reason": "直接进入写作教学场景"},
    "taoli_relevance":     {"tier": "高", "score": 18, "reason": "桃李可复用批改+诊断工作流"},
    "workflow_novelty":    {"tier": "高", "score": 12, "reason": "批改结果回流学情报告形成闭环"},
    "verifiability":       {"tier": "中", "score": 12, "reason": "有官方发布页可验证"},
    "credibility":         {"tier": "高", "score": 9,  "reason": "S 级官方来源"},
    "industry_impact":     {"tier": "中", "score": 8,  "reason": "头部企业功能更新"},
    "total": 79,
    "level": "重点关注",
    "summary": "S 级来源、功能真实可验证,建议重点关注"
  },
  "report_status": "selected",
  "dedup_key": "haolaixueershi|xueersi|xiaosiai1dui1|gexinghuaxuexi-xiezuopiyue|xin_zeng_zuowen_piyue_agent",
  "reviewer_feedback": "",
  "special_override": null,
  "recommended_action": "【语义归 C 分册桃李建议】建议在桃李语文批改场景试点 Agent 工作流"
}
```

### 2.4 与 v0.2 JSON 的差异对照表

| 变更类型 | v0.2 字段 | v1.0.0(本册) | 说明 |
|---|---|---|---|
| 类型变更 | source_url: string(单字符串) | source_url: array\<string\>(≥1) | 多来源并存,支撑跨来源聚合与「同事件保留最高 source_level 行」 |
| 收敛 | taoli_relevance / novelty / credibility / actionability(4 个 0-5 标量) | 删除,收敛为 scoring 六维对象 | 六维权重对齐主文档总权重表 |
| 新增 | — | title / summary | 简报与「信息速览」区的直接数据源 |
| 新增 | — | week | ISO 周号,决定 JSONL 归档文件归属 |
| 新增 | — | dedup_key | 去重键(见第 5 节) |
| 新增 | — | scoring(对象) | 六维 + 总分 + 等级 + 理由 |
| 新增 | — | special_override | 人工强制进入标记 |
| 新增 | — | verification | verify-source-evidence 的来源核查结论(与 D 分册 D4 回源核查对齐) |
| 扩展 | report_status: 仅 "candidate" | report_status: candidate/selected/rejected/frozen/published | 对应第 3 节精简状态流转 |
| 语义迁移 | recommended_action: 字段 | recommended_action: 保留字段 | 取值语义迁至 C 分册「桃李建议」 |

---

## 三、精简状态流转

```
candidate(采集入库)
   │ 初评+去重(D 分册 D5)
   ├─→ selected ──→ frozen(周五冻结) ──→ published(发布归档)
   └─→ rejected(不入正文,仍留历史库)
```

| 状态 | 含义 | 进入条件 |
|---|---|---|
| candidate | 采集 Skill 首次写入,未去重未评分 | 入库默认状态 |
| selected | 通过初评去重,进入生成池 | D5 去重后判定可入正文 |
| rejected | 被过滤(总分 <50,或重复/无实质变化) | 初评去重后判定不入正文 |
| frozen | 本期信号冻结,不再变动 | 周五冻结本期候选池(见主文档「抓取节奏」) |
| published | 已发布归档 | **D 执行手册节点 4 人工操作** |

规则:

- **单向流动**;不得从 frozen 回退 candidate(特殊豁免走主文档「方法论版本管理」)。
- 冻结时,本期每条信号必须已落定 selected 或 rejected,不允许滞留 candidate。
- rejected 信号仍保留在历史 JSONL 中(保留弱信号线索),只是不入正文。
- **发布必须由人工执行**(D 执行手册节点 4):AI 只提交 frozen 结果,实际对外发布与全员触达由人完成。
- 六态状态机、merge / superseded 等复杂流转 → **待 v0.4 深写**,MVP 不引入。

---

## 四、轻量 JSONL 历史信号库

| 项 | 约定 |
|---|---|
| 路径 | `weekly-report/signals/signals-YYYY-WW.jsonl`(YYYY 年,WW 两位 ISO 周号) |
| 首行 schema 元信息 | `{"_schema":"2026-08-v1"}`(schema 变更须同步 bump 并登记修订记录表) |
| 写入方式 | **追加写入**;新信号 append 到本周文件,不重写整文件 |
| 冻结 | 周五生成**本周终版**(本期全部信号落定 report_status) |
| 历史保护 | **不修改历史周文件**;需修正时按修订记录另存,而非覆盖 |
| 备份 | 每次写入前 `cp signals-YYYY-WW.jsonl signals-YYYY-WW.jsonl.bak` |
| 二进制证据 | 图片/视频等不进 JSONL 正文,以**相对路径**引用 `weekly-report/signals/signal-assets/<week>/<file>` |

约定:

- 每条信号一行合法 JSON;按 signal_id 顺序追加。
- 一个事件合并后只保留**一行**(跨来源 URL 并入 source_url[]),见第 5 节。
- 归档后的 published 文件复制到 `weekly-report/published/`(归档动作见 D 执行手册)。

---

## 五、去重规则(内嵌;执行归属 D 分册 D5)

本册只定义 dedup_key 构造与合并优先级;**去重步骤执行在 D 分册 D5**。

**dedup_key 构造**:

```
dedup_key = norm(company) + "|" + norm(brand) + "|" + norm(product)
          + "|" + norm(education_scenario) + "|" + norm(new_change)
```

**规范化 norm()**:

- 全小写、去首尾与内部空白、去全角标点;
- 公司/品牌别名归并:好未来 = 学而思 = TAL;科大讯飞 = 讯飞;小猿 = 猿辅导等(别名表随观察名单演进,登记于修订记录表)。

**合并优先级(同 dedup_key 命中)**:

1. 保留 **source_level 最高**的一行作为主行;
2. 其余行来源 URL 并入主行 source_url[](**跨来源聚合**);
3. new_change 取最具体的一版,evidence 取并集;
4. 状态以主行为准进入后续流转。

**过滤联动**:

- new_change 参与去重键,可过滤「标题党但无实质变化」的重复新闻(呼应主文档「进入周报的判断规则」:高热度 ≠ 高分);
- 多篇 B 级媒体重复报道 → 聚合为一个事件,并回到 S/A 原始来源核查(verify-source-evidence);
- 采集与去重阶段均以 check-source-quality 把关来源质量。

---

## 附:本周文件写入检查单(MVP)

1. 首行已含 `{"_schema":"2026-08-v1"}`;
2. 每条信号 source_url 数组 ≥1,source_level 已按主文档「信息源分级」判定;
3. dedup_key 已构造,同键事件已合并、主行保留最高 source_level;
4. 写入前已做 .bak 备份,追加写入,未触碰历史周文件;
5. 周五冻结时无滞留 candidate;发布动作由人工在 D 执行手册节点 4 完成。
