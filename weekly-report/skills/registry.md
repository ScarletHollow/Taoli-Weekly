# Skill 注册表（MVP 精简版）

> **文档版本**:v1.0.0
> **修订日期**:2026-08-06
> **修订人**:Jason8345
> **变更**:MVP 首版,登记 9 个核心 Skill。完整 27-Skill 注册表与生命周期状态机推迟 **v0.4**(由真实周报暴露的问题驱动后深写)。
> **落地形态**:Skill 由 core-skills-impl 子任务实现为 `.claude/skills/` 平台技能(各含 SKILL.md);本表登记其元数据。

---

## 一、9 个核心 Skill 清单

| 编号 | Skill 名称 | 分类 | 职责 | 版本 | 状态 | 负责人 |
|---|---|---|---|---|---|---|
| M-01 | monitor-education-companies | 采集 | 可配置企业监控,覆盖一级名单 7 家(好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道),输入企业清单+时间窗 | v1.0.0 | 基线 | 方法论负责人 |
| M-02 | monitor-china-models | 采集 | 国内 9 模型能力更新监测(文档视觉/Agent 工作流/生产应用三类能力) | v1.0.0 | 基线 | 方法论负责人 |
| A-01 | classify-education-signal | 分析 | 信号分类(教育业务/模型/政策/海外)+ 教育场景标注 | v1.0.0 | 基线 | 方法论负责人 |
| A-02 | score-taoli-relevance | 分析 | 按 E 评分锚点手册对信号六维打分 + 总分 + 理由 | v1.0.0 | 基线 | 方法论负责人 |
| A-03 | verify-source-evidence | 分析 | B 级线索回源核查,输出 source_level + 已核实结论 | v1.0.0 | 基线 | 方法论负责人 |
| G-01 | write-company-analysis | 生成 | 单一企业 + 本期信号 → 业务层分析段 | v1.0.0 | 基线 | 方法论负责人 |
| G-02 | write-taoli-actions | 生成 | 本期 selected 条目 → 2-4 条桃李建议(四类分级) | v1.0.0 | 基线 | 方法论负责人 |
| G-03 | format-all-staff-newsletter | 生成 | 期元信息 + 各条目集 → 完整 newsletter markdown | v1.0.0 | 基线 | 方法论负责人 |
| C-01 | check-source-quality | 检查 | 每条核心事实 ≥1 S 级来源,source_level/source_url 完整 | v1.0.0 | 基线 | 方法论负责人 |

> **去重与发布不实现为 Skill**:去重由 D 分册「每日流程 D5」步骤负责(AI 读 dedup_key 做事件级去重);发布由 D 分册「每周流程 节点 4 终审发布」人工操作负责(AI 不自动发布)。

---

## 二、Skill 输入输出契约摘要

| Skill | 输入 | 输出 | 产物去向 |
|---|---|---|---|
| monitor-education-companies | 企业清单(默认 7 家)+ 时间窗 | 每家候选信号 JSON(company/brand/product/title/summary/source_url/source_level/education_scenario) | signals JSONL |
| monitor-china-models | 9 模型名单 + 三类能力 | 能力变化候选信号 JSON | signals JSONL |
| classify-education-signal | 候选信号 | 分类标签 + 教育场景 | signals JSONL 字段 |
| score-taoli-relevance | 信号 JSON + 桃李业务画像 | 六维档位 + 总分 + 理由 | signals JSONL scoring |
| verify-source-evidence | 信号 + source_url | 来源核查结论(S/A/B/C + 已核实) | signals JSONL verification |
| write-company-analysis | 单一企业 + 本期信号 | 业务层分析段 | C 模板业务层 |
| write-taoli-actions | 本期 selected 条目 | 2-4 条桃李建议 | C 模板建议区 |
| format-all-staff-newsletter | 期元信息 + 各条目集 | 完整 newsletter | draft.md |
| check-source-quality | 草稿 + 信号库 | 来源质量报告 | check-report |

> 完整契约(输入/输出 schema、触发时机、数据来源、写作规范)由 core-skills-impl 子任务的各 SKILL.md 定义。

---

## 三、无限扩展（占位，v0.4 深写）

- 新增企业 → 在 monitor-education-companies 的企业清单中增删即可,无需新建 Skill。
- 新增分析需求 → 按「analyze-业务-维度」命名新增分析 Skill,分配 A 段编号,登记本表。
- 完整生命周期状态机(Draft→Candidate→Shadow Test→Human Review→Active→Deprecated)与版本记录表推迟 v0.4。
