---
name: verify-source-evidence
description: "B 级线索回源核查:用 WebSearch/WebFetch 把 B 级或存疑来源回源到企业官网/政府/产品原始页,核实事实,判定 source_level 升降档,写入 verification。当信号来源为 B/C 级、多来源并存待聚核、或事实存疑(功能/日期/价格/数据)时使用。"
---

# verify-source-evidence — 来源等级与真实性核查

> **Skill 编号**:A-03(分析类)
> **版本**:v1.0.0
> **所属体系**:桃李未来教育 AI 周报方法论 MVP(主文档 v0.3 + 分册体系)
> **职责**:B 级线索回源核查,验证来源等级与真实性,输出核查结论。写入 `verification{source_level,checked_url,conclusion,note}` 并更新信号 `source_level`。

---

## 一、用途

本 Skill 解决一个问题:**信号标了 B 级(或存疑)来源,这些事实能不能信?** 行业媒体(芥末堆、多知、36 氪、机器之心、量子位、新智元等)只负责「发现线索」,不直接作为唯一事实来源(见主文档「六、信息源分级」)。凡是 B 级/C 级/存疑来源的信号,在进入评分与正文前,都必须回到**企业官网、政府官网、官方产品页或原始文档**核查一次。

具体触发时机(D 执行手册 D4 媒体线索扫描):

- 线索来自 B 级媒体,或来源等级判定为 B 或 C;
- 多篇媒体重复报道同一事件(聚合前先回源 S/A 原始页面);
- 关键事实存疑:产品名称、发布日期、功能、价格、落地数据、合作方等与常识/历史记录不符;
- 信号 `source_url` 中无 S/A 级原始页面,或 `evidence` 无 S 级证据。

**本 Skill 的边界(红线,见 CONTRACT):**

- **不实现去重**:本 Skill 只做回源核查,不做报道行合并;多篇报道行的合并(保留最高 source_level 主行)与 dedup_key 构造归 **D 执行手册 D5 步骤**(对齐 A 分册第五节)。
- **不实现发布**:不自动发布、不把信号置为 published;发布归 **D 执行手册 节点 4 人工操作**。
- **不评分**:六维评分由 `score-taoli-relevance`(E 分册锚点)负责;本 Skill 只修正 `source_level` 与 `verification`,为评分维度「信息可靠度」提供依据。
- **不替人工判断「是否进正文」**:核查结论是事实判断,是否采纳进周报仍由人决定。

**与相邻 Skill 的分工**:

| Skill | 职责 | 与本文的边界 |
|---|---|---|
| `classify-education-signal` | 事件分类 + 教育场景标注 | 先分类建档,后核查来源 |
| `score-taoli-relevance` | 六维评分(按 E 分册) | 核查结论作为「信息可靠度」维度输入;本 Skill 不改分数 |
| `check-source-quality` | 成稿后核对「每条核心事实 ≥1 S 级来源」 | 本 Skill 在采集/归档阶段完成;check 在成稿阶段把关 |

---

## 二、输入 Schema

调用本 Skill 需要一组已建档的候选信号。每条信号的**必填输入**如下(其余字段照原样保留):

```json
{
  "signal_id": "SIG-2026-W33-0001",
  "company": "好未来",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "title": "学而思小思AI 1对1 新增作文批改 Agent",
  "summary": "据媒体报道,学而思小思AI 1对1新增AI作文批改,支持逐句诊断与改写建议。",
  "new_change": "新增作文批改Agent工作流(待核查)",
  "source_url": ["https://www.jiemodui.com/N/xxxx"],
  "source_level": "B",
  "source_type": "media_report",
  "evidence": [
    {"fact": "新增AI作文批改,逐句诊断并生成改写建议", "source_level": "B", "source_url": "https://www.jiemodui.com/N/xxxx"}
  ]
}
```

**输入字段说明**:

| 字段 | 类型 | 说明 |
|---|---|---|
| `signal_id` | string | 待核查信号唯一 ID |
| `company` / `brand` / `product` | string | 定位回源目标(用 `company` 官方站优先) |
| `title` / `summary` / `new_change` | string | 待核实的事实陈述 |
| `source_url` | array\<string\>(≥1) | 当前全部来源;含待核查的 B/C 级 URL |
| `source_level` | enum(S/A/B/C) | 当前来源等级(本 Skill 将复核并可能更新) |
| `source_type` | string | 来源类型(media_report / user_review 等) |
| `evidence` | array\<object\> | 核心事实证据;逐条核实其 `source_level` 是否名副其实 |

