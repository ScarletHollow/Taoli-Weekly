# 设计文档 — 核心 Skill 实现(core-skills-impl)

## 1. 架构总览

实现 **9 个核心 Skill**,覆盖采集→去重→评分→生成→审核→发布全流程。落地形态由本设计定夺(见 §4)。登记于 `weekly-report/skills/registry.md`。

```
9 个核心 Skill
├── 采集(2):  monitor-education-companies, monitor-china-models
├── 分析(3):  classify-education-signal, score-taoli-relevance, verify-source-evidence
├── 生成(3):  write-company-analysis, write-taoli-actions, format-all-staff-newsletter
└── 检查(1):  check-source-quality
```

## 2. 与 docs-mvp 的契约对接

每个 Skill 的输入/输出 schema 与 **A 分册信号 JSON** 对齐;流程步骤引用 **D 分册**;生成产物落位 **C 分册模板**;评分依据 **E 分册锚点**。

| Skill | 输入 | 输出 | 产物去向 |
|---|---|---|---|
| monitor-education-companies | 可配置企业清单(默认一级名单 7 家:好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道) | 每家企业候选信号 JSON(company/brand/product/title/summary/source_url/source_level/education_scenario) | signals JSONL |
| monitor-china-models | 国内 9 模型能力更新 | 候选信号 JSON(能力变化) | signals JSONL |
| classify-education-signal | 候选信号 | 分类标签(教育业务/模型/政策/海外)+ 教育场景 | signals JSONL 字段 |
| score-taoli-relevance | 信号 JSON + 桃李业务画像 | 6 维档位 + 总分 + 理由 | signals JSONL 评分字段 |
| verify-source-evidence | 信号 + source_url | 来源核查结论(S/A/B/C + 是否已核实) | signals JSONL verification |
| write-company-analysis | 单一企业 + 本期信号 | 企业分析段(业务层条目) | C 模板业务层 |
| write-taoli-actions | 本期 selected 条目 | 2-4 条桃李建议(四类分级) | C 模板建议区 |
| format-all-staff-newsletter | 期元信息 + 各条目集 | 完整 newsletter markdown | draft.md |
| check-source-quality | 草稿 + 信号库 | 来源质量报告(每条核心事实 ≥1 S 级) | check-report |

## 3. 落地形态(定夺)

- **MVP 采用 `.claude/skills/` 平台技能**形式:每个 Skill 一个 SKILL.md,含元数据(name/description)+ 指令正文(输入输出 schema、触发时机、写作规范)。
- 理由:与 docs-mvp 的方法论一致,可直接被 Claude Code 调用,D 分册流程步骤可写"调用 Skill X";不引入额外运行时。
- 与完整版关系:Skill 内容按 B 分册注册表规范组织(编号/版本/负责人),但生命周期状态机推迟 v0.4。
- 冒烟测试:每个 Skill 用最小输入样例在本地验证输出符合 schema。

## 4. 关键设计决策

- **评分 Skill 输入**:依赖桃李业务画像(主文档观察名单章节,独立版本),MVP 用简版画像(业务线/学段/客户群)。
- **monitor-education-companies 设计**:输入为「企业清单(默认一级名单 7 家)+ 时间窗」,可配置单家/多家/增删企业;对每家企业执行检索策略(官网/官方公众号/财报/行业媒体)并产出候选信号 JSON。替代原 monitor-tal 的单企业监控,一家 Skill 覆盖全部一级名单,后续新增企业仅需更新配置清单。
- **去重**:MVP 内嵌于流程 D5 步骤(读取 signals JSONL 的 dedup_key 做事件级去重),不单独立 Skill。
- **发布**:由 D 分册每周流程「节点 4 终审发布」人工操作负责,AI 不自动发布;发布动作含信号置 published + 留存 published/YYYY-WW-final.md。
- **来源核查**:verify-source-evidence 用 WebSearch/WebFetch 对 B 级线索回源核查。
- **语言**:所有 Skill 指令正文为中文。

## 5. 兼容与演进

- Skill schema 变更遵循主文档版本管理;Skills 版本记录存 registry.md。
- 完整 27-Skill 注册表与生命周期状态机推迟 v0.4;新 Skill 按注册流程登记。

## 6. 权衡

- 平台技能(轻量,直接可调)vs 独立脚本(更强,但偏离方法论) → 选平台技能,MVP 够用且可演进。
- 采集类 Skill 依赖网络检索,受 WebSearch/WebFetch 可用性约束,失败降级为人工采集标注。
