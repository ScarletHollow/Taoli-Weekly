---
name: classify-education-signal
description: 在周报 D5 建档阶段,对候选信号做类型分类(教育企业/大模型/AI工具/政策/海外)并标注教育场景与目标用户;采集 Skill 产出候选信号 JSON 后调用。
---

# classify-education-signal — 信号分类与场景标注

对采集 Skill(`monitor-education-companies` / `monitor-china-models` 等)产出的候选信号做**类型分类**并**补全教育场景与目标用户**,供评分(`score-taoli-relevance`)、生成(`write-company-analysis`)与比例框架(50/25/15/10)分配使用。

本 Skill 属于**分析类(A-01)**,在 D 执行手册 **D5 步骤**中按 **去重 → 分类 → 初评** 顺序执行(分类在去重之后、初评之前,非并列)。

---

## 一、用途

回答三个问题:

1. **这条信号是什么类型**(教育企业 / 大模型 / AI工具 / 政策 / 海外)——决定其进入监测比例框架的哪个桶,以及后续落区(业务层 / 技术层 / 政策加急 / 速览,见 §四 步骤 2 映射表);
2. **AI 进入了哪一个教育场景**——对齐主文档「企业 AI 工作流拆解模板」与 A 分册 `education_scenario` 字段;
3. **这条信号服务谁**(教师 / 学生 / 家长 / 教务运营 / 学校)——对齐 A 分册 `target_users` 字段。

**输出边界(红线)**:

- 本 Skill **只补分类与场景字段**,不改写 `title` / `summary` / `evidence` / `source_level` 等事实字段;
- 本 Skill **不构造 dedup_key、不做去重**——去重归 D 执行手册 **D5 步骤**(D 分册),见 A 分册「去重规则」;
- 本 Skill **不做六维评分**——评分由 `score-taoli-relevance` 按 E 评分锚点手册执行;
- 本 Skill **不做来源回源核查**——核查由 `verify-source-evidence` 执行;
- 本 Skill **不改变 `report_status`、不自动发布**——发布归 D 执行手册**节点 4 人工操作**(AI 不自动发布)。

---

## 二、输入 schema

输入:一条候选信号 JSON(或一批信号的 JSON 数组,数组逐条处理,逐条返回)。

**必填(本 Skill 分类所依赖)字段**,取自 A 分册「信号 JSON 完整 Schema」:

| 字段 | 类型 | 必填 | 用途 |
|---|---|---|---|
| signal_id | string | 是 | 信号唯一 ID,回写用 |
| company / brand / product | string | 是 | 判定主体与类型(至少 company 存在) |
| title | string | 是 | ≤40 字,辅助判定内容 |
| summary | string | 是 | ≤80 字,辅助判定场景 |
| published_at / week | string | 是 | 时间归属,判定政策时效性辅助 |
| source_url | array\<string\> | 是(≥1) | 判定属地(海外 / 境内)与来源 |
| source_level | enum(S/A/B/C) | 是 | 来源等级,不参与分类但随信号保留 |
| source_type | string | 是 | 判定政策信号(official_policy 等)辅助 |
| new_change | string | 是 | 本次新变化,是「教育业务落地 vs 模型能力更新」判定的核心依据;不可为空 |
| workflow_stage | array\<string\> | 否(有则用) | 辅助判定教育场景 |
| ai_capabilities | array\<string\> | 否(有则用) | 辅助判定场景与目标用户 |
| evidence | array\<object\> | 否(有则用) | 判定理由引用的证据来源 |
| report_status | enum | 否 | 本 Skill **不修改**,原样保留 |

**字段缺失时的处理**:必填字段缺失 → 无法可靠分类,`signal_category` 标 `待人工`,并在 `limitations` 记录缺失字段;不硬猜。

---

## 三、输出 schema

在输入信号 JSON 基础上**新增/补全**以下字段(其余字段原样透传):

### 3.1 新增字段

| 字段 | 类型 | 说明 |
|---|---|---|
| signal_category | enum | 教育企业 / 大模型 / AI工具 / 政策 / 海外 / 待人工(取值见 §四 步骤 2) |
| signal_category_reason | string | 类型判定理由(≤80 字),必须引用信号字段或证据 |
| monitor_bucket | enum | 对应监测比例桶:教育企业及教育科技产品(约50%)/ 国内大模型、AI工具与技术平台(约25%)/ 教育政策、学校及区域落地案例(约15%)/ 海外重要参考(约10%)/ 待定(待人工) |

### 3.2 补全字段(对齐 A 分册)

