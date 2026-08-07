# 核心 Skill 实现(core-skills-impl)

## Goal

实现 **9 个核心 Skill**(覆盖采集→去重→评分→生成→审核→发布全流程),登记于精简 Skill 注册表,为一期真实周报验证提供可调用工具。完整 27-Skill 注册表与生命周期状态机推迟 v0.4。

## Confirmed Facts

- 父任务:08-06-weekly-report-methodology-v03(MVP 落地方向)。
- 依赖:docs-mvp 子任务的信号 JSON / 流程规范(D 分册)提供 Skill 的输入输出契约。
- 9 个核心 Skill 已确认:monitor-education-companies(可配置企业监控,默认一级名单 7 家)、monitor-china-models、classify-education-signal、score-taoli-relevance、verify-source-evidence、write-company-analysis、write-taoli-actions、format-all-staff-newsletter、check-source-quality。
- 去重/跨周对比 MVP 阶段内嵌于流程(**由 D 分册每日流程 D5 步骤负责**:AI 读 dedup_key 做事件级去重),不单独立 Skill。
- **发布**由 D 分册每周流程「节点 4 终审发布」人工操作负责,**AI 不自动发布**;本子任务实现的 Skill 不得包含自动发布行为。

## Requirements

- R1: 实现 9 个核心 Skill,每个含:输入 schema / 输出 schema / 触发时机 / 数据来源 / 产物去向(五栏契约),契约字段与 A 分册信号 JSON 对齐。
- R2: Skill 落地形态:可调用技能(本仓库为 Claude 平台,.claude/skills/ 或任务内脚本),输出写入 signals JSONL 或周报草稿。
- R3: 精简 Skill 注册表:登记 9 个 Skill(编号/名称/分类/版本/负责人),完整生命周期状态机推迟 v0.4。
- R4: 每个 Skill 冒烟测试:最小输入样例可跑通并产出符合 schema 的输出。
- R5: 与 D 分册流程对接:D 分册流程步骤可调用对应 Skill。**monitor-education-companies 须为可配置企业监控**(输入企业清单,默认一级名单 7 家,支持单家/多家/增删),一家 Skill 覆盖全部一级名单。

## Acceptance Criteria

- [ ] AC1: 9 个 Skill 全部实现且可被调用(非仅文档)。
- [ ] AC2: 每个 Skill 通过冒烟测试(最小输入 → schema 合规输出),测试记录留存。
- [ ] AC3: 采集类 Skill(monitor-education-companies 覆盖好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道,monitor-china-models)能采集真实公开信息写入信号 JSON 草稿。
- [ ] AC4: 评分 Skill(score-taoli-relevance)能按 E 分册锚点对信号打分输出六维/总分。
- [ ] AC5: 生成类 Skill(write-company-analysis / write-taoli-actions / format-all-staff-newsletter)能聚合为 newsletter 草稿。
- [ ] AC6: 精简注册表登记完成,与 docs-mvp 分册引用一致。

## Out of Scope

- 不实现 27-Skill 全套、不写生命周期状态机全细节(推迟 v0.4)。
- 不实际采集完整一周数据(由 first-real-weekly 负责,本任务只做冒烟验证)。
- 不实现定时调度/外部平台对接。

## Dependencies

- 依赖 docs-mvp:信号 JSON / 流程规范先行提供。
- 本子任务完成后,first-real-weekly 依赖其 Skill 可调用。

## Open Questions

- Skill 落地目录与可调用方式:`.claude/skills/`(平台技能)还是任务内脚本(MVP 轻量)?由 design.md 定夺。
