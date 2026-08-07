---
name: check-source-quality
description: "来源质量检查：逐条核对信号核心事实是否有 ≥1 个 S 级来源、source_level/source_url 是否完整、有无无来源事实，输出带 minor/major/critical 分级的 check-report。在 D 执行手册节点 2 生成初稿后执行简版 QA、或对采集/去重阶段的信号库做来源质量把关时使用。"
---

# 来源质量检查 Skill（C-01）

## 一、用途

对**本期信号库与草稿**做来源质量检查，回答三个问题：

1. **每条核心事实是否 ≥1 个 S 级来源**（S 级定义见主文档「信息源分级」：政府官网 / 企业官网 / 官方产品页 / 官方技术文档 / API 日志 / 官方 GitHub / 模型卡 / 技术报告 / 财报 / 发布会实录）；
2. **source_level / source_url 是否完整**（A 分册信号 JSON：`source_url` 为数组且 ≥1，`source_level` 必须取 S/A/B/C 之一，两者必须相互对应——`evidence[]` 中每条证据的 `source_url` 可回溯到该条 `source_url` 数组）；
3. **有无无来源事实**（信号正文、draft.md 中出现了无法回溯到任何 `evidence` / `source_url` 的陈述，即「孤儿事实」）。

本 Skill 是**检查类 Skill**，只产检查报告，不修改信号数据、不生成正文。输出 `check-report`，作为 D 执行手册节点 3 人工审核的输入。

**输出位置**：`weekly-report/drafts/YYYY-WW-check-report.md`（YYYY 年、WW 两位 ISO 周号，与本期 draft.md 同目录）。

**边界（红线）**：本 Skill 不做去重（归 D 分册 D5 步骤），不涉及自动发布（发布归 D 分册节点 4 人工操作，AI 不自动发布、不自动把信号置为 published）。本 Skill 只读检查，不改 `report_status`。

## 二、输入 schema

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| draft_path | string | 是 | 本期初稿 `drafts/YYYY-WW-draft.md` 绝对路径 |
| signals_path | string | 是 | 本期信号库 `signals/signals-YYYY-WW.jsonl` 绝对路径 |
| week | string | 是 | 本期 ISO 周号，如 2026-W33（决定 check-report 命名） |
| focus_facts | array\<string\> | 否 | 指定需重点核查的核心事实清单（缺省则逐条全量核查） |

**输入数据来源**：

- 草稿按 **C 周报输出规范** 结构生成（期要点 → 业务层 → 技术层 → 桃李建议 → 附注）；
- 信号库每行一条信号 JSON，字段按 **A 信号数据手册** 2.1 字段总表；核查以 `evidence[]`、`source_url`、`source_level`、`scoring` 为主；
- S/A/B/C 分级判定与使用规则见 **主文档「信息源分级」**；
- B 级线索应已被 `verify-source-evidence` 回源核查（信号 `verification` 字段），本 Skill 复查其是否落实。

## 三、输出 schema

输出为 **check-report**（markdown），由三部分组成：结论、问题清单、附注。落盘路径 `weekly-report/drafts/YYYY-WW-check-report.md`。

```json
{
  "week": "2026-W33",
  "generated_at": "2026-08-07T21:00:00+08:00",
  "overall": "pass | pass_with_minor | needs_major_fix | blocked",
  "signal_count_checked": 6,
  "fact_count_checked": 14,
  "problems": [
    {
      "signal_id": "SIG-2026-W33-0002",
      "fact": "逐句诊断支持多学科",
      "severity": "critical",
      "type": "no_s_level_source",
      "detail": "核心事实唯一来源为 B 级媒体转述，未回源到 S 级页面；draft.md C-B02 已引用该陈述",
      "recommendation": "回源企业官网/产品页核查后再定稿，或将该事实降级为 A 级并删去越级表述",
      "blocking": true
    },
    {
      "signal_id": "SIG-2026-W33-0001",
      "fact": "新增作文批改 Agent 工作流",
      "severity": "major",
      "type": "source_level_mismatch",
      "detail": "evidence[0] source_level=S 但 source_url 指向 mp.weixin.qq.com（B 级），signal 级 source_level 与证据不一致",
      "recommendation": "补齐官方产品页/官网 S 级 URL，或下调 source_level",
      "blocking": true
    },
    {
      "signal_id": "SIG-2026-W33-0003",
      "fact": "上线日期 2026-07-30",
      "severity": "minor",
      "type": "field_incomplete",
      "detail": "source_url 数组含 1 条，但该条无对应 evidence 项；字段完整但证据链缺环节",
      "recommendation": "在 evidence 中为上线日期补充 {fact, source_level, source_url}",
      "blocking": false
    }
  ],
  "notes": [
    "共 14 条核心事实：S 级 11 条、A 级 2 条、B 级 1 条（B 级已标记待回源）",
    "SIG-2026-W33-0002 阻断本次草稿进入节点 3，退回节点 2 修复"
  ]
}
```