| 字段 | 类型 | 说明 |
|---|---|---|
| education_scenario | string | 补全 AI 进入的教育场景,格式 `一级场景` 或 `一级场景-子场景`,取值见 §五 受控词表;来源已填写的按词表校订 |
| education_scenario_reason | string | 场景判定理由(≤80 字),引用 `workflow_stage` / `summary` / `evidence` |
| target_users | array\<string\> | 补全服务对象,取值严格限于 `教师 / 学生 / 家长 / 教务运营 / 学校`,至少 1 项,去重;无法判定不写入枚举(写入 limitations,见 §四 步骤 4) |

### 3.3 写入位置

- `signal_category` / `signal_category_reason` / `monitor_bucket` / `education_scenario_reason` 为**本 Skill 新增的扩展字段**,追加到信号 JSON 顶层(紧邻 `education_scenario` 之后),不覆盖任何既有字段;
- `education_scenario` / `target_users` 写入 A 分册既定顶层字段;
- 无法判定项写入 `limitations`(append),不阻塞后续人工处理。

### 3.4 输出示例(仅示新增字段位置)

```json
{
  "signal_id": "SIG-2026-W33-0001",
  "company": "好未来",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "title": "小思AI 1对1 新增作文批改 Agent 工作流",
  "signal_category": "教育企业",
  "signal_category_reason": "主体为国内教育企业好未来旗下学而思,new_change 为教育产品功能落地",
  "monitor_bucket": "教育企业及教育科技产品(约50%)",
  "education_scenario": "个性化学习-写作批改",
  "education_scenario_reason": "workflow_stage=作业批改→错因诊断→补弱,AI 进入写作练习闭环",
  "target_users": ["学生", "家长"],
  "report_status": "candidate"
}
```

---

## 四、执行步骤

### 步骤 1:校验输入

确认输入是信号 JSON(或数组)。检查 §二 必填字段;缺失 → 按 §二「字段缺失时的处理」执行。校验通过后逐条处理。

### 步骤 2:判定信号类型(signal_category)

按**信号实际内容**判定,不按公司所在名单硬套。自上而下命中第一个条件即取该类型:

**① 海外** — 主体机构 / 产品 / 政策 / 落地市场位于中国大陆以外。
- 判据:company/brand 为境外机构(如 OpenAI / Google / Khan Academy / Duolingo / Chegg 等)、source_url 域名为境外、或信号内容明确为境外教育市场落地。境外政策同样归「海外」(区域优先于类型)。
- 示例:OpenAI 发布面向教师的教学辅助功能、Coursera 推出 AI 助教。

**② 政策** — 主体为政府或官方教育管理机构发布的教育 AI 政策 / 规划 / 标准 / 试点 / 采购。
- 判据:`source_type=policy` 或 `source_type=official_news` 且发布主体为教育部、工信部、国家数据局、省教育厅、教育局等;信号内容为政策文件、试点名单、采购要求、标准规范。
- 境外政府政策 → 归「海外」(命中①),不归「政策」。

**③ 教育企业** — 主体为国内教育企业或其教育产品 / 服务,且 `new_change` 直接作用于教学、学习或教务场景(教育产品功能、教育工作流、教育落地案例、教育采购中标)。
- 判据:company 在教育企业观察名单(一级名单 7 家 + 二级名单),或 product 为教育产品(学习机 / 课堂系统 / 教辅 / 语言学习等),且 `new_change` 为**教育业务落地**。
- 双名单企业(如科大讯飞:既是 A5 教育企业、星火又在模型名单):按 `new_change` 内容判定——**教育产品 / 业务落地 → 教育企业**;模型平台能力更新 → ④。落地优先于能力。

**④ 大模型** — 主体为国内基础大模型平台,且 `new_change` 为**模型 / 平台能力本身**的变化(文档视觉、Agent 与工作流、生产应用三类能力,见主文档「国内模型与 AI 平台观察名单」),教育仅是可能用途、尚无教育场景落地。
- 判据:company 在模型名单(DeepSeek / Kimi / 通义 / 豆包 / 混元 / GLM / 文心 / MiniMax / 星火等),`new_change` 描述 API 价格、上下文长度、多模态、Function Calling、MCP、私有化部署、服务稳定性等**能力**而非教育产品功能。
- 星火教育大模型发布教育产品并进校 → 归 ③ 教育企业。

**⑤ AI工具** — 主体为国内**通用 AI 工具 / 技术平台**(非教育企业、非基础大模型),其能力可被教育采用:Agent 平台、知识库 / RAG 工具、MCP 生态、OCR / 多模态工具、AI 开发框架、低代码平台、数据标注等。
- 判据:非教育企业、非模型名单,`new_change` 为工具 / 平台能力更新,内容含「可用于教育」的服务能力。
- 示例:某知识库产品新增教育文档解析能力、某 Agent 平台上线课程资料整理工作流。