> 输入契约对齐 **A 信号数据手册 2.1 字段总表**。若输入 `source_level=S` 且无存疑点,本 Skill 复核通过即可,不必强制回源。

---

## 三、输出 Schema

每条输入信号,本 Skill 产出两项更新:

### 3.1 `source_level`(更新)

| 规则 | 说明 |
|---|---|
| 保留 | 回源后最高原始来源仍与原判一致,保留原等级 |
| 升档 | **S/A 级原始页面直接核实事实 → 升档**。B→S(官方产品页/官网新闻/技术文档);B→A(官方 App 更新记录、案例、招投标);A→S(确认原始页为企业官网/政府页) |
| 降档 | 回源失败 / 无法核实 / 官方原始页失效 → 降档。S→A(官方 S 页失效,仅剩 A 级原始页支撑);S→B(仅媒体转述,无原始页);A→B(仅有媒体转述,无原始页);B→C(仅社交平台或未确认截图能支撑) |
| **C→S 对齐最高来源级别** | C 级源回源**官方原始页直接命中并核实事实**时可升至 S(命中 S 页则 S),与 A 分册「最高来源级别」(source_level=该信号最高来源级别)一致;无官方原始页、仅 C 级佐证(截图/社交)时不升 |

> 升档必须满足 **「多源独立交叉一致」或「官方原始页直接核实」** 之一(见第五节第 4 步)。单条 S 级 URL 命中但页面内容与事实不符时,不升档并如实记录。

### 3.2 `verification`(写入/覆盖)

```json
"verification": {
  "source_level": "S",
  "checked_url": "https://www.100tal.com/news/xxx",
  "conclusion": "已核实",
  "note": "回源企业官网新闻页,确认2026-08-01上线作文批改Agent;与媒体(B)报道一致,可升档至S"
}
```

**字段说明**(对齐 A 信号数据手册 2.2 嵌套对象约定):

| 字段 | 取值 | 说明 |
|---|---|---|
| `source_level` | S/A/B/C | 核查后的**最终**来源等级(与更新后的 `source_level` 一致) |
| `checked_url` | URL 或 "" | **实际打开过的**原始页面 URL(企业官网/政府页/官方产品页/技术文档)。未能回源时留 `""` |
| `conclusion` | `已核实` / `存疑` / `无法核实` | 三选一(见 3.3) |
| `note` | string | 核查备注:回源了哪个页面、核实了哪条事实、交叉证据有哪些、还差什么 |

**写入约定**:

- `verification` 原为 `null` 时写入新对象;已有旧值(如前一版核查)时覆盖更新,旧版本在 note 中可留痕。
- 未核查或无需核查的信号保持 `verification=null`,但本 Skill 一旦被调用,必须落定非 null。
- 更新写入目标:`weekly-report/signals/signals-YYYY-WW.jsonl` 对应行(仅更新 `source_level` 与 `verification`,不动其他字段)。写入前 `.bak` 备份,追加/改行不重写整文件(见 D 执行手册「交接物与文件规范」)。

### 3.3 conclusion 判定标准

> **conclusion 严格取三值枚举**:`已核实` / `存疑` / `无法核实`,不接受拼接值(如「已核实(多源交叉)」);「多源交叉 / 待补官方页」等核查细节一律写入 `note`。

| conclusion | 判定条件 | 示例 note 关键句 |
|---|---|---|
| **已核实** | 在 S/A 级原始页上**直接命中**媒体宣称的核心事实(产品、日期、功能、数据);或 ≥2 个**独立来源**交叉一致(交叉细节写入 note) | 「回源官网产品页,功能与日期与媒体一致」 |
| **存疑** | 回源到的原始页**与媒体宣称不一致**(夸大、旧闻重发、张冠李戴),或只能找到 C 级佐证 | 「官网无此功能/日期与报道不符,判存疑」 |
| **无法核实** | 回源失败(页面 404/打不开/无法访问)、官网无相关记录、且无其他独立来源 | 「未找到任何 S/A 级原始页,无法核实」 |

---

## 四、执行步骤

按顺序执行;每步结论都进入 note,最终落到 `verification`。

### 第 1 步:识别待核实事实

- 从 `title` / `summary` / `new_change` / `evidence[].fact` 中提取 **≤3 条核心事实**(建议优先:产品/功能、发布日期、关键数据或合作方)。
- 标注每条事实当前来源等级与 URL。
- 若输入 `source_level=S` 且 `source_url` 已含官方页、无存疑点 → 直接判「已核实」,`checked_url` 取该官方页,进入第 5 步写回,不必再搜索。