**severity 与判定**（对齐 D 执行手册「失败处理」）：

| 级别 | 定义 | 判定 | 处理 | overall |
|---|---|---|---|---|
| critical | 核心事实无 S 级来源 / 内容将误导读者 | 核心事实（产品名、发布日期、功能、价格、关键数字、用户数据等）无 ≥1 S 级来源，或该事实被 draft.md 引用 | 阻断发布；退回节点 3 重新审核或移除该条 | blocked |
| major | 事实、来源分级、建议等级或结构与规范不符 | 核心事实有 S 级来源但 source_level/source_url 相互矛盾、evidence 无法回溯、B 级事实越级当 S 级表述、无来源事实被写入正文 | 必须修复后再进入审核；退回节点 2 重跑 | needs_major_fix |
| minor | 不影响事实正确性的措辞、格式或编号问题 | 字段完整但证据链缺环节、措辞/编号/标注位置问题 | 标注在 check-report 后重跑，不阻断流程 | pass_with_minor |

**overall 判定**：任何 critical → `blocked`；否则任何 major → `needs_major_fix`；否则任何 minor → `pass_with_minor`；全通过 → `pass`。

## 四、执行步骤

### 第 1 步：读取输入

1. 读 `signals_path` 信号库与 `draft_path` 初稿；若信号库已冻结，优先读 `signals/signals-YYYY-WW-frozen.json`。
2. 解析每行信号 JSON（首行 `{"_schema":"2026-08-v1"}` 为 schema 元信息，跳过）。
3. 从 draft.md 提取所有核心事实陈述（C-Bxx / C-Txx 条目标题、一句话要点、正文、桃李含义、来源标注、附注存疑点）。

### 第 2 步：逐条核对核心事实来源

对每条核心事实执行三层核对：

1. **S 级覆盖**：该事实是否在信号 `evidence[]` 中至少一条 `source_level=S` 的证据支撑；且该 S 级证据的 `source_url` 属于主文档「信息源分级」的 S 级来源类型。缺 S 级 → **critical**。
2. **字段完整与自洽**：`source_url` 数组 ≥1；`source_level` ∈ {S,A,B,C}；`evidence[].source_url` 能回溯到 `source_url` 数组之一；`evidence[].source_level` 与 signal 级 `source_level` 不矛盾（signal 级取最高，证据不得高于信号级）。违反 → **major**。
3. **无来源事实**：draft.md 中出现的每个事实陈述是否都能在 `evidence[]` 中找到来源；找不到 → **major**（孤儿事实）。孤儿事实若涉及产品名/日期/功能/价格/数据等核心事实 → 升级为 **critical**。

### 第 3 步：B 级与 A 级事实处理

- **B 级**：作为线索可以存在，但**不得作为唯一事实来源**写入主要结论。凡 draft.md 以 B 级为唯一依据的陈述：若 `verification` 已回源核实 → 按其回源后的 `source_level` 判级；未回源 → **major**，并在 check-report 标注「待回源」，交 `verify-source-evidence`。
- **A 级**：核心事实仍须 ≥1 S 级；A 级证据只能补充（如实际案例、演示）。无 S 级时按 **critical** 处理（D 执行手册：无 S 级的按 A 级标注并在正文注明——仅当该事实非「核心事实」时可用此豁免，核心事实无豁免）。
- **C 级**：不得作为事实依据进入主要结论；出现在信号 `limitations` 或附注存疑点中可保留并如实标注。若被当作事实写入正文 → **major**。