**⑥ 待人工** — 无法按以上判定:主体不明、`new_change` 为空或仅为口号、内容无实质信息。`signal_category=待人工`、`monitor_bucket=待定`,并在 `limitations` 记录原因。

**联动:类型 → 监测桶 → newsletter 落区**

- 将类型映射到 `monitor_bucket`(见 §三 3.1),供后续比例框架分配与监测覆盖检查使用;
- 按下表将 `signal_category` 映射到 newsletter 落区,兑现「决定写进业务层还是技术层」的宣称:

| signal_category | newsletter 落区 | 说明 |
|---|---|---|
| 教育企业 | 业务层(C-Bxx) | 教育产品 / 业务落地 → 业务层分析段 |
| 大模型 | 技术层(C-Txx) | 模型 / 平台能力更新 → 技术层条目 |
| AI工具 | 技术层(C-Txx) | 通用工具 / 平台能力 → 技术层条目 |
| 政策 | 政策加急 | 政策 / 规划 / 试点 / 采购,时效强优先加急 |
| 海外 | 速览 | 海外重要参考 → 速览区 |
| 待人工 | 暂不落区 | 待人工判定后再按上述规则映射 |

> **落区说明**:本表为**类别级偏好**;最终落区(必看/重点关注/速览/不进)以 E 评分锚点手册按评分确定,两者结合使用。

### 步骤 3:标注教育场景(education_scenario)

1. 从 `workflow_stage` / `summary` / `evidence` 提取 AI 实际进入的教育环节,从 §五「教育场景受控词表」选取最贴合的**一级场景**;需要更精确时追加**子场景**,格式 `一级场景-子场景`。
2. 判定规则:
   - **教育企业**:直接按产品功能进入的场景标注(如作文批改 → `作业批改` 或 `个性化学习-写作批改`);场景判定优先于公司名。
   - **大模型**:取该能力**最直接可服务**的教育场景(文档视觉 → `测评考试-试卷解析` / `个性化学习-题目理解`);若无明确教育落地 → `通用能力-待定教育场景`,并在 `limitations` 注明「尚未有教育场景落地」。
   - **AI工具**:取该工具在**教育侧的可能用途**场景,同「大模型」规则。
   - **政策**:取政策**直接作用**的教育场景(课后服务政策 → `课后辅导与自习`;教师 AI 能力要求 → `教师发展`);全局性综合文件 → `学校治理与区域-政策环境`。
   - **海外**:按信号实际进入的场景正常标注,与境内规则一致。
3. 一条信号可覆盖多个场景时,取**最主要的一个**作为 `education_scenario`;其余场景如需保留,写入 `limitations` 的 note,不拆多条。
4. 写 `education_scenario_reason`(≤80 字),必须引用字段或证据,禁止推测。

### 步骤 4:标注目标用户(target_users)

1. 从 `summary` / `workflow_stage` / `evidence` 判断直接服务对象,取值严格限于 `教师 / 学生 / 家长 / 教务运营 / 学校`:
   - 学生端功能(练习、自学、AI 伴学)→ `学生`;
   - 备课、批改、课堂、教研 → `教师`;
   - 家长报告、家长端 → `家长`;
   - 排课、招生、考勤、运营自动化 → `教务运营`;
   - 学校 / 区域平台、采购、进校方案 → `学校`;
   - 政策信号 → 取政策作用对象(`学校` / `教师`,试点类另加 `教务运营`);
   - 大模型 / AI工具能力信号 → 取该能力最可能被采用的教育角色(通常 `教师` 或 `教务运营`),并在 `education_scenario_reason` 说明依据。
2. 至少 1 项;可多项(如既服务学生又服务家长)。去重后按固定顺序 `教师 / 学生 / 家长 / 教务运营 / 学校` 排列。
3. 无法判定 → **不写入 `target_users` 枚举**(枚举仅限 `教师 / 学生 / 家长 / 教务运营 / 学校`),改为在 `limitations` 追加 `{aspect:"target_users", note:"待人工判定"}`,不硬猜。

### 步骤 5:一致性校验与输出

1. 校验 `education_scenario` 与 `workflow_stage` / `ai_capabilities` 是否自洽(如场景为 `作业批改` 但能力清单无批改相关项 → 在 `limitations` 记录疑似不一致,不改动原字段);
2. 校验 `target_users` 均为合法枚举值;
3. 校验 `report_status` **未被本 Skill 修改**(保持输入值,通常为 candidate);
4. 输出补全后的信号 JSON。数组输入则逐条输出数组。

