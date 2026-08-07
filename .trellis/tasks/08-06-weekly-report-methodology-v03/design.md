# 设计文档 — AI周报方法论 v0.3 MVP 落地

> **需求唯一来源**:`prd.md`(AI周报方法论 v0.3 MVP 落地 + 一期真实周报验证)。
> 本文档为该 PRD 的技术设计。范围以 PRD 为准:精简方法论 + 9 个核心 Skill + 一期真实周报;完整深写推迟 v0.4。

## 1. 架构总览

MVP 交付三件套,构成可运行的第一版闭环:

```
① 精简方法论文档  → 提供规范(信号 JSON / 流程 / 模板 / 锚点)
② 9 个核心 Skill  → 提供执行能力(采集/分析/生成/检查)
③ 一期真实周报    → 验证闭环(2026-W31 真实数据跑通)
```

文档体系存放于 `weekly-report/`:

```
weekly-report/
├── README.md
├── 桃李未来-周报方法论主文档-v0.3.md   # 骨架:范围/名单/分级/评分权重/工作流模板/抓取节奏/最小范围/版本管理
├── 分册/
│   ├── A-信号数据手册.md       # 信号 JSON + 精简状态 + JSONL 库 + 去重规则
│   ├── C-周报输出规范.md       # 全员版 newsletter 模板 + 简版语言规范
│   ├── D-执行手册.md           # 人机分工 + 每日/每周流程 + 简版 QA + 审核 checklist
│   └── E-评分锚点手册.md       # 6 维权重 + 每维 3 档锚点 + 过滤规则
├── skills/
│   └── registry.md             # 9 个核心 Skill 精简注册表
├── signals/                     # JSONL 历史信号库(signals-YYYY-WW.jsonl)
├── drafts/                      # 每周草稿
├── published/                   # 发布版快照
└── archive/
    └── v0.2-原稿.md             # v0.2 全文归档,不覆盖
```

## 2. 精简策略(对照完整版)

| 维度 | 完整版(v0.4 深写) | MVP(本次) |
|---|---|---|
| 方法论文档 | 主文档 + 5 分册全量 | 主文档骨架 + 4 精简分册(A/C/D/E) |
| Skill | 27-Skill 注册表 + 生命周期状态机 | 9 个核心 Skill + 精简注册表 |
| 状态机 | 六态 + merge/superseded 细节 | 精简态(candidate/selected/rejected/frozen/published) |
| 评分锚点 | 6×5 档全量锚点 + 多案例 | 6 维权重 + 每维 3 档(高/中/低)+ 过滤规则 |
| QA gate | 6-check 全量 + 多层失败处理 | 简版(首期 check-source-quality;可读性检查并入 C 分册简版语言规范,不单独立 Skill) |
| 审核 checklist | 全量 A-G 项 | 精简版 |

> 架构不丢:信号 JSON 契约、评分权重、流程骨架保留;深度由真实周报暴露的问题驱动 v0.4 补全。

## 3. 关键契约

### 3.1 评分体系(唯一口径)

- **6 维 100 分制**:教育业务关系 25 / 桃李关系 25 / 工作流创新 15 / 可验证 15 / 可靠度 10 / 影响力 10。
- 权重数值**单点维护于主文档**;A 分册只引用;E 分册提供 3 档锚点(高/中/低),三者链接不复制数值。

### 3.2 信号 JSON 与精简状态

- 信号 JSON 字段:`signal_id / company / brand / product / title / summary / published_at / event_at / week / source_url[](≥1) / source_level(S-A-B-C) / source_type / education_scenario / workflow_stage / ai_capabilities / new_change / target_users / evidence[] / limitations[] / scoring(六维+总分+理由) / report_status / dedup_key / reviewer_feedback`。
- 精简状态流转:`candidate`(采集入库)→ `selected/rejected`(初评去重后)→ `frozen`(周五冻结)→ `published`(发布归档)。MVP 不引入六态与 merge/superseded 细节。

### 3.3 JSONL 历史信号库

- 目录:`signals/signals-YYYY-WW.jsonl`(ISO 周命名)。
- 首行写 schema 元信息 `{"_schema": "2026-08-v1"}`;追加写入;冻结生成本周终版;不修改历史周文件。
- 非 git 仓库,写入前对本周文件做 `.bak` 备份。
- 去重与跨周对比读取本库的 `dedup_key`。

### 3.4 9 个核心 Skill(全流程覆盖)

| 分类 | Skill | 职责 | 输出/产物去向 |
|---|---|---|---|
| 采集 | **monitor-education-companies** | 可配置企业监控,覆盖一级名单 7 家(好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道) | 候选信号 JSON → signals JSONL |
| 采集 | monitor-china-models | 国内 9 模型能力更新监测 | 候选信号 JSON → signals JSONL |
| 分析 | classify-education-signal | 信号分类(教育业务/模型/政策/海外)+ 教育场景 | signals JSONL 字段 |
| 分析 | score-taoli-relevance | 按 E 锚点六维打分 + 总分 + 理由 | signals JSONL scoring |
| 分析 | verify-source-evidence | B 级线索回源核查(source_level + 已核实) | signals JSONL verification |
| 生成 | write-company-analysis | 企业分析段(业务层条目) | C 模板业务层 |
| 生成 | write-taoli-actions | 2-4 条桃李建议(四类分级) | C 模板建议区 |
| 生成 | format-all-staff-newsletter | 聚合完整 newsletter | draft.md |
| 检查 | check-source-quality | 每条核心事实 ≥1 S 级来源 | check-report |

> **monitor-education-companies 设计要点**:输入为「企业清单(默认一级名单 7 家)」+ 时间窗,可配置单家或多家;对每家企业执行检索(官网/官方公众号/财报/行业媒体)并产出候选信号 JSON。替代原 monitor-tal 的单企业监控,支持一家 Skill 覆盖全部一级名单,后续新增企业仅需更新配置清单。

### 3.5 去重与发布归属(明确)

- **去重:由 D 分册「每日流程 D5(保存候选 + 初评去重)」步骤负责**。AI 在该步骤读取 signals JSONL 的 `dedup_key` 做事件级去重(同事件合并,保留最高 source_level 行),不单独立 Skill。周五冻结前对全池重跑一次。
- **发布:由 D 分册「每周流程 节点 4(终审发布)」步骤负责**。为**人工操作**,AI 不自动发布;发布动作= 推送全员渠道 + 信号置 `published` + 留存发布版快照 `published/YYYY-WW-final.md`。

## 4. 执行模型(人机协作半自动)

- **AI**:采集(D1-D4)/ 去重(D5)/ 初评评分 / 生成草稿。
- **人**:冻结候选池(周五 14:00)/ 审核评分修正(周一 09:30)/ 终审发布(周一 12:00)。
- AI 不自动冻结、不自动发布、评分不直接进正文。

## 5. 兼容与演进

- v0.2 全部有效内容保留(监测比例/名单/S-A-B-C 分级/评分权重/工作流模板/抓取节奏/最小范围),归档于 `archive/v0.2-原稿.md`。
- 分册独立版本号;主文档「版本管理」章节定义修订流程。
- 完整深写(27-Skill 注册表、六态、6×5 锚点、多层 QA)标注为 v0.4 待办。

## 6. 权衡

- 主文档可读性优先,细节下沉分册;深度收敛,架构与数据契约保留。
- 平台技能落地(`.claude/skills/`),直接可调,D 流程步骤引用 Skill 名。
- 采集类 Skill 依赖 WebSearch/WebFetch,失败降级为"人工代采集"标注,不阻塞流程。