### 第 2 步:检索原始页面(WebSearch)

对每条核心事实做定向检索,目标是**企业官网、官方产品页、政府官网、官方技术文档**:

- 检索式:`` `<company> <product> <功能/日期>` ``、`site:` 限定官方域(如 `site:100tal.com`)、或直接查 `company 官网 产品页`。
- 域名甄别:官方域一般是公司名拼音/缩写(100tal.com、neworiental.com、yuanfudao.com、iFlytek 域、seewo.com、youdao.com);拿不准时优先检索并核对站内是否有官方产品页导航。
- 同一事件有多篇 B 级报道时,**聚合回源**:各篇指向的原始事实一致即可交叉确认,不必逐篇回源。
- 无官方页命中时,再扩大检索范围查 A 级来源(学校/教育局案例、招投标、App 更新记录、专利)作为替代证据。

### 第 3 步:打开页面核实(WebFetch)

- 对命中的 S/A 级候选页面调用 WebFetch,核对核心事实是否**逐条命中**。
- **禁止只靠标题/摘要下结论**:必须读到正文中与 `fact` 对应的句子。
- 记录:页面实际 URL(可能与搜索链接不同,以最终打开的为准)、命中的事实、命中日期。
- 页面 404/打不开/内容无关 → 该 URL 不算命中,回第 2 步或进入「无法核实」。

### 第 4 步:判定等级与 conclusion

按以下顺序判定(冲突时以「低风险优先」):

1. **官方原始页直接核实** → `conclusion=已核实`;判定可否升档:
   - 命中**企业官网/官方产品页/技术文档/政府页/财报/发布会记录** → 升到 S;
   - 命中**官方 App 更新记录/案例/招投标/学校教育局案例** → 升到 A;
   - 无法命中官方原始页、仅多篇 B 级交叉一致 → **保持 B**,conclusion 记 `已核实`,note 写「多源交叉一致,无 S/A 原始页,等级封顶 B,待补官方页」。
2. **官方原始页与媒体宣称矛盾** → `conclusion=存疑`;`source_level` 不升,必要时降档(如发现事实仅出自未确认截图 → 降 C)。
3. **无法回源**(检索无果 + 页面全 404 + 无 A 级替代证据)→ `conclusion=无法核实`;`source_level` 降档:原 S → A(仅剩 A 级原始页支撑)、原 S → B(仅媒体转述)、原 A → B、原 B → C(无法维持原等级)。
4. **C 级来源** → 命中官方原始页并核实事实可升至 S(与「最高来源级别」一致,对齐 3.1);无官方原始页命中则保持 C 并记「无法核实」。

**多源独立交叉一致的可信度**:

- **官方页 + 独立媒体**一致 → 最强,正常升档;
- **多个独立媒体**一致但**均无原始页** → 事实可信,但等级封顶 B(不升 S);
- **单篇媒体**且无原始页 → 不可作为核心事实,保持 B/C 并在 note 记录「单源、待原始页佐证」。

### 第 5 步:写回信号并落文件

- 更新信号的 `source_level`(如判定升/降档)与 `verification`(source_level / checked_url / conclusion / note)。
- 若 `evidence` 中出现 S/A 级命中,可补充/替换对应事实的 `source_level` 与 `source_url`(evidence 与 verification 保持一致)。
- 写入 `weekly-report/signals/signals-YYYY-WW.jsonl` 对应行:先 `cp signals-YYYY-WW.jsonl signals-YYYY-WW.jsonl.bak`,再改行,不重写整文件、不触碰历史周文件。
- **红线确认**:本步骤只写信号库字段,不冻结、不置 published、不发布(发布由 D 执行手册节点 4 人工完成)。

### 第 6 步:产出核查摘要(供人工终审)

每条信号给出**一行摘要**,格式:

> `SIG-xxx · 媒体宣称「<核心事实>」 → 回源 <checked_url> · conclusion=<已核实/存疑/无法核实> · 等级 B→S`

存疑/无法核实的条目,summary 中列出**还差什么证据**才能核实(如「需官方产品页确认上线日期」),供评分(score-taoli-relevance)与人工终审参考。

---

## 五、写作规范

核查备注(note)与核查摘要的写法约束:

1. **事实优先、措辞克制**:只写「核实了什么、在哪个页面、与媒体是否一致」,不写「疑似」「可能」(那属于存疑判定,写进 conclusion 而不是模糊带过)。
2. **区分自述与实证**:企业官网(S 级)是企业自述,与第三方实测/案例(A 级)是两回事;note 中明确标注「S 级=官方自述,未亲验」。
3. **每条核心事实落到一个 URL**:note 里核对的事实必须能对应到 checked_url(或 evidence 里某条 source_url),不允许「无出处结论」。
4. **诚实记录失败**:页面 404、检索无果、官网无记录,都是有效核查结果,如实写,不编造 checked_url。
5. **不夸大升档**:单条 URL 命中不等于事实真实;功能夸大、日期不符时判「存疑」而不是升档。宁可保留 B/C 并标注待补证据,也不为凑 S 级而升档。
6. **等级与 conclusion 必须一致**:conclusion=已核实 + source_level 仍为 B(无原始页)是允许的;但 conclusion=无法核实 时 source_level 不得为 S/A。
7. **中文正文,短句**:note ≤ 80 字为宜;面向人工终审可读,可回溯。
8. **不改其他字段**:本 Skill 只动 `source_level`、`verification`(必要时 `evidence` 的等级/URL),不动 title/summary/new_change/scoring/report_status/dedup_key。

---

## 六、示例

### 示例 1:媒体线索 → 官方页核实 → 升 S(已核实)

**输入**

```json
{
  "signal_id": "SIG-2026-W33-0002",
  "company": "科大讯飞",
  "brand": "讯飞",
  "product": "星火教师助手",
  "title": "科大讯飞星火教师助手新增AI听评课",
  "summary": "据行业媒体,星火教师助手新增AI听评课,自动生成课堂反馈报告。",
  "new_change": "新增AI听评课(待核查)",
  "source_url": ["https://www.jiqizhixin.com/articles/xxxx"],
  "source_level": "B",
  "source_type": "media_report",
  "evidence": [{"fact": "新增AI听评课并生成课堂反馈报告", "source_level": "B", "source_url": "https://www.jiqizhixin.com/articles/xxxx"}]
}
```

**执行**

1. 核心事实:「星火教师助手新增 AI 听评课,自动生成课堂反馈报告」。
2. WebSearch `科大讯飞 星火教师助手 AI听评课` → 命中 `site:iflytek.com` 官方产品页。
3. WebFetch 官方产品页 → 正文确认「AI 听评课」功能与「生成课堂反馈报告」表述;页面发布日期与媒体一致。
4. 判定:官方产品页直接命中 → `已核实`;原始页为企业官网 → 升档 B→S。

**输出**

```json
{
  "signal_id": "SIG-2026-W33-0002",
  "source_level": "S",
  "verification": {
    "source_level": "S",
    "checked_url": "https://www.iflytek.com/product/xxxx",
    "conclusion": "已核实",
    "note": "回源官方产品页(iflytek.com),确认AI听评课与课堂反馈报告功能上线,与媒体(B)报道一致,升档至S;S级为官方自述,未亲验"
  }
}
```

### 示例 2:多源交叉、无原始页 → 保持 B(已核实,封顶 B)

**执行**:WebSearch 返回芥末堆 + 36 氪 + 教育行业公众号三篇一致报道,但**均未找到企业官网/产品页**对应内容,也无 App 更新记录。

**输出**:`conclusion=已核实`,`source_level` 保持 `B`,`checked_url=""`。note:「三篇独立媒体交叉一致,事实可信;无官方原始页佐证,等级封顶B,待补官方页」。

### 示例 3:回源矛盾 → 存疑,不升档

**执行**:媒体称「8 月上线智能批阅机」,回源企业官网新闻页 → 官网显示该产品 3 月已上线、8 月仅是旧闻重发。

**输出**:`conclusion=存疑`,`source_level` 保持 `B`(不升档),note:「官网产品页显示3月已上线,8月报道为旧闻重发,与媒体宣称的'新上线'不符,判存疑」。

### 示例 4:C 级截图线索 → 无官方原始页 → 保持 C,无法核实

**执行**:线索来自小红书截图(C 级),WebSearch/WebFetch 均未找到企业官网或任何 S/A 级佐证。

**输出**:`conclusion=无法核实`,`source_level` 保持 `C`,note:「仅C级截图,未命中任何官方原始页;按 3.1,无官方原始页核实不可升档,该事实不可作为核心证据进入正文」。

---

## 七、版本与维护

- 本 Skill 版本 **v1.0.0**,随主文档 v0.3 / A 分册 v1.0.0 同批落盘。
- schema 演进、升降档规则的修订走主文档「方法论版本管理」流程,并在 skills/registry.md 交叉登记。
- 完整版(六态状态机下的核查状态、merge/superseded 处理)待 v0.4 深写。