---

## 五、写作规范

### 5.1 教育场景受控词表(一级场景 + 常用子场景)

| 一级场景 | 常用子场景 | 典型判定信号 |
|---|---|---|
| 教研备课 | 教案生成 / 课件制作 / 组卷命题 / 教研分析 | 教师备课、教案、课件、试卷命题 |
| 课堂授课 | 课堂互动 / AI助教 / 课堂实录分析 / 智能黑板 | 课堂、AI 助教、实时互动、黑板 |
| 作业批改 | 自动批改 / 作文批改 / 口语评测 / 错题归因 | 批改、作文、口语、错题归因 |
| 练习巩固 | 智能练习 / 变式重练 / 错题本 / 自主学习 | 练习、变式题、错题、刷题 |
| 测评考试 | 组卷 / 智慧考试 / 学情测评 / 作文评分 | 考试、测评、组卷、评分 |
| 学情诊断 | 学情报告 / 知识图谱 / 错因诊断 | 学情、诊断、报告、知识图谱 |
| 个性化学习 | AI 1对1 / 自适应路径 / 学习规划 / 课后补弱 | 1对1、个性化、学习路径、规划 |
| 课后辅导与自习 | AI 伴学 / 自习室 / 课后服务 | 自习、伴学、课后托管、课后服务 |
| 家校沟通 | 家长报告 / 家长端 / 家校协同 | 家长、家校、周报、报告 |
| 教务运营 | 排课 / 招生 / 考勤 / 教务自动化 | 排课、招生、教务、运营、CRM |
| 教师发展 | 教师培训 / 教研评价 / 教学技能 | 师训、教研、教师发展、评价 |
| 学校治理与区域 | 区域学情 / 教育治理 / 数据平台 | 区域、教育局、数据平台、治理 |
| 素养与素质教育 | 阅读 / 编程 / 口语 / 艺术 / 思维训练 | 阅读、编程、口语、思维、素养 |
| 语言学习 | 口语 / 翻译 / 写作 | 语言、口语、翻译、作文 |
| 职业教育 | 技能培训 / 实训 | 职教、实训、技能 |
| 高等教育 | 高校教学 / 科研辅助 | 高校、大学、科研 |
| 通用能力-待定教育场景 | — | 模型 / 工具能力尚无教育落地时使用 |

- 词表随方法论演进可扩展,扩展须登记 A 分册修订记录表;MVP 内若出现词表未覆盖场景,取最相近一级场景并备注。
- 政策通用性文件用 `学校治理与区域-政策环境`;模型 / 工具无明确教育落地用 `通用能力-待定教育场景`。

### 5.2 标注语言规则

1. 全部标注字段用简体中文,术语与 A 分册一致,不用英文缩写(允许品牌名等专名);
2. `education_scenario` 一级场景 ≤ 12 字,`一级场景-子场景` ≤ 24 字;
3. `signal_category_reason` / `education_scenario_reason` ≤ 80 字,格式:`依据 <字段>=<值>(+ <来源>)`,禁止推测性措辞(「可能」「或许」「想必」);
4. `target_users` 仅允许 5 个枚举值,不新增自定义用户类型;
5. 无法判定一律显式标 `待人工` 并写入 `limitations`,**不许硬猜填充**;证据缺失时倾向保守标注。其中 `target_users` **不写入枚举值 `待人工`**,仅在 `limitations` 记 `{aspect:"target_users", note:"待人工判定"}`(见 §四 步骤 4)。

### 5.3 判定纪律

- 类型判定**看内容不看公司名单**:双名单企业按 `new_change` 内容归入教育企业或大模型;
- 场景判定**先场景后公司**:即使是一线企业,`new_change` 未触及具体教育场景也不虚构场景;
- 海外信号归「海外」优先于「政策 / 教育企业 / 大模型 / AI工具」;境外政策归「海外」;
- 本 Skill 产出不改变 `report_status`,不触发任何发布行为。

---

## 六、示例

### 示例一:教育企业(完整输入 → 输出)

**输入(候选信号,节选):**