### 第 4 步：S 级自述 vs 案例区分

企业自述（S 级，如官网/发布会）与真实案例（A 级，如学校应用案例）必须在正文中**区分表述，不得混写**（C 分册 3.3 硬性约束）。发现混写 → **major**。

### 第 5 步：生成 check-report

1. 汇总 `overall` 与 `problems[]`（按 critical → major → minor 排序）。
2. 记录 `notes`：本轮核查事实总数、各级来源分布、S 级缺口汇总、阻断项清单。
3. 落盘 `weekly-report/drafts/YYYY-WW-check-report.md`，作为节点 3 人工审核的输入。

### 第 6 步：处理与交接

- `pass` / `pass_with_minor` → 可进入节点 3 人工审核，minor 项在报告内标注后重跑核对即可。
- `needs_major_fix` → 退回 D 执行手册节点 2 重跑，修复后再执行本 Skill。
- `blocked` → **阻断发布**；退回节点 3 重新审核或移除该条。任何情况下本 Skill 不自动发布、不自动改 `report_status`。

## 五、写作规范

1. **只读检查**：本 Skill 不修改信号 JSON、不改写 draft.md、不改 `report_status`；所有问题以问题清单形式呈现，修复动作由人触发。
2. **判断依据唯一**：来源级别以主文档「信息源分级」为唯一依据；字段规则以 A 信号数据手册为唯一依据；severity 以 D 执行手册「失败处理」为唯一依据。
3. **核心事实清单**（必须 S 级）：产品名称、发布日期、功能变化、价格、关键数字、用户/落地数据、政策条款。**非核心事实**（分析推断、趋势判断、行业评论）不强制 S 级，但必须有可回溯来源，否则按 major 记录。
4. **可追溯**：每个问题必须给出 signal_id + 具体事实 + 违规类型 + 修复建议，禁止泛化描述。
5. **措辞中性**：报告用事实语言（「无 S 级来源」「source_level 不一致」），不做价值评判，不写推广腔。
6. **红线**：本 Skill 不触发任何发布动作、不写 published、不处理去重（去重归 D5 步骤）。

## 六、示例

**场景**：D 执行手册节点 2 生成 `drafts/2026-W33-draft.md` 后，对 `signals/signals-2026-W33.jsonl` 执行来源质量检查。

**输入信号（节选）**：

```json
{
  "signal_id": "SIG-2026-W33-0001",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "new_change": "新增作文批改Agent:逐句诊断到生成改写建议并回流学情报告",
  "source_url": ["https://mp.weixin.qq.com/s/xxx"],
  "source_level": "S",
  "evidence": [
    {"fact": "新增作文批改Agent", "source_level": "S", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "verification": null
}
```

**检查判定**：

| 事实 | 判定 | severity | 依据 |
|---|---|---|---|
| 新增作文批改 Agent（draft.md C-B01 核心事实） | source_level 声称 S，但唯一 URL 是微信订阅号（B 级），S 级证据不成立 | **critical** | 核心事实无真 S 级来源，且已被 draft.md 引用 |
| evidence 与 source_url 自洽性 | 证据可回溯，但分级虚高 | 随上条 critical | source_level= S 与 URL 域不符 |

**check-report 节选**：

```markdown
# 2026-W33 来源质量检查报告
- 结论：blocked
- 核查信号 6 条 / 核心事实 14 条：S 级 11、A 级 2、B 级 1
- critical 1：SIG-2026-W33-0001「新增作文批改Agent」唯一来源为 B 级微信订阅号，
  source_level=S 虚高，draft.md C-B01 已引用。修复：回源到学而思官网/官方产品页
  取得 S 级 URL，或降级为 A/B 级并在正文注明后移除越级表述。
- 处理：阻断本次草稿进入节点 3，退回节点 2 修复。
```

> 本示例为构造场景，仅演示判定与报告结构，不代表真实信号。