```json
{
  "signal_id": "SIG-2026-W33-0001",
  "company": "好未来",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "title": "小思AI 1对1 新增作文批改 Agent 工作流",
  "summary": "学而思小思AI 1对1新增AI作文批改,支持逐句诊断与改写建议,并写入学情报告。",
  "published_at": "2026-08-03",
  "event_at": "2026-08-01",
  "week": "2026-W33",
  "source_url": ["https://www.100tal.com/news/xxx", "https://mp.weixin.qq.com/s/xxx"],
  "source_level": "S",
  "source_type": "official_product_update",
  "new_change": "新增作文批改Agent:逐句诊断到生成改写建议并回流学情报告",
  "workflow_stage": ["作业批改", "错因诊断", "补弱"],
  "ai_capabilities": ["多模态理解", "作文诊断", "改写生成"],
  "evidence": [
    {"fact": "官方发布页于2026-08-01上线", "source_level": "S", "source_url": "https://www.100tal.com/news/xxx"},
    {"fact": "教师端可审核AI批改结果", "source_level": "A", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "report_status": "candidate"
}
```

**执行判定:**

- 类型:主体为国内教育企业(学而思),`new_change` 为教育产品功能落地 → `教育企业`;
- 场景:`workflow_stage=作业批改→错因诊断→补弱`,AI 进入写作练习闭环 → `个性化学习-写作批改`;
- 用户:批改面向学生作答、结果回流学情并可写家长报告 → `["学生", "家长"]`。

**输出(补全字段,其余透传):**

```json
{
  "signal_id": "SIG-2026-W33-0001",
  "company": "好未来",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "title": "小思AI 1对1 新增作文批改 Agent 工作流",
  "signal_category": "教育企业",
  "signal_category_reason": "依据 company=好未来、brand=学而思,new_change 为教育产品功能落地",
  "monitor_bucket": "教育企业及教育科技产品(约50%)",
  "education_scenario": "个性化学习-写作批改",
  "education_scenario_reason": "依据 workflow_stage=作业批改→错因诊断→补弱,AI 进入写作练习闭环",
  "target_users": ["学生", "家长"],
  "published_at": "2026-08-03",
  "event_at": "2026-08-01",
  "week": "2026-W33",
  "source_url": ["https://www.100tal.com/news/xxx", "https://mp.weixin.qq.com/s/xxx"],
  "source_level": "S",
  "source_type": "official_product_update",
  "new_change": "新增作文批改Agent:逐句诊断到生成改写建议并回流学情报告",
  "workflow_stage": ["作业批改", "错因诊断", "补弱"],
  "ai_capabilities": ["多模态理解", "作文诊断", "改写生成"],
  "evidence": [
    {"fact": "官方发布页于2026-08-01上线", "source_level": "S", "source_url": "https://www.100tal.com/news/xxx"},
    {"fact": "教师端可审核AI批改结果", "source_level": "A", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "report_status": "candidate"
}
```

### 示例二:其余类型分类要点(构造,节选)

| 输入要点(构造) | signal_category | monitor_bucket | education_scenario | target_users |
|---|---|---|---|---|
| Kimi 官方发布行业信息整理 Agent、长上下文与工具调用能力更新 | 大模型 | 国内大模型、AI工具与技术平台(约25%) | 通用能力-待定教育场景 | 教务运营 |
| 教育部发布《人工智能+教育行动计划》配套实施要求 | 政策 | 教育政策、学校及区域落地案例(约15%) | 学校治理与区域-政策环境 | 学校 |
| 某开源知识库产品新增教育文档 / 试卷解析能力 | AI工具 | 国内大模型、AI工具与技术平台(约25%) | 通用能力-待定教育场景 | 教师 |
| OpenAI 面向教师推出教学助手,可基于班级材料生成讲义 | 海外 | 海外重要参考(约10%) | 教研备课-教案生成 | 教师 |
| 某企业学习机发布会仅口号、无产品与工作流变化,`new_change` 为空 | 待人工 | 待定 | 通用能力-待定教育场景 | —(limitations: 待人工判定) |

### 示例三:双名单企业(边界判定,构造)

- 科大讯飞「星火大模型新增多模态公式识别能力(API)」→ `new_change` 为**模型能力**,归 `大模型`;
- 科大讯飞「星火教师助手进校,覆盖备课与课堂互动」→ `new_change` 为**教育产品落地**,归 `教育企业`。

---

## 附:与相邻 Skill / 分册的边界

| 事项 | 归属 |
|---|---|
| 去重(dedup_key 构造与合并) | D 执行手册 **D5 步骤**(不实现为 Skill) |
| 六维评分 | `score-taoli-relevance`(A-02,按 E 评分锚点手册) |
| 来源回源核查 | `verify-source-evidence`(A-03) |
| 发布 / 置 published | D 执行手册 **节点 4 人工操作**(AI 不自动发布) |
| 分类与场景标注 | **本 Skill classify-education-signal(A-01)** |
